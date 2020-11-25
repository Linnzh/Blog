# 开始使用百度云服务器

## 登陆

```sh
ssh root@106.12.96.157

# 创建新用户 linnzh
useradd linnzh

```

### 添加普通用户 sudo 权限

```sh
sudo visudo
# 向该文件添加一条
# linnzh ALL=(ALL:ALL) NOPASSWD:ALL
# 使该用户具有 root 的所有权限，并可不输入密码
# 可通过 sudo vim /etc/passwd
# 查看刚才的用户是否在该文件中
```

### 使用密钥登陆

```sh
# 生成本机公钥和密钥
ssh-keygen

cd .ssh

# 创建 授权文件
touch authorized_keys

# 将授权访问机器的公钥复制进该文件

# 然后就可以在授权访问机器上无密码登陆
```

### 使用别名(免输用户名和IP)登陆

```sh
# 在授权访问机器上操作
# 进入 ~/.ssh 目录并创建 config 文件
vim config

# 并键入以下内容
Host test
    HostName 106.12.96.157
    User linnzh
    Port 22
    IdentityFile ~/.ssh/id_rsa

# 保存后即可使用 ssh test 的形式登陆服务器
```

## 安装 zsh

```sh
sudo apt-get install zsh -y

# 查看当前可用 shell
cat /etc/shells

# 切换默认终端
chsh -s /bin/zsh

# 查看当前使用 shell
echo $SHELL

# 退出终端，重新登陆即刻生效

# 安装 Git Vim
sudo apt-get install git vim -y

# 复制该链接内容<https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh>
# 至 install.sh 文件
# 并运行该 oh-my-zsh 安装文件
sh ./install.sh
```
