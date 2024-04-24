---
title: "REMOTE"
bookToc: true
bookFlatSection: true
---
# Working with a Server Remotely

## 1. Jupyter
**Server**
- Generate configuration.
```sh
$ jupyter lab --generate-config
Writing default config to: /root/.jupyter/jupyter_lab_config.py
```

- Edit configuration `/root/.jupyter/jupyter_lab_config.py`.
```py
c.ServerApp.allow_root = True

c.ServerApp.open_browser = False
c.ExtensionApp.open_browser = False

c.ServerApp.ip = '0.0.0.0'
c.ServerApp.port = 8888
```

**Client**
- Open browser: <server_ip>:8888


## 2. GDB Server & VS Code
### 2.1. Launch
**Server**
```sh
gdbserver <server_ip:port> program
```

**Client - VS Code**
- Edit `launch.json`.
```json
"configurations": [
	{
		"name": "(gdb) Launch",
		"type": "cppdbg",
		"request": "launch",
		"program": "/path/to/program",
		"args": [],
		"stopAtEntry": false,
		"cwd": "${fileDirname}",
		"environment": [],
		"externalConsole": false,
		"MIMode": "gdb",
		"miDebuggerServerAddress": "server_ip:port",
		"setupCommands": [
		    {
		        "description": "Enable pretty-printing for gdb",
		        "text": "-enable-pretty-printing",
		        "ignoreFailures": true
		    },
		    {
		        "description": "Set Disassembly Flavor to Intel",
		        "text": "-gdb-set disassembly-flavor intel",
		        "ignoreFailures": true
		    }
		]
	}
]
```
Essential parameters:
- `program`
- `miDebuggerServerAddress`

### 2.2. Attach
**Server**
```sh
gdbserver --multi <server_ip:port>
```

**Client**
```json
"configurations": [
	{
        "name": "(gdb) Attach",
        "type": "cppdbg",
        "request": "attach",
        "program": "/path/to/program",
        "MIMode": "gdb",
        "miDebuggerServerAddress": "server_ip:port",
        "miDebuggerPath": "/usr/bin/gdb",
        "useExtendedRemote": true,
        "setupCommands": [
            {
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            },
            {
                "description": "Set Disassembly Flavor to Intel",
                "text": "-gdb-set disassembly-flavor intel",
                "ignoreFailures": true
            }
        ]
    },
]
```
Essential parameters:
- `program`
- `miDebuggerServerAddress`
- `miDebuggerPath`
- `useExtendedRemote`






