---
title: "HEADLESS SERVER"
bookToc: false
bookFlatSection: true
---
# Tips To Work With Headless Servers

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

### 2.2. Multi-process Mode
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






