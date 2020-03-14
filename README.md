# 准备工作
1.你需要拥有一个自己的**域名**，并会正确设置**DNS解析**，如果你不知道以上两个名词的含义，请自行Google学习    
2.如果你在使用**Cloudflare**等包含**CDN服务**的域名解析提供商，请**确保CDN服务关闭**，在Cloudflare上为**让云朵变灰**     
3.请使用**Debian9**系统搭建，其它系统未经测试

# 开始部署
- 下载证书申请脚本
```bash
apt-get update && apt-get -y install socat         
wget -qO- get.acme.sh | bash       
source ~/.bashrc
```
- 安装脚本 （注意：**yourdomain.com**请替换为你自己的域名）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/trojan.crt --key-file /etc/trojan/trojan.key --ecc
```
- 安装Nginx并编辑配置文件
```bash
apt update && apt install nginx
vim /etc/nginx/conf.d/v2ray.conf
```
- 将以下内容粘贴 （注意：**yourdomain.com**请替换为你自己的域名，**proxy.com** 请替换为你想镜像的网站。     
**切勿替换为墙内不可直连的网站，例如Google/Facebook/Youtube等等。最好也不要替换为服务器在国内的网站，例如 百度/豆瓣 等等**
```bash
server {
    listen 80 ;
    server_name  yourdomain.com;  
    location / {
        proxy_pass proxy.com;
        proxy_redirect     off;
        proxy_connect_timeout      75; 
        proxy_send_timeout         90; 
        proxy_read_timeout         90; 
        proxy_buffer_size          4k; 
        proxy_buffers              4 32k; 
        proxy_busy_buffers_size    64k; 
        proxy_temp_file_write_size 64k; 
     }
}
```
- 安装Trojan并编辑配置文件
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/trojan
cd /etc/trojan && vim config.json
```
- 将以下内容粘贴 （注意：**password0**请更改为**你自己设置的密码**，最好在8位以上，其余部分无需更改，除非你知道自己在做什么）
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/trojan/trojan.crt",
        "key": "/etc/trojan/trojan.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```
- 启动BBR加速
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
- 启动服务
```bash
nginx -s reload
docker run -d --name trojan --restart always --net host -v /etc/trojan:/etc/trojan teddysun/trojan
```

# 客户端
*因本人不使用苹果系产品，因此苹果用户请自行寻找其他教程*            

点击[这里](https://github.com/trojan-gfw/igniter/releases)下载**安卓**客户端         
配置如下： **地址**填你的域名，**端口**填 443 ，**密码**填你刚才设置的密码，其他选项无需更改        

点击[这里](https://github.com/charlieethan/Trojan/releases/download/V1.0/Trojan.zip)下载**Windows 64位**客户端

# 注意事项
1.目前这个项目还处在开发阶段，windows客户端只能*代理端口*，浏览器上网需配合插件使用      
请点击[链接](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)下载Chrome浏览器插件      
2.为什么推荐用我的教程搭建而非一键脚本？代码和步骤公开透明，没有后门，保护你的隐私。毕竟你不知道写脚本的那个人，背后是谁
