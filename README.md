# Debian 11手动安装Xray

## 1. 准备工作
### 1.1 设置root密码（适用于谷歌云，腾讯云等不提供root密码的VPS）
### 切换root
```
sudo -i
```
### 设置root密码
```
passwd     # 需输入2次自定义密码
```
```
sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
reboot
```
## 2. 安装依赖并执行更新
### 安装依赖
```
sudo apt-get install openssl cron socat curl wget unzip
```
### 执行更新
```
sudo apt update && sudo apt upgrade -y
```
### 可选-自动删除旧包和依赖项
```
sudo apt autoremove
```
## 3. 安装Google BBR
### 3.1 为了启用BBR算法，需要修改sysctl配置：
```
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
### 3.2 使用以下命令确认 BBR 已启用：
```
sudo sysctl net.ipv4.tcp_available_congestion_control
```
### 3.3 验证：
```
sudo sysctl -n net.ipv4.tcp_congestion_control
```
### 3.4 检查内核模块是否已加载：
```
lsmod | grep bbr
```
## 4. 安装acme.sh
```
systemctl stop nginx
curl  https://get.acme.sh | sh -s email=my@example.com     
# 替换 my@example.com 为自己的邮箱地址
~/.acme.sh/     # 把 acme.sh 安装到 home 目录下
alias acme.sh=~/.acme.sh/acme.sh     # 创建一个 bash 的 alias, 方便使用    
acme.sh --issue -d mydomain.com --standalone --keylength ec-256     # 替换 mydomain.com 为自己的域名地址
acme.sh --install-cert -d mydomain.me --fullchain-file /etc/ssl/private/xray.crt --key-file /etc/ssl/private/xray.key --ecc     # 替换 mydomain.com 为自己的域名地址,cret和key要有755的权限
acme.sh --upgrade --auto-upgrade
```

## 5. 安装并启动Nginx
```
sudo apt-get install nginx
mkdir -p /var/www/website/html      # 创建一个网站专用的文件夹
```


## 7. 安装Xray
### 7.1 安装Xray
```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
xray uuid     # UUID
```
### 7.2 VLESS over TCP with XTLS + 回落 & 分流 to WHATEVER（终极配置）（路径：usr/local/etc/config.jason）
