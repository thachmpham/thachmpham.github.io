---
title: "lsof"
---

:::::::::::::: {.columns}
::: {.column width=60%}

| Process                           | Example                   | Explain                                   |
|:----------------------------------|:--------------------------|:------------------------------------------|
| Filter by process pid             | `-p 123`                  | Show files opened by pid 123              |
|                                   | `-p ^123`                 | Show files opened by pid != 123           |
| Filter by process command         | `-c rsyslogd`             | Show files opened by rsyslogd             |
|                                   | `-c ^rsyslogd`            | Show files opened by comm != rsyslogd     |
| Filter by command regex           | `-c '/rsys.*/'`           | Show files opened by rsys.*               |
|                                   | `-c '/rsyslogd|vi/'`      | Show files opened by rsyslogd or vi       |

| Path                              | Example                   | Explain                                   |
|:----------------------------------|:--------------------------|:------------------------------------------|
| Filter by file name               | `lsof hello.txt`          | Show processes open hello.txt             |
| Filter by directory               | `+d /var/log`             | Show processes access /var/log directly   |
| Filter by directory recursively   | `+D /var/log`             | Show processes access /var/log recursively|
| Filter by file descriptor         | `-d 3`                    | Show processes have fd 3                  |

:::
::: {.column width=40%}

| Socket                            | Example                   | 
|:----------------------------------|:--------------------------|
| Filter by address                 | `-i addr`                 |
| Filter by IP                      | `-i 127.0.0.1`            |
| Filter by protocol                | `-i TCP`                  |
| Filter by port                    | `-i :8080`                |
| Unix domain socket                | `-U`                      |
| Numeric address                   | `-n`                      |

Address format:  
`[46][protocol][@hostname|hostaddr][:service|port]`

| AND operation                     | Explain                   |
|:----------------------------------|:--------------------------|
| `-c rsyslog -a -d 1`              | comm=rsyslogd and fd=1    |

:::
::::::::::::::