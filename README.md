# Watcher

Watcher shows the state of every git repo in one folder on one screen. It can also list the workspaces of Herdr.

## What it does

Watcher is one Bash script. It looks at each folder in a parent folder that holds a `.git` directory. It prints one line for each repo and sorts the lines by the time of the last commit. The newest repo is at the top.

Each line shows these values:

- The repo name.
- The current branch.
- The number of changed files, as `N dirty`. This number includes untracked files.
- The commits behind and ahead of the upstream branch, as `↓N ↑N`. The line shows `no-remote` when the branch has no upstream branch.
- The age of the last commit.

You can select a line to see `git status -sb` and the last 8 commits. You can also open `lazygit` or a terminal in the repo.

## Requirements

- Linux.
- `bash`, `git`, `awk`, `sort`, and `setsid`.
- `fzf`.
- `lazygit`, for the `Enter` key.
- `xdg-terminal-exec`, for the `Ctrl-T` key.
- `jq` and `herdr`, for the `--herdr` mode.

## Install

1. Copy the script to a folder in your `PATH`:

   ```
   install -m 755 watcher ~/.local/bin/watcher
   ```

2. Open a new terminal.
3. Run `watcher`.

## Usage

Run the command:

```
watcher
```

Watcher reads the folder `~/Work` by default. To read another folder, set the variable `PROJECTS_DIR`:

```
PROJECTS_DIR=~/src watcher
```

Keys:

| Key | Action |
|---|---|
| Type text | Filter the list by any part of a line. |
| `Enter` | Open `lazygit` in the selected repo. |
| `Ctrl-T` | Open a new terminal in the selected repo. |
| `Esc` | Close Watcher. |

Watcher lists only the direct subfolders of the parent folder. It lists a linked worktree and a submodule in the same way as a normal repo.

### Herdr mode

Run Watcher with the option `--herdr`:

```
watcher --herdr
```

This mode lists the workspaces of the running Herdr server. It does not read git repos. Each line shows the status of the agent, the workspace label, and the title of the agent chat.

The list has this order: `blocked`, `done`, `unknown`, `working`, and `idle`. A workspace that needs your input is at the top.

The preview shows the last 30 lines of the agent screen. `Enter` runs `herdr workspace focus` for the selected workspace and closes Watcher. `Esc` closes Watcher. The key `Ctrl-T` is not bound in this mode.

## Configuration

| Setting | Default | Effect |
|---|---|---|
| `PROJECTS_DIR` | `$HOME/Work` | The parent folder that Watcher reads. |

All other values are in the script. Edit `watcher` to change the keys, the columns, or the number of commits in the preview.

## How it works

The function `row` prints one line for one repo. The line has three fields, separated by tabs: the Unix time of the last commit, the folder name, and the text to show.

The script calls `row` for each subfolder that holds a `.git` directory. The command `sort -rn` sorts the lines by the first field, newest first. A repo with no commit has the time `0` and goes to the end.

The script sends the sorted lines to `fzf`. The option `--with-nth 3` shows only the third field. The placeholder `{2}` is the folder name. The preview and the two key bindings use `{2}` to find the repo folder.

The `Ctrl-T` binding starts `xdg-terminal-exec` in the background with `setsid`, in the folder of the selected repo.

In the Herdr mode, `jq` joins the output of `herdr workspace list` and `herdr agent list` by the workspace ID. It prints one line for each workspace. The line has four fields, separated by tabs: the sort rank of the status, the workspace ID, the pane ID, and the text to show. A workspace with no agent has `-` as the pane ID, and the preview stays empty. The script sorts the lines by the first field and sends them to `fzf`.

## License

Watcher uses the GNU General Public License, version 3 or any later version. The file `LICENSE` has the full text.
