
# [mac 安装nginx 并用Nginx和Host把自定义域名指向本地实现Https访问](https://www.jianshu.com/p/fc1e81efc867)
## 一、安装
```javascript
brew install nginx
```
## 二、配置
1、找到Nginx文件夹
```javascript
cd /usr/local/etc/nginx
```
2、openssl生成自签名证书
创建服务器私钥，命令会让你输入一个口令
```javascript
openssl genrsa -out server.key 1024
```
根据私钥生成证书申请,创建签名请求的证书（CSR）
```javascript
openssl req -new -key server.key -out server.csr
```
下面的选项至少写一个，才可以生成证书成功
```javascript
Country Name (2 letter code) []:ch
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []:
Email Address []:
```
在加载SSL支持的Nginx并使用上述私钥时除去必须的口令：
```javascript
$ cp server.key server.key.org
$ openssl rsa -in server.key.org -out server.key
```
最后标记证书使用上述私钥和CSR
```javascript
openssl x509 -req -in server.csr -out server.crt -signkey server.key -days 3650
```
4、配置nginx：修改/usr/local/etc/nginx/nginx.conf 文件
```javascript
location / {
root  html（当前静态文件的路径）;
index  index.html index.htm;
}
```
> 不过最好不要把配置写到/usr/local/etc/nginx/nginx.conf里面,而是写在当前目录下面的servers的文件夹下，创建不同的config更加清晰：
例如：servers下建立一个millet.conf
在millet.conf里面配置
```javascript
server {
        listen       443 ssl;
        server_name  static.millet.com;
         #server.crt和server.key都在nginx下面
        ssl_certificate      server.crt;
        ssl_certificate_key  server.key;
        location / {
            root   （当前静态文件的路径）;
            index  index.html index.htm；
        }
    }
```
ihost
```javascript
127.0.0.1 static.millet.com
```
#### 启动nginx 
```javascript
sudo nginx
或者
sudo brew services start nginx
```
#### 停止nginx
```javascript
sudo nginx -s stop
或者
sudo brew services stop nginx
```
#### 重新加载配置文件
```javascript
sudo nginx -s reload
或者
sudo brew services restart nginx
```
#### 安装常见问题：
安装Nginx输入命令brew install nginx报错
You have not agreed to the Xcode license. Please resolve this by running:
sudo xcodebuild -license

nginx设置本地跨域

在mac 终端运行命令的时候会被提示没有同意xcode 证书 ，这个时候需要在Terminal中同意license
此时 输入命令 sudo xcodebuild -license 空格到最后 输入agree 就可以了
 
 

# [mac openssl生成你需要的域名的证书并且设置信任证书](https://www.jianshu.com/p/71095e5ca9b3)
 
### 创建密钥
首先，进入 nginx 配置目录，创建 openssl 配置文件 req.conf，其中的 CN, DNS.1, DNS.2 等需要替换为自己的域名：
```javascript
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = US
ST = VA
L = SomeCity
O = MyCompany
OU = MyDivision
CN = www.company.com
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = www.company.net
DNS.2 = company.com
DNS.3 = company.net
```
接着，执行如下命令，创建证书：
```javascript
openssl req -x509 -nodes -days 730 -newkey rsa:2048 -keyout cert.pem -out cert.pem -config req.conf -extensions 'v3_req'
```
### 配置nginx
```javascript
server {
    listen 443 ssl;
    server_name www.example.com;
    ssl_certificate cert.pem;
    ssl_certificate_key cert.pem;
    location / {
        root /Users/example/hello/world;
        index index.html index.htm;
    }
```
服务器证书（ssl_certificate）是一个公开文件，每个请求连接的客户端都会收到一份。私有密钥（ssl_certificate_key）是加密单元，需要存储在保密的地方，但要确保 nginx 主线程可访问。私有密钥一般和证书存储到同一位置。
cert.pem 就是上一个步骤产生的证书和密钥，在一个文件中。
### 配置浏览器
打开 Chrome 的开发者工具下的【security】选项卡，查看当前的证书，然后下载下来，双击添加到操作系统中，修改为始终信任就可以了。
如果还继续显示是不安全，像下图这样
![image.png](https://upload-images.jianshu.io/upload_images/2790249-bf55b94311309b02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以参照以下解决办法
Open developer tools
Goto the Application tab
Clear storage
Close and re-open tab
详细问题参照 [“Active content with certificate errors”](https://stackoverflow.com/questions/44145936/chrome-active-content-with-certificate-errors)
 
# [移动端Charles抓取https包](https://www.jianshu.com/p/d30339e0f5ae)
 
**一、安装charles抓包工具**
**二、配置Charles，允许抓取https包**
Proxy->SSL Proxying Settings…，勾选Enable SSL Proxying，Add一个locations，通过通配符* 抓取所有域名的https。（如果想只抓取某个域名的，设置具体域名的即可）
![image.png](https://upload-images.jianshu.io/upload_images/2790249-1d043b6fa5666473.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/2790249-bfdf31a2f707ff5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**三、PC端Charles安装https证书**
Help->SSL Proxying ->Install Charles Root Certificate，然后在钥匙串中信任证书即可
如果想要抓取pc端的接口请求勾选上macOS proxy就可以了
![image.png](https://upload-images.jianshu.io/upload_images/2790249-e7168a064945388d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**四、手机端手机端配置PC的代理并下载Charles的证书**
首先需要手机连接到与当前电脑同一个wifi局域网，对该wifi网络进行高级设置-代理：
代理服务器主机名：使用PC的本机IP地址
代理服务器端口：使用Charles设置的Port值，默认是8888，可以在下图proxy Settings查看端口号
![image.png](https://upload-images.jianshu.io/upload_images/2790249-77356ca93f4d9056.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/2790249-0cf97ba85f8f16b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后确定保存，第一次配置完代理，PC端会弹窗询问是否允许代理，点击allow
![image.png](https://upload-images.jianshu.io/upload_images/2790249-47ff85d660781b49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时我们的手机就已经挂上代理了。你也可以这么通过add添加
![image.png](https://upload-images.jianshu.io/upload_images/2790249-a370ec70bedc4070.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/2790249-f0d2bfa892652d8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
挂上代理了之后手机端需要在浏览器中访问chls.pro/ssl 下载证书
![image.png](https://upload-images.jianshu.io/upload_images/2790249-72e630fafbfe1717.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/2790249-38ae3f2317742b34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
ios10以上系统不会自动信任证书，需要在设置->通用->关于本机，信任安装的证书
现在客户端环境下，我们已经可以随时调试本地的H5代码了。
Charles结合nginx和ihost开发，在nginx代理的时候在开启charles用手机访问开发文件可能访问不到，可能是自制证书不信任的问题，需要对nginx和chrome做特殊处理

