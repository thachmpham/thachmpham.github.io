---
title: "Debug"
bookToc: true
bookFlatSection: true
---
# Debug Tricks

## 1. Jupyter
Generate configuration.
```sh
$ jupyter lab --generate-config
Writing default config to: /root/.jupyter/jupyter_lab_config.py
```

Edit configuration `/root/.jupyter/jupyter_lab_config.py`.
```py
c.ServerApp.allow_root = True

c.ServerApp.open_browser = False
c.ExtensionApp.open_browser = False

c.ServerApp.ip = '0.0.0.0'

c.NotebookApp.token = ''
c.NotebookApp.password = ''
```

Start jupyter lab.
```sh
$ jupyter lab
```

Find the server address.
```sh
$ lsof -P -i -n | grep jupyter
jupyter-l 6857            root    6u  IPv4  46928      0t0  TCP *:8888 (LISTEN)
```


## 2. GDB Server
### 2.1. Launch A Program
**Server**: Starts a `gdbserver` instance that listens at `172.16.111.130:2000` and allows remote debug `/usr/bin/ls` program.
```sh
$ gdbserver 172.16.111.130:2000 /usr/bin/ls
```

**Client (Option 1)**: Connect to the remote target debugger with `gdb extended-remote`.
```sh
$ gdb
(gdb)	target extended-remote 172.16.111.130:2000
(gdb)	b main
(gdb)	r
Breakpoint 1, main (argc=1, argv=0xfffffffff358) at ../src/ls.c:1649
(gdb)	c
(gdb)	monitor exit
```

**Client (Option 2)**: Connect to the remote target debugger with `gdb remote`.
```sh
$ gdb
(gdb)	target remote 172.16.111.130:2000
(gdb)	b main
(gdb)	c
Breakpoint 1, main (argc=1, argv=0xfffffffff358) at ../src/ls.c:1649
```

References:
- [GDB `extended-remote` vs `remote`](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html#:~:text=With%20target%20remote%20mode%3A%20When,though%20no%20program%20is%20running.)


### 2.2. Attach To A Running Process
**Server**: Starts `/usr/bin/vim` program.
```sh
$ /usr/bin/vim
$ ps aux | grep vim
root       13368  0.2  0.1  16180  9836 pts/3    Sl+  18:02   0:00 /usr/bin/vim
```

**Server**: Starts a `gdbserver` instance that listens at `172.16.111.130:2000` and allows remote debug process `13368`.
```sh
$ gdbserver --attach 172.16.111.130:2000 13368
```

**Client**: Connect to the remote target debugger.
```sh
$ gdb
(gdb)	target extended-remote 172.16.111.130:2000
(gdb)	bt
#14 0x0000aaaabb2343f4 [PAC] in vim_main2 () at /usr/src/debug/vim-9.1.181-1.fc39.aarch64/src/main.c:895
#15 main (argc=<optimized out>, argv=<optimized out>) at /usr/src/debug/vim-9.1.181-1.fc39.aarch64/src/main.c:441
```

### 2.3. Multi-process Mode
**Server**: Starts the GDB server in multi-process mode and listens at `172.16.111.130:2000`.
```sh
$ gdbserver --multi 172.16.111.130:2000
```

**Client (Sample 1)**: Launch program `/usr/bin/vim`.
```sh
$ gdb
(gdb)	target extended-remote 172.16.111.130:2000
(gdb)	set remote exec-file /usr/bin/vim
(gdb)	file /usr/bin/vim
(gdb)	b main
(gdb)	r
```

**Client (Sample 2)**: Attach to the running process `13615`.
```sh
$ gdb
(gdb)	target extended-remote 172.16.111.130:2000
(gdb)	attach 13615
(gdb)	bt
```

## 3. VS Code Remote Debug
Extension: [C/C++ for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).  
Parameter: [`launch.json` reference](https://code.visualstudio.com/docs/cpp/launch-json-reference).

### 3.1. Launch A Program
**Server**: Starts a `gdbserver` instance that listens at `172.16.111.132:2000` and allows remote debug `/usr/bin/ls` program.
```sh
$ gdbserver 172.16.111.132:2000 /usr/bin/ls
```

**Client**: Configure `launch.json`.
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            // -----    remote target settings: begin     ----- //
            "miDebuggerServerAddress": "172.16.111.132:2000",   // gdbserver address
            // -----    remote target settings: end       ----- //


            // -----    local host settings: begin  ----- //
            "program": "/usr/bin/ls",
            "stopAtConnect": true,
            // -----    local host settings: end    ----- //


            // -----    general settings: begin ----- //
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "stopAtEntry": true,
            "cwd": "${fileDirname}",
            "externalConsole": false,
            "MIMode": "gdb",
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
            ],
            // -----    general settings: end   ----- //
        }
    ]
}
```

### 3.2. Attach To A Running Process
**Server**: Starts `/usr/bin/vim` program.
```sh
$ /usr/bin/vim
$ ps aux | grep vim
root        1758  0.2  0.4  16364  9752 pts/0    Sl+  16:59   0:00 /usr/bin/vim
```

**Server**: Starts a `gdbserver` instance that listens at `172.16.111.132:2000` and allows remote debug process `13368`.
```sh
$ gdbserver --attach 172.16.111.132:2000 1758
```

**Client**: Configure `launch.json`.
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            // -----    remote target settings: begin     ----- //
            "miDebuggerServerAddress": "172.16.111.132:2000",   // gdbserver address
            // -----    remote target settings: end       ----- //


            // -----    local host settings: begin  ----- //
            "program": "/usr/bin/vim",
            "useExtendedRemote": true,
            "miDebuggerPath": "/usr/bin/gdb",
            // -----    local host settings: end    ----- //


            // -----    general settings: begin ----- //
            "name": "(gdb) Attach",
            "type": "cppdbg",
            "request": "attach",
            "MIMode": "gdb",
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
            // -----    general settings: end   ----- //
        }
        
    ]
}
```

### 3.3. Multi-process Mode
**Server**: Starts the GDB server in multi-process mode and listens at `172.16.111.132:2000`.
```sh
$ gdbserver --multi 172.16.111.132:2000
```

**Client**: Setup `launch.json` to attach to a running process.
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            // -----    remote target settings: begin     ----- //
            "miDebuggerServerAddress": "172.16.111.132:2000",   // gdbserver address
            // -----    remote target settings: end       ----- //


            // -----    local host settings: begin  ----- //
            "program": "/usr/bin/vim",
            "useExtendedRemote": true,
            "miDebuggerPath": "/usr/bin/gdb",
            // -----    local host settings: end    ----- //


            // -----    general settings: begin ----- //
            "name": "(gdb) Attach",
            "type": "cppdbg",
            "request": "attach",
            "MIMode": "gdb",
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
            // -----    general settings: end   ----- //
        }
        
    ]
}
```