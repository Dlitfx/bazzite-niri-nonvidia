# bazzite-niri-nonvidia &nbsp; [![bluebuild build badge](https://github.com/dlitfx/bazzite-test/actions/workflows/build.yml/badge.svg)](https://github.com/dlitfx/bazzite-test/actions/workflows/build.yml)

关于如何基于本模板搭建你自己的仓库，快速上手说明见 [BlueBuild 文档](https://blue-build.org/how-to/setup/)。

完成搭建后，建议你更新本 README，说明这个自定义镜像的用途。

## 安装

> [!WARNING]  
> [这是实验性功能](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable)，请自行斟酌后使用。

要把一个已有的 Fedora 原子系统切换到最新构建：

- 先切换到未签名镜像，以便正确安装签名密钥与策略：

  ```
  rpm-ostree rebase ostree-unverified-registry:ghcr.io/dlitfx/bazzite-niri-nonvidia:latest
  ```

- 重启以完成切换：

  ```
  systemctl reboot
  ```

- 然后切换到已签名镜像，如下所示：

  ```
  rpm-ostree rebase ostree-image-signed:docker://ghcr.io/dlitfx/bazzite-niri-nonvidia:latest
  ```

- 再次重启以完成安装：

  ```
  systemctl reboot
  ```

`latest` 标签会自动指向最新构建。该构建仍始终使用 recipe.yml 中指定的 Fedora 版本，因此你不会被意外升级到下一个大版本。

## ISO

如果在 Fedora 原子系统上构建，你可以按[此处](https://blue-build.org/how-to/generate-iso/#_top)的说明生成离线 ISO。遗憾的是，由于体积过大，这些 ISO 无法免费分发在 GitHub 上，因此公开项目需要另找地方托管。

## 验证

这些镜像使用 [Sigstore](https://www.sigstore.dev/) 的 [cosign](https://github.com/sigstore/cosign) 签名。你可以从本仓库下载 cosign.pub 文件，运行以下命令来验证签名：

```bash
cosign verify --key cosign.pub ghcr.io/dlitfx/bazzite-niri-nonvidia
```

## 上传前recipe.yml文件语法校验

这个命令会检查 YAML 语法和模块的 schema 是否符合规范。

```bash
bluebuild validate ./recipes/recipe.yml
```

## 系统后续升级

### 配置方法：为 rpm-ostreed 服务单独设置代理

根据 Red Hat 和 Fedora 的文档，正确做法是通过 systemd drop-in 文件给 `rpm-ostreed.service` 注入代理环境变量。

**1. 创建配置目录和文件**

```bash
sudo mkdir -p /etc/systemd/system/rpm-ostreed.service.d/
```

**2. 写入代理配置**

使用 `sudo` 编辑或创建文件 `/etc/systemd/system/rpm-ostreed.service.d/00-proxy.conf`，内容如下（把其中的地址和端口换成你实际的代理）：

```ini
[Service]
Environment="http_proxy=http://你的代理地址:端口"
Environment="https_proxy=http://你的代理地址:端口"
```

如果代理需要用户名密码，格式是 `http://用户名:密码@代理地址:端口`。如果你的代理是 SOCKS5 类型，`rpm-ostree` 可能不支持，需要确认代理软件是否提供了 HTTP 代理端口。

**3. 重载并重启服务**

```bash
sudo systemctl daemon-reload
sudo systemctl restart rpm-ostreed.service
```

之后再次运行 `rpm-ostree upgrade`，下载流量就应该经过代理了。

系统镜像完成切换后

```bash
rpm-ostree upgrade
```

## appimage安装

项目目录下有`install-appimages.sh`维护里面的下载链接，执行脚本后，再使用GearLever统一安装。
