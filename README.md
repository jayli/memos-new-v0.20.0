### 解决 memos v0.20.0 的 content too long 的问题

> <https://github.com/usememos/memos/issues/2826>

我改了源码，重新打了 docker 的镜像，一个是给 arm 架构 openwrt 用的，一个是在 x86 平台上用的。使用对应平台的镜像解压成 tar 包后自己导入 docker 即可。

### 修改方法：

我在本地 MacBook 上重新打包：

一，配置好国内的源，docker desktop 的 setting 中的 Docker Engine 里的 json 文件里加上，

    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://ccr.ccs.tencentyun.com"
    ]

然后check出源代码：

    git clone git@github.com:usememos/memos.git

二， checkout 到 v0.20.0 版本，然后修改限制：`api/v2/memo_services.go` 文件的第 30 行

构建 Mac 包：

    docker build -t memos .

构建 openwrt 包：

    cd memos
    docker buildx build --platform linux/arm64/v8 -t memos .

如果遇到网络问题，给 Dockerfile 中加上代理和国内的源，修改 Dockerfile 对应的行：

    RUN corepack enable && pnpm i --frozen-lockfile --registry=https://registry.npmmirror.com
    RUN CGO_ENABLED=0 && go env -w GOPROXY=https://goproxy.cn && go build -o memos ./bin/memos/main.go

构建完成后就自动导入到了本机的 docker 中了。
    
三， 从 docker 导出镜像到 `memo_v0.20.0.tar` 里: 

    docker save -o memo_v0.20.0.tar memos

四，导入镜像到 Docker

    docker load < memo_v0.20.0.tar

五， 启动 docker 镜像：

    docker run --init -d --restart=unless-stopped --publish 5320:5230 -v ~/Memos_tmp:/var/opt/memos --name memos_new memos:latest ./memos

