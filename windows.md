# Windows

## Account information

In the windows command shell or PowerShell, type `net user myUserNam /domain` to see password information like expiry as well as group memberships of your user.

## Symbolic link

To create a symbolic link, open the powershell in administrator mode and type something like

```bash
# PowerShell
New-Item -ItemType SymbolicLink -Path C:\Users\Me\.cache\gpt4all\mistral-7b-instruct-v0.1.Q4_0.gguf -Target D:\second-home\.cache\gpt4-all\mistral-7b-instruct-v0.1.Q4_0.gguf
# Command Prompt
mklink /D Link Target
```
