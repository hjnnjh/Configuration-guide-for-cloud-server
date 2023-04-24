

# Configuration-guide-for-cloud-server

**本指南用于记录一个云服务器实例从购买到完成配置本人需要的一些服务的全过程，以供今后购买新的云服务器实例需要进行配置时参考**。

## 配置SSH密钥登录

- 目前使用的云服务器为腾讯云的轻量应用服务器，首先在腾讯云的在控制台中创建SSH密钥

![image-20230422220238066](https://gitee.com/zephyrushjnnjh/image-repo/raw/master/img/202304222202159.png)

- 将实例关闭之后，将密钥绑定至实例

![image-20230422220423576](https://gitee.com/zephyrushjnnjh/image-repo/raw/master/img/202304222204616.png)

- 配置本机`~/.ssh/config`文件

- 登录服务器，修改ssh相关配置（如关闭密码登录：将`PasswordAuthentication`修改为`no`）

```shell
sudo vim /etc/ssh/sshd_config
```

## 安装zsh, oh-my-zsh, powerlevel10k

###  安装zsh

下载`zsh`

```shell
sudo apt install zsh
```

切换到root用户下修改ubuntu用户的密码

```shell
passwd ubuntu
```

修改默认终端为`zsh`（不用加`sudo`，因为需要修改的是当前用户的默认终端，需要输入当前用户的密码）

```shell
chsh -s /bin/zsh
```

### 安装并配置[oh-my-zsh](https://ohmyz.sh/)

安装`git`

```shell
sudo apt update
sudo apt install git
```

安装`clash`（dddd）

- 下载对应的安装包：https://github.com/Dreamacro/clash/releases

![image-20230422230303783](https://gitee.com/zephyrushjnnjh/image-repo/raw/master/img/202304222303822.png)

- 解压 `clash`

- 将解压出的文件重命名成 `clash`

- 将 `clash` 移动到 `/usr/bin/` 目录下

- 赋予 `clash` 可执行权限

  ```shell
  sudo chmod +x /usr/bin/clash
  ```

- 默认配置目录是 `$HOME/.config/clash`, 默认配置目录启动

  ```shell
  clash
  ```

  有几率遇到`Country.mmdb`下载不了的情况，需手动上传`Country.mmdb`文件

- 配置`$HOME/.config/clash/config.yaml`文件

- 配置`systemd`服务

```shell
[Unit]
Description=clash service
After=network.target

[Service]
Type=simple
User=ubuntu
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
ExecStart=/usr/bin/clash
Restart=always

[Install]
WantedBy=multi-user.target
```

```shell
sudo systemctl daemon-reload
sudo systemctl start clash
sudo systemctl enable clash
```

- 在`~/.zshrc`中添加以下内容，为`zsh`配置代理

```shell
# proxy settings
alias setproxy="export http_proxy=http://127.0.0.1:7890; export https_proxy=http://127.0.0.1:7890; echo 'Set Proxy Successfully!'"
alias unsetproxy="unset http_proxy; unset https_proxy; echo 'Unset Proxy
Successfully!'"
```

- 打开终端代理

```shell
setproxy
```

安装`oh-my-zsh`

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

- 配置`oh-my-zsh`的插件
- 安装`zsh-autosuggestions`

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

- 安装`zsh-syntax-highlighting`

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

- 修改`~/.zshrc`中的`plugins`选项

```apl
plugins=(git zsh-autosuggestions zsh-syntax-highlighting z sudo)
```

```shell
source ~/.zshrc
```

### 安装[powerlevel10k](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k)

- 首先安装以下字体（在`p10k`GitHub主页可下载）

  ![image-20230423001846694](https://gitee.com/zephyrushjnnjh/image-repo/raw/master/img/202304230018746.png)

```shell
sudo mv MesloLGS* /usr/share/fonts/
sudo apt install fontconfig
sudo fc-cache -f -v
```

- 安装`powerlevel10k`


```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

- Set `ZSH_THEME="powerlevel10k/powerlevel10k"` in `~/.zshrc`.

```shell
source ~/.zshrc
```

- 按照提示完成配置，效果如下：

![WechatIMG253](https://gitee.com/zephyrushjnnjh/image-repo/raw/master/img/202304231252714.png)

## 配置[frp](https://github.com/fatedier/frp)内网穿透工具

###  公网服务器配置服务端frps

- 配置`frps.ini`文件

```shell
[common]
bind_port = 7000  # 用于frp服务端与客户端通信的端口
vhost_http_port = 8889  # web服务端口，我主要用来远程连接jupyter notebook
```

- 配置`systemd`服务`frps.service`

```shell
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=ubuntu
Restart=on-failure
RestartSec=5s
ExecStart=/home/ubuntu/frp/frps -c /home/ubuntu/frp/frps.ini  # frps执行文件和配置文件
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

### 内网服务器配置客户端frpc

- 配置`frpc.ini`文件

```shell
[common]
server_addr = xxx.xxx.xxx.xxx  # 公网服务器ip
server_port = 7000

[jupyter]
type = http
local_port = 8889  #要映射的jupyter端口, 需提前在服务器控制台防火墙放行
custom_domains = xxx.xxx.xxx.xxx  # 公网服务器ip
```

- 配置`systemd`服务`frpc.service`

```shell
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/home/ubuntu/frp/frpc -c /home/ubuntu/frp/frpc.ini  # frpc执行文件和配置文件
ExecReload=/home/ubuntu/frp/frpc reload -c /home/ubuntu/frp/frpc.ini  # frpc执行文件和配置文件
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

### 启动frp服务

先启动`frps`，再启动`frpc`

```shell
sudo systemctl daemon-reload
sudo systemctl start frps
sudo systemctl enable frps
```

```shell
sudo systemctl daemon-reload
sudo systemctl start frpc
sudo systemctl enable frpc
```

修改`~/.ssh/config`中内网服务器登录端口号为`bind_port`，整个配置过程完成。

## 配置[Aria2-Pro](https://p3terx.com/archives/docker-aria2-pro.html)

TODO，后面有空折腾再继续更新。
