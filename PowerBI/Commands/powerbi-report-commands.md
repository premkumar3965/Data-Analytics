# Power BI Report — Commands & Techniques Reference

Reference notes on how the `SuperStore_Analysis` Power BI report was built
and edited **programmatically** (outside the Power BI Desktop UI), for
whenever this needs to be repeated or extended. Command style follows
`Git/gh-cli-commands.md` — Command / Explanation.

## Contents

1. [Finding Power BI Desktop's local data-engine port](#1-finding-power-bi-desktops-local-data-engine-port)
2. [Reading the model with ADOMD.NET (DAX queries)](#2-reading-the-model-with-adomdnet-dax-queries)
3. [Editing the model with TOM (Tabular Object Model)](#3-editing-the-model-with-tom-tabular-object-model)
4. [Editing report visuals as files (PBIR format)](#4-editing-report-visuals-as-files-pbir-format)
5. [Validating JSON before reloading in Desktop](#5-validating-json-before-reloading-in-desktop)
6. [Gotchas hit along the way](#6-gotchas-hit-along-the-way)

---

## 1. Finding Power BI Desktop's local data-engine port

Every open Power BI Desktop file runs its own local Analysis Services
engine (`msmdsrv.exe`) on a random localhost port. Find it before
connecting to anything:

```powershell
Get-Process -Name "PBIDesktop","msmdsrv" -ErrorAction SilentlyContinue |
    Select-Object Id, ProcessName, StartTime

Get-NetTCPConnection -OwningProcess <msmdsrv PID> -State Listen |
    Select-Object LocalAddress, LocalPort
```

The port changes every time Power BI Desktop restarts — re-run this each
session.

## 2. Reading the model with ADOMD.NET (DAX queries)

Power BI Desktop ships its own ADOMD client DLL (no separate install
needed). Use it to run DAX queries against the live model — good for
verifying measures/totals without touching anything:

```powershell
$dll = "C:\Program Files\WindowsApps\Microsoft.MicrosoftPowerBIDesktop_<version>_x64__8wekyb3d8bbwe\bin\Microsoft.PowerBI.AdomdClient.dll"
Add-Type -Path $dll
$conn = New-Object Microsoft.AnalysisServices.AdomdClient.AdomdConnection("Data Source=localhost:<port>")
$conn.Open()
$cmd = $conn.CreateCommand()
$cmd.CommandText = "EVALUATE ROW(""Total Sales"", [Total Sales])"
$reader = $cmd.ExecuteReader()
while ($reader.Read()) { $reader.GetValue(0) }
$conn.Close()
```

List databases first with `SELECT [CATALOG_NAME] FROM $SYSTEM.DBSCHEMA_CATALOGS`
if you don't already know the catalog GUID, then reconnect with
`;Catalog=<guid>` appended to the data source string.

The `Microsoft.MicrosoftPowerBIDesktop_...` folder path lives under
`C:\Program Files\WindowsApps\` — find the exact version with:

```bash
find "/c/Program Files/WindowsApps" -maxdepth 1 -iname "*PowerBIDesktop*"
```

## 3. Editing the model with TOM (Tabular Object Model)

ADOMD.NET is **read-only** (DAX/MDX queries only). To create tables,
columns, relationships, and measures programmatically, use the full
AMO/TOM client library — **not bundled with Power BI Desktop itself**, but
available for free if **SQL Server Management Studio (SSMS)** is
installed:

```powershell
$ideDir = "C:\Program Files\Microsoft SQL Server Management Studio 22\Release\Common7\IDE"
Add-Type -Path (Join-Path $ideDir "Microsoft.AnalysisServices.Core.dll")
Add-Type -Path (Join-Path $ideDir "Microsoft.AnalysisServices.dll")
Add-Type -Path (Join-Path $ideDir "Microsoft.AnalysisServices.Tabular.dll")

$server = New-Object Microsoft.AnalysisServices.Tabular.Server
$server.Connect("Data Source=localhost:<port>")
$db = $server.Databases[0]
$model = $db.Model
# ... build Table / Partition / Column / Relationship / Measure objects ...
$model.SaveChanges()
$server.Disconnect()
```

Core recipe used to rebuild the Superstore model from scratch:

- **Import table**: `Table` + `Partition` with an `MPartitionSource`
  (its `.Expression` is a plain Power Query M string) + explicit
  `DataColumn` objects (one per CSV column, with matching `DataType`).
  Auto schema-detection from the M query alone did **not** work reliably
  (see gotchas) — always define columns explicitly.
- **Calculated table** (e.g. the `Date` table): same pattern but with a
  `CalculatedPartitionSource` (DAX expression) and `CalculatedTableColumn`
  objects instead of `DataColumn`.
- **Relationship**: `SingleColumnRelationship` — set `.FromColumn` /
  `.ToColumn` only; `.FromTable`/`.ToTable` are read-only (derived).
- **Measure**: `Measure` object with `.Expression` = plain DAX string,
  added to a table's `.Measures` collection.
- **Trigger a data refresh**: `$table.RequestRefresh([Microsoft.AnalysisServices.Tabular.RefreshType]::Full)`
  then `$model.SaveChanges()`.
- **Fix a relationship added after data was already loaded**:
  `$model.RequestRefresh([...]::Calculate)` then `$model.SaveChanges()`
  (recalculates relationships/measures without a full reimport).

## 4. Editing report visuals as files (PBIR format)

Once a `.pbix` is saved as **Power BI Project files** (File → Save As →
`.pbip`), the report layout becomes plain JSON under
`<Report>.Report/definition/`, one file per visual:
`definition/pages/<pageId>/visuals/<visualId>/visual.json`.

This can be hand-authored/edited directly — no Power BI API needed for
report visuals otherwise. Two ways to get the exact JSON shape right
instead of guessing:

- **Microsoft's public PBIR JSON schemas** — fetch and read directly,
  e.g. `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.9.0/schema.json`
  (and the `page`, `visualConfiguration`, `semanticQuery` schemas it
  references).
- **`microsoft/skills-for-fabric` on GitHub** — an official Microsoft
  skill built specifically for authoring PBIR reports with an AI agent.
  Far more useful than the raw schemas alone: has worked examples and
  documented gotchas for every visual type.
  ```bash
  curl -s "https://raw.githubusercontent.com/microsoft/skills-for-fabric/main/plugins/powerbi-authoring/skills/powerbi-report-authoring/references/<file>.md" -o "<local path>.md"
  ```
  Key reference files used: `slicers.md`, `card.md`, `cartesian.md`,
  `theming.md`, `page-formatting.md`, `formatting-overview.md`,
  `color-strategy.md`, `table.md`.

  > Note: `WebFetch` summarizes pages through a small model and will
  > refuse or truncate large verbatim JSON blocks. For anything where
  > exact JSON matters, `curl` the raw file directly and read it with
  > the normal file-read tool instead.

Central places that control the *whole report's* look (edit these, not
every individual visual, when a global style change is wanted later):

- **Theme** (data colors, fonts, backgrounds for chart internals):
  `StaticResources/RegisteredResources/Superstore-Custom-Theme.json`,
  registered via `themeCollection.customTheme` +
  `resourcePackages` in `definition/report.json`.
- **Page canvas/wallpaper color**: `definition/pages/<pageId>/page.json`
  → `objects.background` / `objects.outspace` (page background has no
  `show` property — it's always visible, unlike a visual's background).

## 5. Validating JSON before reloading in Desktop

Power BI Desktop gives no useful error for a malformed PBIR file — it
just silently fails to load or drops the change. Always lint every JSON
file after an edit, before asking to reload in Desktop:

```powershell
$root = "<Report folder>"
Get-ChildItem -Path $root -Filter *.json -Recurse | ForEach-Object {
    try { Get-Content $_.FullName -Raw | ConvertFrom-Json -ErrorAction Stop | Out-Null; "OK   $($_.Name)" }
    catch { "FAIL $($_.Name) -- $($_.Exception.Message)" }
}
```

(Plain `python -m json.tool` isn't available in this environment —
PowerShell's `ConvertFrom-Json` works fine as a substitute.)

**Reload procedure**: since Power BI Desktop holds the whole project in
memory once opened, it will not pick up file edits made outside it.
Close the file **without saving** (so Desktop's in-memory state doesn't
get written back over the edit), then reopen the `.pbip`.

## 6. Gotchas hit along the way

- **Date parsing culture bug**: `Table.TransformColumnTypes(..., type date)`
  in Power Query uses the *machine's* locale by default. The Superstore
  CSV's `M/D/YYYY` US-format dates silently mis-parsed to `null` for any
  day-of-month > 12 under a non-US locale, dumping ~60% of rows into a
  blank/unmatched year bucket. Fix: pass `"en-US"` as the explicit third
  (culture) argument to `Table.TransformColumnTypes`.
- **TOM schema auto-detection doesn't reliably work**: adding a `Table`
  with an M/DAX partition but no explicit `Column` objects, then calling
  `RequestRefresh`, produces a table with a single synthetic
  `RowNumber-<guid>` stub column instead of the real columns — even for
  a trivial `#table(...)` M expression with no file/credentials involved.
  Always define columns explicitly; don't rely on auto-detection.
- **Wrong class name**: the calculated-table partition source class is
  `CalculatedPartitionSource`, not `CalculatedTableSource` (easy guess,
  wrong — verify TOM type names via reflection if unsure:
  `[Microsoft.AnalysisServices.Tabular.Table].Assembly.GetTypes() | Where-Object Name -match '...'`).
- **`Relationship.FromTable`/`.ToTable` are read-only** — set
  `.FromColumn`/`.ToColumn` only; the table references are derived
  automatically from the columns.
- **PBIR visual titles get an auto-generated subtitle** (from the bound
  field names, e.g. "Sum of Sales by Category") the moment a
  `visualContainerObjects.title` is set — a separate `subTitle` object,
  shown by default via the base theme. Set
  `visualContainerObjects.subTitle.show = false` explicitly per visual
  to remove it.
- **Setting *any* `visualContainerObjects` (VCO) on a visual drops the
  theme's default padding for that visual** (resets to 0, chrome sits
  flush against the border). Whenever setting one VCO (e.g. `subTitle`),
  also explicitly set `background`, `border` (with `radius`), `padding`,
  and `visualHeader` together — don't leave the others to "inherit",
  they silently reset instead.
- **`cardVisual` (the modern KPI card) uses role name `Data`**, not
  `Values` (that's the deprecated legacy `card` visual's role name).
  Using the wrong one renders an empty visual.
- **Slicer style (List/Dropdown/Tile) is three different things**, not
  one property with three values:
  - `slicer` visual + `objects.data.mode = 'Dropdown'` → dropdown
  - `slicer` visual + `objects.data.mode = 'Basic'` → inline list
  - `advancedSlicerVisual` (a completely different `visualType`) → tile/button layout
- **`dataPoint.fill` without a selector can render an invisible
  series** on some chart types (data is there, tooltips work, nothing
  draws). For a single-measure chart, don't set `dataPoint` at all —
  the theme's first data color already applies automatically — or use
  `dataPoint.defaultColor` if an explicit override is really needed.
