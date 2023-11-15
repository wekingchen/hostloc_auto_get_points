# Hostloc Auto Get Points Docker 容器

这个项目包含了一个 Docker 容器，用于定期运行 Python 脚本 `hostloc_auto_get_points.py`。该脚本用于自动登录 Hostloc 网站并执行特定操作以获取积分。

## 构建镜像

1. **准备 Dockerfile**

    首先，确保你的项目目录中有一个 Dockerfile。Dockerfile 应该像下面这样：

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

    在包含 Dockerfile 的目录下运行以下命令来构建 Docker 镜像：

    ```bash
    docker build -t hostloc-bot .
    ```

    这将创建一个名为 `hostloc-bot` 的镜像。

## 配置定时任务文件

在你的 `/root/hostloc` 目录下创建一个名为 `crontab.conf` 的文件。这个文件将包含定时运行脚本的配置。例如：

```
* * * * * python /usr/src/app/hostloc_auto_get_points.py >> /usr/src/app/hostloc.log 2>&1
```

这个例子中的 cron 表达式表示每分钟执行一次脚本。根据你的需要，可以调整这个表达式。

## 运行容器

运行以下命令来启动 Docker 容器：

```bash
docker run -d --name my-hostloc-bot -v /root/hostloc:/usr/src/app hostloc-bot
```

这个命令将启动一个名为 `my-hostloc-bot` 的容器，同时将宿主机的 `/root/hostloc` 目录映射到容器内的 `/usr/src/app`。

## 更新和维护

如果你需要更新 Python 脚本或 `crontab.conf` 配置，只需在宿主机上修改相应文件。由于使用了卷映射，更改将自动反映到容器中。但请注意，更改 `crontab.conf` 后，可能需要重启容器或手动重新加载 `cron` 服务。

---
