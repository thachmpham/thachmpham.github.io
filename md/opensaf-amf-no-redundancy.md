<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>


---
title:  'OpenSAF AMF - High Available SNMP'
---


# Create Daemon Program
Create directory `/opt/demo`.
```sh
mkdir /opt/demo
```

Create file `/opt/demo/loop.sh`.
```sh

counter=1

while true; do
    logger "hello $counter"
    ((counter++))
    sleep 5
done

```

Create file `/opt/demo/main.sh`.
```sh

program='/opt/demo/loop.sh'
pidfile='/var/run/demo.pid'

start() {
    echo 'starting...'

    start-stop-daemon --start --background \
        --make-pidfile --pidfile $pidfile \
        --exec /bin/bash -- $program

    echo 'started'
}

stop() {
    echo 'stopping...'

    start-stop-daemon --stop --pidfile $pidfile \
        --retry 5

    echo 'stopped'
}

status() {
    if [ -f $pidfile ] \
        && start-stop-daemon --status --pidfile $pidfile;\
    then
        echo "running, pid: $(cat $pidfile)"
    else
        echo 'not running'
    fi
}

case $1 in
    start)
        start;;
    stop)
        stop;;
    status)
        status;;
    *)
        echo 'Usage: main.sh {start|stop|status}'
        exit 1;;
esac

exit 0

```
