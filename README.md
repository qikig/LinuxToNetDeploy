# LinuxToNetDeploy
再linux 下部署 Net Core 服务
#换源命令 sudo nano /etc/apt/sources.list
# 替换为阿里云源，例如：
```text
 deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```
# nano 命令 Ctrl + O 保存
#          Ctrl + X 退出
# 推荐安装 微软的Edit 文本编辑工具好用，安装参考 https://bohu.net/blog/10245/
```text
# 安装 Zstandard 
apt install zstd

# 下载软件包
wget https://github.com/microsoft/edit/releases/download/v1.2.0/edit-1.2.0-x86_64-linux-gnu.tar.zst

# 解压缩到用户的当前目录
tar xvf edit-1.2.0-x86_64-linux-gnu.tar.zst

# 将其移动到 bin 目录中以便随时访问，请相应调整路径
mv edit /usr/local/bin/edit

# 查看版本号
edit -v

# 启动编辑器
edit
```

# 1. linux 安装net sdk 
  #https://learn.microsoft.com/zh-cn/dotnet/core/install/linux
# 2. ubnntu 20 需要创建 连接，大于20 版本不需要
```text
    1.
      wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
      sudo dpkg -i packages-microsoft-prod.deb
      rm packages-microsoft-prod.deb
    2.
      sudo apt-get update && \
      sudo apt-get install -y dotnet-sdk-8.0
```
## 3.linux 安装 nginx 命令：
```text
sudo apt install nginx
状态：systemctl status nginx 
重启：systemctl restart nginx
停止：systemctl stop nginx
加载配置文件：systemctl reload nginx
开机自启：sudo systemctl enable nginx
查看配置正确：sudo nginx -t
```
## nginx 配置基本配置样例
```json
server {
listen 80; #设置监听的端口
server_name localhost 127.0.0.1; #设置绑定的主机名、域名或ip地址
    location / {
       root  /var/www/html/; #设置服务器默认网站的根目录位置,需要手动创建
        index index.html index.htm; #设置默认打开的文档
    } 
   location /swgger/ {
        proxy_pass  http://127.0.0.1:5001/swagger/;  
    }
}
```
## 4.配置net启动
# net启动服务文件 可以用sudo vi /etc/systemd/system/myfirstwebapp.service  VI命令编辑添加
# vi 常用命令 i 光标前，a 光标后插入
    # Esc键退出插入模式 输入:（冒号），进入命令行模式。输入wq

## 下面保存xxx.service 修改服务文件通过 systemctl daemon-reload 加载
参考地址
https://learn.microsoft.com/zh-cn/troubleshoot/developer/webapps/aspnetcore/practice-troubleshoot-linux/2-3-configure-aspnet-core-application-start-automatically
---------------
```text
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/helloapp
ExecStart=/usr/bin/dotnet /var/www/helloapp/helloapp.dll
Restart=always
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
#配置启动环境 Development
Environment=ASPNETCORE_ENVIRONMENT=Production
#禁用 .NET 遥测日志，减少噪音
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
#配置启动端口
Environment=ASPNETCORE_URLS=http://localhost:5009
#输出日志 最后禁用 可以通过 journalctl -u 服务名 --since "5 minutes ago" 来查看
#StandardOutput=append:/var/log/webapp/webapp2.log
#StandardError=inherit
[Install]
WantedBy=multi-user.target
```
---------------
# 服务重启命令
```text
 sudo systemctl start myfirstwebapp.service 

 sudo systemctl stop myfirstwebapp.service 

 sudo systemctl restart myfirstwebapp.service 

 sudo systemctl status myfirstwebapp.service

# 加入开机自启 sudo systemctl enable myfirstwebapp.service
```

# 运维相关命令 
```text
 查看服务日志 journalctl -u myfirstwebapp
 修改文件权限  chmod 777 /etc/xxx/
 查看 某个进程 ps aux | grep xxx
 访问网址 curl http：//127.0.0.1/
 查看文件 cat xxx.TXT 或者 less xx.txt
 查看文件大小 du -h 文件名
 复制文件 cp /path/to/source/file.txt /home/username/documents/
 移动文件 mv [源文件路径] [目标目录]

将文件复制到 Linux
pscp -i <private key path> <local file to upload> user@host:<Linux path to save>
若要将 c：\web\publish.zip 文件复制到 Linux 中的用户主目录
pscp -i d:\secure\myprivatekey.ppk c:\web\publish.zip <UserName>@buggyamb:<Linux path to save>
```
