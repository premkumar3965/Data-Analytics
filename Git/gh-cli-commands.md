# GitHub CLI (`gh`) Commands

Running log of `gh` (GitHub CLI) commands used, with explanation and
examples. `gh` is GitHub's official command-line tool for working with
repos, issues, PRs, etc. without leaving the terminal (different from
plain `git`, which only talks to git itself).

----------------------------------------
`gh auth status`

Shows whether you're logged in to GitHub via the CLI, which account is
active, the auth protocol (https/ssh), and the token's scopes
(permissions). This is how you check what access the CLI currently has.

```
gh auth status
```
----------------------------------------

----------------------------------------
`gh repo list`

Lists repositories owned by an account. `--limit` caps how many are
returned (default is 30). `--json` outputs structured JSON with only the
requested fields instead of the default table — handy for scripting or
piping into other tools.

```
gh repo list <owner> --limit <n>
gh repo list premkumar3965 --limit 200 --json name,isPrivate,updatedAt,url
```
----------------------------------------

----------------------------------------
`gh repo delete`

Permanently deletes a repository (code, issues, PRs, stars — everything).
`--yes` skips the interactive confirmation prompt. Requires the auth token
to have the `delete_repo` scope. **Irreversible.**

```
gh repo delete <owner>/<repo> --yes
gh repo delete premkumar3965/mcino-Introduction-to-Git-and-GitHub --yes
```
----------------------------------------

----------------------------------------
`gh auth refresh` — add a permission scope

By default, `gh auth login` grants a limited set of scopes (`repo`,
`read:org`, `gist`, `workflow`) — deliberately *not* `delete_repo`, so a
plain login can never delete anything. `gh auth refresh` asks for one more
scope on top of what you already have, without logging out. It reopens the
browser to approve just that extra permission.

```
gh auth refresh -h <host> -s <scope>
gh auth refresh -h github.com -s delete_repo
```
----------------------------------------

----------------------------------------
`gh auth logout` — log out

Removes the stored credentials for an account/host from this machine. Used
before logging back in with a different account, or before re-logging in
with a different, smaller set of scopes.

```
gh auth logout -h <host>
gh auth logout -h github.com
```
----------------------------------------

----------------------------------------
`gh auth login` — log in

Starts GitHub's login flow. With no flags it grants the default scope set
and opens a browser for you to approve. Adding `--scopes` lets you name the
exact list of scopes up front, instead of accepting the defaults and using
`gh auth refresh` afterward.

```
gh auth login -h <host>
gh auth login -h github.com --scopes "repo,read:org,gist,workflow"
```
----------------------------------------

## Adding vs. removing a scope

There's no direct "remove one scope" command — `gh auth refresh` only ever
*adds* scopes. To take a scope like `delete_repo` back away:

- **Logout + login with an explicit list** — `gh auth logout` then
  `gh auth login --scopes "repo,read:org,gist,workflow"` (leaving
  `delete_repo` out).
- **Revoke from GitHub's side** — github.com → **Settings** →
  **Applications** → **Authorized OAuth Apps** → find **"GitHub CLI"** →
  **Revoke**, then `gh auth login` again for a fresh token with only the
  default scopes.

Safest habit: only add `delete_repo` right before you actually need to
delete something, then remove it again afterward so an accidental
`gh repo delete` isn't possible day-to-day.
