# Hostloc Auto Get Points Docker 容器

这个项目包含了一个 Docker 容器，用于定期运行 Python 脚本 `hostloc_auto_get_points.py`。该脚本用于自动登录 Hostloc 网站并执行特定操作以获取积分。

## 步骤 1: 准备 Dockerfile

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

## 步骤 2: 构建 Docker 镜像

在 Dockerfile 所在目录运行以下命令构建 Docker 镜像：

```bash
docker build -t goordonchen/hostloc-bot:latest .
```

## 步骤 3: 上传镜像至 Docker Hub

1. **登录 Docker Hub**

    使用您的 Docker Hub 用户名和密码登录。请确保将 `USER` 替换为您的 Docker Hub 用户名，`PSW` 替换为您的 Docker Hub 密码。

    ```bash
    echo 'PSW' | docker login --username USER --password-stdin
    ```

2. **推送镜像**

    将构建好的镜像推送到 Docker Hub：

    ```bash
    docker push goordonchen/hostloc-bot:latest
    ```

## 步骤 4: 配置 Python 脚本和定时任务

1. **配置脚本**

    - 将 `hostloc_auto_get_points.py` 脚本放在 `/root/hostloc` 目录下。
    - 确保脚本中的用户名和密码已正确配置：

        ```python
        username = "你的账户"
        password = "你的密码"
        ```

        替换 `"你的账户"` 和 `"你的密码"` 为实际的 Hostloc 账号和密码。

2. **配置定时任务**

    - 在 `/root/hostloc` 目录下创建名为 `crontab.conf` 的文件以配置定时任务。例如，要每天凌晨 1 点执行脚本，可以这样配置：

        ```
        0 1 * * * python /usr/src/app/hostloc_auto_get_points.py >> /usr/src/app/hostloc.log 2>&1
        ```
        后续可在 `/root/hostloc` 目录下hostloc.log中查看运行日志。

## 步骤 5: 运行容器

使用以下命令从 Docker Hub 上拉取并运行最新版的 `hostloc-bot` 容器：

```bash
docker pull goordonchen/hostloc-bot:latest
docker run -d --name my-hostloc-bot -v /root/hostloc:/usr/src/app goordonchen/hostloc-bot:latest
```

## 感谢

本项目使用了 [Jox2018](https://github.com/Jox2018) 的 [hostloc_getPoints](https://github.com/Jox2018/hostloc_getPoints) 脚本。

---
