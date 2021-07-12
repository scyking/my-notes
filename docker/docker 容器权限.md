# docker 容器权限

---

[toc]

## 概述

> 出于容器之间和容器与宿主机的安全隔离保护，在默认的Docker配置下，Docker容器是没有系统权限的。

## 配置[权限](../linux/linux%20capability.md)

### [docker run](https://docs.docker.com/engine/reference/commandline/run/) 参数

```bash
--cap-add list                   Add Linux capabilities
--cap-drop list                  Drop Linux capabilities
--privileged                     Give extended privileges to this container
```

### [docker-compose](https://docs.docker.com/compose/compose-file/compose-file-v3/#cap_add-cap_drop) 配置

```yml
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```
