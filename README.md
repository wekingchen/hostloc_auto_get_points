# Hostloc Auto Get Points Docker 容器

这个项目包含了一个 Docker 容器，用于定期运行 Python 脚本 `hostloc_auto_get_points.py`。该脚本用于自动登录 Hostloc 网站并执行特定操作以获取积分。

## 构建镜像

1. **准备 Dockerfile**

    确保你的项目目录中有一个 Dockerfile。Dockerfile 应该如下所示：

    ```Dockerfile
    FROM python:3.8-alpine

    WORKDIR /usr/src/app

    RUN apk add --no-cache gcc musl-dev libffi-dev openssl-dev

    RUN pip install --no-cache-dir requests pyaes

    RUN apk add --no-cache tzdata && \
        cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
        echo "Asia/Shanghai" > /etc/timezone && \
        apk del tzdata

    RUN echo "#!/bin/sh" > /entrypoint.sh
    RUN echo "crontab /usr/src/app/crontab.conf" >> /entrypoint.sh
    RUN echo "crond -f" >> /entrypoint.sh

    RUN chmod +x /entrypoint.sh

    CMD ["/entrypoint.sh"]
    ```

2. **构建 Docker 镜像**

    在 Dockerfile 所在目录运行以下命令构建 Docker 镜像：

    ```bash
    docker build -t hostloc-bot .
    ```

## 配置 Python 脚本

确保 `hostloc_auto_get_points.py` 脚本中的用户名和密码部分已正确配置：

```python
username = "你的账户"
password = "你的密码"
```

替换 `"你的账户"` 和 `"你的密码"` 为实际的 Hostloc 账号和密码。

## 配置定时任务文件

在 `/root/hostloc` 目录下创建名为 `crontab.conf` 的文件。这个文件包含定时任务配置，例如：

```
* * * * * python /usr/src/app/hostloc_auto_get_points.py >> /usr/src/app/hostloc.log 2>&1
```

## 运行容器

使用以下命令启动 Docker 容器：

```bash
docker run -d --name my-hostloc-bot -v /root/hostloc:/usr/src/app hostloc-bot
```

## 更新和维护

若需更新脚本或配置文件，直接在宿主机上进行修改。由于使用了卷映射，更改将自动反映到容器中。

## 感谢

本项目使用了 [Jox2018](https://github.com/Jox2018) 的 [hostloc_getPoints](https://github.com/Jox2018/hostloc_getPoints) 脚本。

---
