# 迅雷远程下载服务（Docker）

[![Docker Pulls](https://img.shields.io/docker/pulls/swz128/xunlei.svg)](https://hub.docker.com/r/swz128/xunlei)
<!-- [![Docker Version](https://img.shields.io/docker/v/swz128/xunlei)](https://hub.docker.com/r/swz128/xunlei) -->
[![GitHub Stars](https://img.shields.io/github/stars/swz128/xunlei)](https://star-history.com/#swz128/xunlei&Date)

**Fork自 [cnk3x/xunlei](https://github.com/cnk3x/xunlei/tree/docker)，感谢作者**

从迅雷群晖套件中提取出来用于其他设备的迅雷远程下载服务程序。仅供测试，测试完请大家自觉删除。

下载保存目录 `/xunlei/downloads`， 对应迅雷应用内显示的下载路径是 `/downloads` 或者 `/迅雷下载`

下载后的文件保存在 `/mnt/xunlei/downloads`

可以通过 `ln -s /mnt/xunlei/downloads $HOME/Downloads` 来链接到下载目录，方便使用

## 更新

latest: 3.21.0

~~latest: 3.7.1~~

~~3.7.1: 已尝试修复插件的运行环境，大家可以尝试安装测速和网心云插件试试。~~

## 安装

[容器镜像: swz128/xunlei](https://hub.docker.com/r/swz128/xunlei)

阿里云镜像（国内访问）: registry.cn-shenzhen.aliyuncs.com/swz128/xunlei:latest

[源码仓库: https://github.com/swz128/xunlei/tree/docker](https://github.com/swz128/xunlei/tree/docker)

- 环境变量 `XL_WEB_PORT`: 网页访问端口，默认 `2345`。
- 环境变量 `XL_WEB_ADDRESS` 绑定端口，默认 `:port`
- 环境变量 `XL_DEBUG`: 1 为调试模式，输出详细的日志信息，0: 关闭，不显示迅雷套件输出的日志，默认0.
- 环境变量 `UID`, `GID`, 设定运行迅雷下载的用户，使用此参数注意下载目录必须是该账户有可写权限。 <https://github.com/cnk3x/xunlei/issues/85>
- 环境变量 `XL_BA_USER` 和 `XL_BA_PASSWORD`: 给迅雷面板添加基本验证（明码）。 <https://github.com/cnk3x/xunlei/issues/57>
- `host` 网络下载速度比 `bridge` 快, 如果没有条件使用host网络，映射`XL_WEB_PORT`设定的端口`tcp`即可。
- 下载保存目录 `/xunlei/downloads`, 数据目录：`/xunlei/data`, 请持久化。
- `hostname`: 迅雷会以主机名来命名远程设备，你在迅雷App上看到的就是这个。
- ~~安装好绑定完后可以在线升级到迅雷官方最新版本~~ 这点不确定了，得自己尝试。


### docker compose（推荐）

开启命令：`docker compose -f compose-latest.yml up -d`

`-f`：Compose configuration files
`up`：创建并启动容器
`-d`：后台运行

然后在浏览器中打开：http://localhost:4321

```yaml
# host更改端口 4321
# compose-latest.yml
services:
  xunlei:
    image: swz128/xunlei:latest
    privileged: true
    container_name: xunlei
    hostname: mynas
    network_mode: host
    environment:
      - XL_WEB_PORT=4321
    volumes:
      - /mnt/xunlei/data:/xunlei/data
      - /mnt/xunlei/downloads:/xunlei/downloads
    restart: unless-stopped
```

### docker shell

```bash
# 以下以 /mnt/sdb1/downloads 为实际的下载保存目录 /mnt/sdb1/xunlei 为实际的数据保存目录 为例
# 根据实际情况更改
# 如果已经安装过的(/mnt/sdb1/xunlei 目录已存在)，再次安装会复用，而且下载目录不可更改，如果要更改下载目录，请把这个目录删掉重新绑定。

# 国内访问将 swz128/xunlei:latest 替换为 registry.cn-shenzhen.aliyuncs.com/swz128/xunlei:latest

# host网络，更改端口为 4321
docker run -d --name=xunlei --hostname=mynas --net=host -e XL_WEB_PORT=4321 -v /mnt/xunlei/data:/xunlei/data -v /mnt/xunlei/downloads:/xunlei/downloads --restart=unless-stopped --privileged swz128/xunlei:latest
```

## 镜像构建流程

1. 前往迅雷NAS版本官网下载最新版本的迅雷套件，下载地址：<https://nas.xunlei.com/>
2. 将1中下载的迅雷套件放置于`spk`目录下
3. 原构建脚本报错，需要手动编译 golang 代码：（[golang环境搭建](https://www.runoob.com/go/go-environment.html)）
   ```shell
   $ cd xlp/
   $ go build -v -ldflags '-s -w -extldflags "-static"' -tags netgo
   ```
4. 执行`docker build -t xxx/xunlei --build-arg TARGETARCH={amd64|arm64} .`构建镜像

## 已知问题

- [x] ~~插件可能无法使用~~
- [ ] go 代码部分需要手动编译

## Used By

[kubespider](https://github.com/jwcesign/kubespider/blob/main/docs/zh/user_guide/thunder_install_config/README.md)
