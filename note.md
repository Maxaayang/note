****# 端口映射
```shell
ssh -L8889:localhost:8889 ten
```

![jupyter notebook不用映射端口](https://huixxi.github.io/2017/06/09/Linux服务器上搭建TensorFlow机器学习环境/#more)

# 更改权限
```shell
sudo chown -R addr:addr ./*
```

# jupyter notebook切换kernal
```python
conda install ipykernel
python -m ipykernel install --user --name 本地环境名称 --display-name "在jupyter上显示的环境名称"

python -m ipykernel install --user --name py27 --display-name "py27"

# 查看kernal
jupyter kernelspec list

# 删除指定的kernal
jupyter kernelspec remove kernel_name∫∫∫∫
```

# clash
- https://v2rayse.com/client 下载clash-linux-amd64-v1.11.4.gz
- https://github.com/Dreamacro/clash/releases/download/v1.11.8/clash-linux-amd64-v1.11.8.gz
- gunzip clash.gz
- chmod +x clash
- mkdir /opt/clash  mv clash /opt/clash/
- config.yaml
- vim /etc/systemd/system/clash.service

```shell
[Unit]
Description=clash-core
[Service]
Type=simple
ExecStart=/opt/clash/clash -f /opt/clash/config.yaml
```

- sudo systemctl daemon-reload
- sudo systemctl start clash
- sudo systemctl status clash
- 现在桌面可以了，终端还不可以
- vim ~/.bashrc
- alias proxy="export http_proxy=http://127.0.0.1:7890; export https_proxy=http://127.0.0.1:7890"
- alias unproxy="unset http_proxy;unset https_proxy"
- ui Dreamacro/clash-dashboard gh-pages分支 https://github.com/Dreamacro/clash-dashboard/tree/gh-pages
- config.yaml  external-ui: /opt/clash/ui   IP:9090/ui
- 使用docker
- apt install docker.io
- docker pull dreamacro/clash
- docker run -d --name clash -p 7890:7890 -p 7891:7891 -p 9090:9090 -v /opt/clash/config.yaml:/root/.config/clash/config.yaml -v /opt/clash/ui:/opt/clash/ui dreamacro/clash
- 查看日志 docker logs -f 3c436de541b7edf45167df7b7d6b844fd42cabdac926e4298d6420693fd87b73

# 查看端口号
netstat -ap | grep 8080

# zsh plugin not find
原因: 没有把插件的代码仓库克隆到本地位置上
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

设置为默认shell(需重启shell)
chsh -s /bin/zsh
查看目前的shell
echo $SHELL

查看当前目录下所有文件夹的大小：du -sh *

# 切换默认终端

```shell
# 查看有哪些终端
cat /etc/shells

# 切换终端
chsh -s /bin/bash
```

github token: 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

# win10小鹤双拼
https://ifttl.com/add-flypy-to-win10-microsoft-pinyin-and-other-configuration/

# DNN library is not found

import os
os.environ['CUDA_VISIBLE_DEVICES'] = '/[gpu](https://so.csdn.net/so/search?q=gpu&spm=1001.2101.3001.7020):0'

# Ubuntu换源

vim /etc/apt/sources.list

```shell
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse


deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security multiverse




```

sudo apt update

# conda换源
https://blog.csdn.net/scl52tg/article/details/120959893

python 各种类型直接互转
https://blog.csdn.net/qq_38703529/article/details/120216078

# ssh
sudo yum install openssh-server
sudo service sshd start
sudo yum install initscripts -y

ssh-keygen -t rsa

# sudoers
[vim](https://so.csdn.net/so/search?q=vim&spm=1001.2101.3001.7020) /etc/sudoers
root ALL=(ALL) ALL

# 普通用户docker
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo service docker restart
sudo chmod a+rw /var/run/docker.sock


linux 桌面
sudo service gdm stop

# mac Ctrl + Command 拖动窗口
defaults write -g NSWindowShouldDragOnGesture -bool true