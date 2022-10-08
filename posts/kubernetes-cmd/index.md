# 镜像和容器的Entrypoint和CMD


### Entrypoint和CMD

|  | No ENTRYPOINT | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT ["exec_entry", "p1_entry"]
| :-----| :----: | :----: | :----: |
| __No CMD__ | error, not allowed | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry |
| __CMD ["exec_cmd", "p1_cmd"]__ | exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd |
| __CMD ["p1_cmd", "p2_cmd"]__ | p1_cmd p2_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd |
| __CMD exec_cmd p1_cmd__ | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

### 镜像和容器

| 镜像 Entrypint | 镜像 Cmd | 容器 command | 容器 args | 执行命令|
| :-----: | :----: | :----: | :----: | :----: |
| [/ep-1] | [foo bar] | <not set> | <not set> | [ep-1 foo bar] |
| [/ep-1] | [foo bar] | [/ep-2]  | <not set> | [ep-2] |
| [/ep-1] | [foo bar] | <not set> | [zoo boo ]| [ep-1 zoo boo] |
| [/ep-1] | [foo bar] | [/ep-2] | [zoo boo ]| [ep-2 zoo boo] |
