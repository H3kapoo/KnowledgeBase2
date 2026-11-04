# VSCode / VSCodium

### 1. [Creating a debug task](#creating-a-debug-task)
A ```.vscode``` folder needs to be created at the root.
Inside it, create ```launch.json``` and ```tasks.json```. <br/>
Minimal examples bellow for both files, some options are optional. <br/>
Tasks.json is also optional if you don't need a prelaunch setup to run before the tested program.
```json
// launch.json
{
    "version": "0.2.0",
    "configurations": [
    {
      "name": "name_of_the_debug_session",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/path_to_program_to_debug",
      "args": ["additional_command_line", "arguments"],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "task_name" // task to be run before launch
    }
    ]
}
```
```json
// tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "task_name",
            "type": "shell",
            "command": "command_to_be_ran"
        }
    ]
}
```
After setting these up, a new option to debug in the Debug menu should appear named ```name_of_the_debug_session```.

### 2. [Clangd for VSCodium](#clangd-for-vscodium)
When using Vscodium, usual Microslop extenstions for C/C++ intellisense are not available. While those can be reintroduced
manually via some configuration, it defeats the purpose of the program. <br/>
Clangd extension take the wheel here as the extension to install. It needs to install ```clang``` locally though. <br/>
Clangd has it's own config file ```.clangd``` at the project root that can configure the extension. <br/>
The config file has a TOML structure:
```clangd
CompileFlags:
  Add: [-std=c++23] # Change c++ standard used by clang
```

### 3. [Install VSCodium](#install-vscodium)
Install directly from the terminal:
```bash
$ snap install codium --classic
```

## References Used
- https://vscodium.com/
