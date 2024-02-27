# Windows

## Symbolic link

To create a symbolic link, open the powershell in administrator mode and type something like

```bash
New-Item -ItemType SymbolicLink -Path C:\Users\Me\.cache\gpt4all\mistral-7b-instruct-v0.1.Q4_0.gguf -Target D:\second-home\.cache\gpt4-all\mistral-7b-instruct-v0.1.Q4_0.gguf
```
