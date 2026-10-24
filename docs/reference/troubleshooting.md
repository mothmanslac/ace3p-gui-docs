# Troubleshooting

Open **View → Messages** first. Every error the app reports is listed there,
most with an expandable detail.

## Starting the app

| Problem | Fix |
|---|---|
| Windows: **Windows protected your PC** | Click **More info**, then **Run anyway**. |
| Windows: nothing happens on double-click | Windows Defender scans the file on first run; wait a moment. If a console opens and closes, run the `.exe` from a Command Prompt to read the message. |
| Linux: `Permission denied` | `chmod +x ACE3P-GUI install-shortcut.sh` |
| Linux: `GLIBC_2.xx not found` | The build is newer than the machine. Ask for a build made on an older base system. |
| Desktop icon does nothing (GNOME) | Right-click it and choose **Allow Launching**. |
| `ace3p-gui: command not found` | Add `export PATH="$HOME/.local/bin:$PATH"` to `~/.bashrc`. |
| `No free port between 8000 and …` | Start with `--port <number>` and a free port. |
| Browser shows **401 Unauthorized** | The tab has no access token. Open the full link printed in the console (`/?token=...`). |
| The page looks outdated after an update | Reload with ++ctrl+shift+r++. |

## Mesh

| Problem | Fix |
|---|---|
| Cubit not found | Install Cubit, set `ACE3P_CUBIT_EXE`, or use a `.gen`/`.ncdf`. The job still meshes on the cluster. |
| "No export line" message | Add `export genesis "<name>.gen" overwrite` at the end of the journal. |
| No boundary-condition circles | The journal defines no sidesets, or no mesh is selected. Type the sideset numbers into the fields. |
| Meshing never finishes | The journal waits for input or is very large. Test it in Cubit first. |

## Sign-in

| Problem | Fix |
|---|---|
| Wrong password on NERSC | Enter the OTP in its own field, not after the password. Wait for a new OTP code. |
| Duo push never arrives (S3DF) | Check Duo on your phone, then press **Authenticate** again. |
| Session expired | Sign in again from Settings. Sessions end after 8 idle hours or when the app restarts. |

## Submission

| Problem | Fix |
|---|---|
| **Run Simulation** stays grey | Hover over it: the tooltip lists what is missing (mesh, required fields, sbatch). |
| Run directory refused | A job of yours is still active in it. Use another directory. |
| `Invalid account` | Set your own account in Simulation Settings. |
| `Job violates accounting/QOS policy` | Lower walltime or nodes, or choose another QOS. |

## Job failed

1. Open the job with **View** and press **error** to read the `.err` log.
2. Look for the first error, not the last:

| Message in the log | Cause |
|---|---|
| `cubit: command not found` or `Cannot find the mesh file ./x.ncdf` | Meshing failed on the cluster. Check the Cubit lines in the `.out` log. |
| `DUE TO TIME LIMIT` | Walltime reached. Increase it. |
| `oom-kill`, `Out Of Memory` | Not enough memory. Add nodes, or lower `FiniteElement → Order`. |
| Parse error in the input file | A value or name the solver does not accept. Check additional parameters and hand edits. |

3. Ask the [AI Assistant](../ai/assistant.md): *Why did my most recent job
   fail?* It reads the logs and the input file for you.

## Report a problem

Open an issue on
[GitHub](https://github.com/AlexxLauu/ACE3P-GUI-Project/issues) with:

- the version from **Settings → Updates**,
- your platform and where the app runs (laptop, Perlmutter, S3DF),
- the steps that lead to the problem,
- the text from the Messages window.
