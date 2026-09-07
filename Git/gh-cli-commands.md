# GitHub CLI (`gh`) Commands

Running log of `gh` (GitHub CLI) commands used, with syntax and explanation.
`gh` is GitHub's official command-line tool for working with repos, issues,
PRs, etc. without leaving the terminal (different from plain `git`, which
only talks to git itself).

---

## `gh auth status`

**Syntax:**
```
gh auth status
```

**Explanation:** Shows whether you're logged in to GitHub via the CLI, which
account is active, the auth protocol (https/ssh), and the token's scopes
(permissions). Useful to check before running anything that needs auth.

---

## `gh repo list`

**Syntax:**
```
gh repo list <owner> --limit <n> --json <fields>
```

**Example used:**
```
gh repo list premkumar3965 --limit 200 --json name,isPrivate,updatedAt,url
```

**Explanation:** Lists repositories owned by `<owner>`. `--limit` caps how
many are returned (default is 30). `--json` outputs structured JSON with
only the requested fields instead of the default table — handy for scripting
or piping into other tools.

---

## `gh repo delete`

**Syntax:**
```
gh repo delete <owner>/<repo> --yes
```

**Example used:**
```
gh repo delete premkumar3965/mcino-Introduction-to-Git-and-GitHub --yes
gh repo delete premkumar3965/github-final-project --yes
gh repo delete premkumar3965/Centralized-repository-shipping_calculations --yes
gh repo delete premkumar3965/LogisticsShippingRates --yes
gh repo delete premkumar3965/gkpbt-css-circle --yes
```

**Explanation:** Permanently deletes a repository (code, issues, PRs, stars —
everything). `--yes` skips the interactive confirmation prompt. Requires the
auth token to have the `delete_repo` scope. **Irreversible** — used here to
remove 5 repos while keeping `Data-Analytics`.

---

## `gh auth refresh` — adding a permission scope

**Syntax:**
```
gh auth refresh -h <host> -s <scope>
```

**Example used:**
```
gh auth refresh -h github.com -s delete_repo
```

**Explanation:** By default, `gh auth login` grants a limited set of scopes
(`repo`, `read:org`, `gist`, `workflow`) — deliberately *not* `delete_repo`,
so a login alone can never delete anything. `gh auth refresh` asks for one
more scope on top of what you already have, without logging out. It reopens
the browser for you to approve just that extra permission. This is what
gave the token the ability to run `gh repo delete` above.

You can check which scopes your token currently has with `gh auth status`
(see the "Token scopes" line).

## Removing a permission scope

There's no direct "remove one scope" command — `gh auth refresh` only ever
*adds* scopes. To take a scope like `delete_repo` back away, there are two
options:

**Option 1 — logout and log back in with only the scopes you want:**
```
gh auth logout -h github.com
gh auth login -h github.com --scopes "repo,read:org,gist,workflow"
```
(`--scopes` on `login` lets you name the exact list up front, instead of
getting the default set and refreshing.)

**Option 2 — revoke it from GitHub's side, then re-login:**
1. Go to github.com → **Settings** → **Applications** →
   **Authorized OAuth Apps**
2. Find **"GitHub CLI"** and click **Revoke**
3. Run `gh auth login` again — you're back to a fresh token with only the
   default scopes

Either way, the safest habit is: only add `delete_repo` right before you
actually need to delete something, then remove it again afterward so an
accidental `gh repo delete` isn't possible day-to-day.
