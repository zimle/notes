# Windows Subsystem for Linux

## Move a distribution to another device

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
