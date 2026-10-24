# Install and launch

ACE3P GUI ships as one standalone download per platform. It contains Python,
every dependency and the interface. You do not install anything else.

Download the release from
[github.com/AlexxLauu/ACE3P-GUI-Project/releases](https://github.com/AlexxLauu/ACE3P-GUI-Project/releases).

| You are on | Download | Run |
|---|---|---|
| Windows | `ACE3P-GUI-v<version>-windows-x86_64.exe` | Double-click it |
| Perlmutter (NERSC) | `ACE3P-GUI-v<version>-linux-x86_64.tar.gz` | Unpack, run `./ACE3P-GUI` |
| S3DF (SLAC) | `ACE3P-GUI-v<version>-linux-x86_64.tar.gz` | Unpack, run `./ACE3P-GUI` |
| Linux desktop | `ACE3P-GUI-v<version>-linux-x86_64.tar.gz` | Unpack, run `./ACE3P-GUI` |
| macOS | No prebuilt binary | [Run from source](#run-from-source) |

The application starts a small local server and opens the interface in your
default web browser. The console or terminal window that opens with it **is**
the application: close it to stop the app.

## Windows

1. Download the `.exe`.
2. Double-click it.
3. If Windows SmartScreen shows **Windows protected your PC**, click
   **More info**, then **Run anyway**. The warning appears because the file is
   not code-signed.
4. Your browser opens on the app. You are signed in to the app automatically
   through a one-time link.

To update, close the browser tab and the console window, then run the newer
`.exe`. Job history and settings are stored separately and are kept.

## Perlmutter and S3DF

Run the app on the cluster from a graphical desktop: **ThinLinc** on Perlmutter,
**NoMachine** on S3DF. Open a terminal on that desktop and run:

```bash
tar xzf ACE3P-GUI-v<version>-linux-x86_64.tar.gz
cd ACE3P-GUI-v<version>-linux-x86_64
./ACE3P-GUI
```

Firefox opens on the app. You do not sign in: the app detects that it runs on
the cluster and uses your own account, file system and Slurm directly. The
header shows a badge with the cluster name and your user name.

Create a desktop icon for the next launch:

```bash
./install-shortcut.sh
```

This adds an **ACE3P GUI** icon to the desktop and the applications menu and an
`ace3p-gui` command in `~/.local/bin`. Run it again after replacing the folder
with a newer release. Run `./install-shortcut.sh --remove` to remove it.

!!! tip "Sign in to the other cluster"
    Running on Perlmutter fills the NERSC slot with no sign-in. You can still
    sign in to S3DF from the same window (and the reverse on S3DF), so a study
    prepared on one cluster can be submitted to the other.

### Over plain SSH

With no graphical desktop, the app prints an SSH tunnel command when it starts.
Run that command on your own computer, then open the printed address in your
browser:

```bash
ssh -L 8123:localhost:8123 you@perlmutter.nersc.gov
```

Cubit and ParaView do not open this way. Use ThinLinc or NoMachine for them.

### Single-file build

Each release also has `ACE3P-GUI-v<version>-linux-x86_64.bin`, the same app
as one file. It unpacks itself under `~/.cache/ACE3P-GUI` on every launch, so it
starts more slowly than the unpacked folder.

## Command-line options

```text
ACE3P-GUI                  pick a free port, open a browser
ACE3P-GUI --port 8500      use exactly this port
ACE3P-GUI --no-browser     start the server only
ACE3P-GUI --host 0.0.0.0   listen beyond this computer
ACE3P-GUI --help           list every option
```

The app binds the first free port at or above 8000, so several users can run it
on one shared login node. The port in use is printed at start-up.

!!! warning
    `--host 0.0.0.0` makes the app reachable from other machines. On a shared
    login node that means every other user on the node. Use it only on a
    private workstation.

## Access link and token

Every launch creates a secret access token. The app opens your browser on a
link that carries it, and the browser keeps it as a cookie. A browser tab
without the token gets **401 Unauthorized**. If you open the app in a second
browser, copy the full link printed in the console:

```text
[STARTUP] open http://127.0.0.1:8000/?token=...
```

## Run from source

Use this on macOS or to work on the code. It needs Python 3.9 or newer and `git`.

=== "Linux, macOS, clusters"

    ```bash
    git clone https://github.com/AlexxLauu/ACE3P-GUI-Project.git
    cd ACE3P-GUI-Project
    ./install.sh
    ```

=== "Windows"

    ```powershell
    git clone https://github.com/AlexxLauu/ACE3P-GUI-Project.git
    cd ACE3P-GUI-Project
    .\install.bat
    ```

The installer builds a virtual environment inside the clone, installs the app,
and creates the same desktop shortcut as the release. Update with
`./install.sh --update`.

## Update

Replace the `.exe` or the unpacked folder with the newer release. Job history,
uploads and settings live outside the application and are kept:

- Windows: `%LOCALAPPDATA%\ACE3P-GUI`
- Linux and the clusters: `~/.local/share/ACE3P-GUI`

If the page looks outdated after an update, press
++ctrl+shift+r++ (macOS: ++cmd+shift+r++) to reload it without the cache.
