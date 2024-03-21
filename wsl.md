# Windows Subsystem for Linux

## Move a distribution to another device

From the [official documentation](https://learn.microsoft.com/en-us/windows/wsl/basic-commands) (**important** notes below):

```powershell
# Create a folder where you would like to store your distro
New-Item -ItemType Directory -Path <Install location, e.g: D:\WSLDistros\Ubuntu>

# Export your distro to that folder as a VHD
wsl --export --vhd <Distroname, e.g: Ubuntu> <Install Location with filename, e.g: D:\WSLDistros\Ubuntu\ext4.vhdx>

# Unregister your old distro
# Please note this will erase your existing distro's file contents, please ensure the backup file you created in the 2nd step is present at the location and that the export operation completed successfully.
# Please exercise caution when using this command, as it is destructive and could cause data loss.
wsl --unregister <Distroname>

# Import your VHD backup
wsl --import-in-place <Distroname> <Install Location with filename>
```

Note that two important points are missing:

1. Before starting, one has to shut down wsl via `wsl --shutdown` in your PowerShell. Maybe, you want to quit Docker Desktop or other services before for a clean shutdown.
1. When importing, the default user will be `root` which might deviate from the original config. To mitigate, run `whoami` in the old distro to be deleted, note the name and finally set the default user for the distro via `cd $env:USERPROFILE\AppData\Local\Microsoft\WindowsApps` and `ubuntu config --default-user whoami_result`.

**Alternative** method (but similar):

This instruction is copied from [equiman](https://dev.to/equiman/move-wsl-file-system-to-another-drive-2a3d). We want to move our standard distro `Ubuntu` (see all distros via `wsl --list --verbose`) to device `d`.

1. Check which user is the default user in the distro via typing `whoami` in the distro you want to move.
2. Shut down wsl via `wsl --shutdown` in your PowerShell. Maybe, you want to quit Docker Desktop or other services before for a clean shutdown.
3. Backup the distro via `wsl --export Ubuntu D:\backup\ubuntu.tar` and check if its really there. Hint: Windows does not like throwing files at the top level and you will usually encounter an `access denied` error. Make sure that your path exists.
4. Now remove the distro from wsl: `wsl --unregister Ubuntu`. This is destructive.
5. Import the backup via `wsl --import Ubuntu D:\wsl D:\backup\ubuntu.tar`.
6. Set the default user for the distro via `cd $env:USERPROFILE\AppData\Local\Microsoft\WindowsApps` and `ubuntu config --default-user whoami_result`
7. Optional: If your default distro was moved, you might have to set it again the default distro. Execute `wsl --list` to see the which current distro is the default one and change with `wsl --setdefault my_favorite_distro` the new default one. This might be especially necessary for Docker integration with the default distro. If on restart, docker is not accepted by your moved distro, consider enabling it manually in Docker Desktop/Resources/WSL integration

## WSL does not boot

If `wsl` hangs (even `wsl -l -v`), killing the processes of `Windows Subsystem for Linux` in the `Task Manager` and restarting wsl (e.g. via starting the distro) *might* help.

## Clipboard

WSL and Windows clipboard seem to be two separate things.
To communicate with the Windows clipboard from wsl, one can use these aliases in one's `.bashrc`:

```bash
# alias to communicate with the windows clipboard
# copy into clip, used as `echo "Hello" | pbcopy"`
alias pbcopy='clip.exe'
# paste to stdout from clipboard, remove carriage
alias pbpaste="powershell.exe -command 'Get-Clipboard' | tr -d '\r'"
```

## Trivia

- [official commands](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)
