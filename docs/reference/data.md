# Where the app stores data

## On your computer

The app keeps its data outside the application folder, so replacing the
application with a newer release keeps it.

| Platform | Folder |
|---|---|
| Windows | `%LOCALAPPDATA%\ACE3P-GUI` |
| Linux, Perlmutter, S3DF | `~/.local/share/ACE3P-GUI` |

The folder is readable only by you. It holds:

| Item | Content |
|---|---|
| `jobs.db` | The job registry: jobs, names, notes, favorites, trash. |
| `uploads/` | Meshes prepared for the Mesh Viewer. |
| `app_config.json` | Remembered paths to Cubit and ParaView. |
| `ai-agent/` | AI Assistant settings, your instructions, your guides, saved chats, and API keys if you chose to save them. |

The browser keeps small preferences (current solver, the workflow builder
draft, panel sizes, the last post-processing directory) in its local storage
for this page.

Passwords are never stored.

## On the cluster

| Item | Location |
|---|---|
| Job files | The run directory you chose: mesh, `input.<solver>`, `submit.sbatch`, logs `<solver>-<jobid>.out/.err`, and results. |
| Job history state | `~/.ace3p_gui_state.json` in your cluster home directory: names, notes, favorites and deletions, shared by every computer you use. |
| Post-processing files | `.ace3p-pp-*` files in the working directory, removed by **Clean up**. |
| Download archive | `.ace3p_<job>_results.tar.gz` next to the run directory, removed after the download. |

## Remove everything

1. Delete the application file or folder.
2. Delete the data folder listed above.
3. On Linux, run `./install-shortcut.sh --remove` if you created the shortcut.
4. Optionally delete `~/.ace3p_gui_state.json` on each cluster.
