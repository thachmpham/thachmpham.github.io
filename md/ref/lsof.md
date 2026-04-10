---
title: "lsof"
---

:::::::::::::: {.columns}
::: {.column}

| Process to file                   |                           |                                      |
|:----------------------------------|:--------------------------|:-------------------------------------|
| Filter by process pid             | `-p 123`                  | Files opened by pid 123              |
|                                   | `-p ^123`                 | Files opened by pid != 123           |
| Filter by process command         | `-c rsyslogd`             | Files opened by rsyslogd             |
|                                   | `-c ^rsyslogd`            | Files opened by comm != rsyslogd     |
| Filter by command regex           | `-c '/rsys.*/'`           | Files opened by rsys.*               |
|                                   | `-c '/rsyslogd|vi/'`      | Files opened by rsyslogd or vi       |

| File to process                   |                           |                                           |
|:----------------------------------|:--------------------------|:------------------------------------------|
| Filter by file name               | `lsof hello.txt`          | Processes open hello.txt             |
| Filter by directory               | `+d /var/log`             | Processes access /var/log directly   |
| Filter by directory recursively   | `+D /var/log`             | Processes access /var/log recursively|
| Filter by file descriptor         | `-d 3`                    | Processes have fd 3                  |

:::
::: {.column}

| AND operation                     |                           |
|:----------------------------------|:--------------------------|
| `-c rsyslog -a -d 1`              | comm=rsyslogd and fd=1    |

| Socket                            |                           | 
|:----------------------------------|:--------------------------|
| Filter by address                 | `-i addr`                 |
| Filter by IP                      | `-i 127.0.0.1`            |
| Filter by protocol                | `-i TCP`                  |
| Filter by port                    | `-i :8080`                |
| Unix domain socket                | `-U`                      |
| Numeric address                   | `-n`                      |

Address format:  
`[46][protocol][@hostname|hostaddr][:service|port]`

:::
::::::::::::::