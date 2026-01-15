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
# nano 命令用不来？ 推荐安装 微软的Edit 文本编辑工具好用，安装参考 https://bohu.net/blog/10245/
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
        # 证书
        #ssl_certificate           /etc/ssl/certs/testCert.crt;
        #ssl_certificate_key       /etc/ssl/certs/testCert.key;
        #ssl_session_timeout       1d;
        #ssl_protocols             TLSv1.2 TLSv1.3;
        #ssl_prefer_server_ciphers off;
        #ssl_ciphers               ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
        #ssl_session_cache         shared:SSL:10m;
        #ssl_session_tickets       off;
        #ssl_stapling              off;

        add_header X-Frame-Options DENY;
        add_header X-Frame-Options "SAMEORIGIN";#免受点击劫持
        add_header X-Content-Type-Options nosniff;#阻止 MIME 方式探查

    location / {
       root  /var/www/html/; #设置服务器默认网站的根目录位置,需要手动创建
        index index.html index.htm; #设置默认打开的文档
    } 
   location /swgger/ {
        proxy_pass  http://127.0.0.1:5001/swagger/;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection $connection_upgrade;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
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
 给程序目录www用户权限 sudo chown -R www-data:www-data /path/to/your/app
 chmod -R 755 /path/to/your/app
 查看 某个进程 ps aux | grep xxx
 访问网址 curl http：//127.0.0.1/
 查看文件 cat xxx.TXT 或者 less xx.txt
 查看文件大小 du -h 文件名
 复制文件 cp /path/to/source/file.txt /home/username/documents/
 移动文件 mv [源文件路径] [目标目录]
 清理journal日志 只保留近一周的日志 journalctl --vacuum-time=1w
 查找大文件 find /path/to/search -type f -size +100M

将文件复制到 Linux
pscp -i <private key path> <local file to upload> user@host:<Linux path to save>
若要将 c：\web\publish.zip 文件复制到 Linux 中的用户主目录
pscp -i d:\secure\myprivatekey.ppk c:\web\publish.zip <UserName>@buggyamb:<Linux path to save>
```
docker-run-rancher
国内安装
https://forums.rancher.cn/t/docker-run-rancher-rancher-mirrored-pause/3546/2
```text
root@ksd:~# mkdir -p /data/rancher/k3s/agent/images/
root@ksd:~# docker run --rm --entrypoint "" -v $(pwd):/output rancher/rancher:v2.9.0 cp /var/lib/rancher/k3s/agent/images/k3s-airgap-images.tar /output/k3s-airgap-images.tar
root@ksd:~# ls
docker.sh  k3s-airgap-images.tar  snap
root@ksd:~# cp k3s-airgap-images.tar /data/rancher/k3s/agent/images/

docker run -d --restart=unless-stopped --privileged \
    -p 10080:80 -p 10443:443  \
    -e CATTLE_SYSTEM_DEFAULT_REGISTRY=registry.cn-hangzhou.aliyuncs.com \
    -e CATTLE_BOOTSTRAP_PASSWORD=rancher \
    -v /data/rancher:/var/lib/rancher \
    registry.cn-hangzhou.aliyuncs.com/rancher/rancher:v2.9.0
8b62637c72cfb4b3704d5811f8e9bbd37bd05a93c9155465e776b5e0481f5465
```
注册服务
 .\npc.exe install
 net start npc

.NET 6 以上的 windows 服务错误 

配置文件路径不正确，create host builder 的时候加入一些参数
```text
var options = new WebApplicationOptions
            {
                Args = args,
                ContentRootPath = WindowsServiceHelpers.IsWindowsService()
        ? AppContext.BaseDirectory
        : default
            };
            var builder = WebApplication.CreateBuilder(args);
            builder.Host.UseWindowsService();
在sc命令中，=号前面不能有空格，而=号后面必须有一个空格，切记。另外要以管理员的身份打开命令行。
```
sc create Service binPath="C:\Service.exe" start= auto displayname="Service"

sc delete Service

sc stop 服务名

sc start 服务名

jenkins 发布到windows的问题
Source files
/var/lib/jenkins/workspace/testweb 下的路径 jenkins下wordkspace/项目目录下的 路径
xx/xx//bin/Release/net8.0/publish/**

Remove prefix
填写Source files 一样的，要不会再服务器端创建目录

SSH Servers
Remote Directory
要填写/  这对应的就c盘
/web
对应c:/web/

#linux 单文件框架依赖 运行 已经安装了skd和运行时 出现.NET location: Not found
 设置 .net 安装路径 export DOTNET_ROOT=/usr/share/dotnet    

