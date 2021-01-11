# github-docs-to-wiki
Syncs markdown documentation files in a repo to its wiki

## Usage

| Name | Usage | Required? |
| - | - | - |
| githubToken | A GitHub PAT with Repo access. Note: This cannot be the `GITHUB_TOKEN` secret as that is scoped to the source repo, but the wiki is a separate repo | Yes |
| defaultBranch | Specifies the default branch name to use for converted absolute links | No, default is the output of `git branch --show-current` |
| rootDocsFolder | Relative path within the repo to the root documentation folder | No, default is the repo's root |
| convertRootReadmeToHomePage | If true, the `README.md` file at the root of the repo will be renamed to `Home.md` in the wiki so that it is used as the wiki homepage | No, default is false |
| useHeaderForWikiName | If true, will extract the top-line header (denoted by a single `#`) and use that as the wiki page's name. Note: if this results in a name collision the sync will fail | No, default is false and wiki names will be the relative path to the file with `/` converted to `__` (e.g. `path/to/doc.md` becomes `path__to_doc.md`) |
| customWikiFileHeaderFormat | If set, inserts a header at the top of each wiki file with the given format<br/>Supports the following format subsitutions:<br/>- `{sourceFileLink}`: the absolute url to the source file in the repo | No, default will not add a header |
| customCommitMessageFormat | If set, uses the given format for the commit message to the wiki. Useful to correlate changes to the source.<br/>Supports the following format subsitutions:<br/>- `{commitMessage}`: the latest commit message for HEAD<br/>- `{shaFull}`: the full SHA of HEAD<br/>- `{shaShort}`: the short SHA of HEAD | No, default is `"Sync Files"` |

## How it works

1. Clone the wiki repo, delete all existing files
2. The root directory is recursively scanned for documentation files (using `*.md`)
   1. For each file, all links are extracted and checked for conversion
      1. If a link is already absolute, do nothing
      2. If a link is to another `.md` file within the docs path, convert the link to point to a wiki page
      3. Else, convert the link to an absolute Url pointing to a file in the repo
   2. If `useHeaderForWikiName` is set, check the file for a header and, if present, remove that line and set the wiki name accordingly
   3. Otherwise set the wiki name as the path to the file with `/` converted to `__`
   4. Output the new content to the wiki
3. Scan the wiki directory and update any links where the name was overridden in `2.ii`
4. Commit and push the wiki files
