# 相关依赖
#!/bin/sh
yum install -y epel-release && \
yum update -y && \
yum install -y deltarpm && \
yum install -y openssh-server sudo which file curl zip unzip wget && \
yum install -y libmicrohttpd-devel jansson-devel libnice-devel glib22-devel opus-devel libogg-devel pkgconfig  gengetopt libtool autoconf automake make gcc gcc-c++ git cmake libconfig-devel openssl-devel

#upgrade libsrtp 1.5.4
#wget https://github.com/cisco/libsrtp/archive/v1.5.4.tar.gz
#tar xfv v1.5.4.tar.gz
#cd libsrtp-1.5.4
#./configure --prefix=/usr --enable-openssl
#make shared_library && sudo make install

#cd -

wget https://github.com/cisco/libsrtp/archive/v2.0.0.tar.gz
tar xfv v2.0.0.tar.gz
cd libsrtp-2.0.0
./configure --prefix=/usr --enable-openssl
make shared_library && sudo make install

cd -

#install sofia-sip for sip-gateway plugin
wget https://sourceforge.net/projects/sofia-sip/files/sofia-sip/1.12.11/sofia-sip-1.12.11.tar.gz
tar zxf sofia-sip-1.12.11.tar.gz && cd sofia-sip-1.12.11 && ./configure --prefix=/usr CFLAGS=-fno-aggressive-loop-optimizations && make && make install

cd -

#install usrsctp for Data channel support
git clone https://github.com/sctplab/usrsctp && cd usrsctp && \
./bootstrap && \
./configure --prefix=/usr && make && make install

cd -

#install libwebsocket for android or ios instead of http/https
git clone https://github.com/warmcat/libwebsockets && \
mkdir libwebsockets/build && cd libwebsockets/build && \
cmake -DMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_C_FLAGS="-fpic" .. && \
make && make install

cd -
# Janus安装
#Janus
#if cannot configure plugin sofia,Perhaps you should add the directory containing `sofia-sip-ua.pc' to the PKG_CONFIG_PATH environment variable,
#for example centos7 :export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
#if cannot load libsofia-sip-ua.so.0 , try ldconfig -v
git clone https://github.com/meetecho/janus-gateway.git && \
cd janus-gateway &&\
sh autogen.sh && \
./configure --prefix=/opt/janus --disable-rabbitmq --disable-docs &&\
make && make install && make configs

上述脚本整理了sip（sofia）和 datachannel插件安装，消息通道额外支持了websocket
Extension
1.若要使用libsrtp1.5.4，需要在configure janus 时加上 --disable-libsrtp2
2.如果需要支持mqtt可以安装paho.mqtt.c后重新configure janus编译安装

git clone https://github.com/eclipse/paho.mqtt.c.git
cd paho.mqtt.c
make && sudo make install

其他插件如果使用，可以参考janus官方文档。
# 运行
nohup /opt/janus/bin/janus >> /var/log/janus.log 2>&1 &
查看Janus是否运行正常。
netstat -anp | grep janus
# 配置 
bin  Janus 执行文件；etc janus配置文件；include 依赖。
janus.cfg 如下
.静态配置的turn server
[nat]
turn_server = 127.0.0.1
turn_port = 3478
turn_type = udp
turn_user = username
turn_pwd = password
2.配置TURN REST API动态获取turn 服务
turn_rest_api = http://yourbackend.com/path/to/api
turn_rest_api_key = anyapikeyyoumayhaveset
turn_rest_api_method = GET
  janus.transport.http.cfg
  [general]
json = indented                         ; Whether the JSON messages should be indented (default),
                                                        ; plain (no indentation) or compact (no indentation and no spaces)
base_path = /janus                      ; Base path to bind to in the web server (plain HTTP only)
threads = unlimited                     ; unlimited=thread per connection, number=thread pool
http = yes                                      ; Whether to enable the plain HTTP interface
port = 18000                                    ; Web server HTTP port
;interface = eth0                       ; Whether we should bind this server to a specific interface only
;ip = 192.168.0.1                       ; Whether we should bind this server to a specific IP address (v4 or v6) only
https = yes                                     ; Whether to enable HTTPS (default=no)
secure_port = 18001    
在这里可以配置监听端口线程数等，支持http和https
janus.plugin.sip.cfg
以sip(sofia)插件为例子
[general]
; Specify which local IP address to use. If not set it will be automatically
; guessed from the system
;local_ip = 1.2.3.4

; Enable local keep-alives to keep the registration open. Keep-alives are
; sent in the form of OPTIONS requests, at the given interval inseconds.
; (0 to disable)
keepalive_interval = 120

; Indicate if the server is behind NAT. If so, the server will use STUN
; to guess its own public IP address and use it in the Contact header of
; outgoing requests
behind_nat = no

; User-Agent string to be used
; user_agent = Cool WebRTC Gateway

; Expiration time for registrations
register_ttl = 3600

; Range of ports to use for RTP/RTCP (default=10000-60000)
rtp_port_range = 40000-60000

# 运行demo
janus的demo目录在/opt/janus/share/janus/demos
配置到NGINX中。
点击start出现上述问题是因为自签证书问题导致，可访问https://yourip_or_domain:port会提示证书问题点击继续访问信任，再返回start siptest即可，port为janus的 https监听端口
正常开始后，如下配置点击register 即可开始sip呼叫

Extension
默认demo里面没有配置iceserver，如果服务端报错ice fail可以在客户端代码加上iceserver

vim siptest.js
janus = new Janus(
				{
					server: server,
					iceServers: [{urls: "turn:turnip:port", username: "username", credential: "password"}],
					success: function() {
          
  demo中默认没有启用录音功能，但是每个插件都有提供录音接口，可在源码内查看对应REST API说明，在siptest中录音接口说明如下
  {
	"request" : "recording",
	"action" : "<start|stop, depending on whether you want to start or stop recording something>"
	"audio" : <true|false; whether or not our audio should be recorded>,
	"video" : <true|false; whether or not our video should be recorded>,
	"peer_audio" : <true|false; whether or not our peer's audio should be recorded>,
	"peer_video" : <true|false; whether or not our peer's video should be recorded>,
	"filename" : "<base path/filename to use for all the recordings>"
}

开始录音
sipcall.send({"message": { "request": "recording", "action":"start","audio":true,"peer_audio":true,"filename":"/opt/janus/share/janus/recordings/" + $('#authuser').val() + "to" + $('#peer').val() + "_" + getNowFormatDate()}});

停止录音
sipcall.send({"message": { "request": "recording", "action":"stop"}});
其他插件如videocall videoroom类似可以录音

janus.js
demo中使用了janus.js。janus.js是和janus服务器进行通信的javascript库，通过使用janus.js简化了webrtc api的使用，以及前端与janus服务器建立连接，交换sdp等功能。不使用janus.js可以自己实现类似功能与janus服务器通信

# Turn部署
git clone https://github.com/coturn/coturn.git
cd coturn
./configure
make && make install

配置
配置参考corturn 配置
可以配置静态账户或者REST API使用
验证turn可以在下面网址进行https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/
若收集的candidates中有添加的turn server ip则可以正常使用

三、配置
进入coturn/bin目录

1、使用turnadmin添加用户及计算密码

turnadmin的使用方法，可以通过man查看。如果查看turnadmin属性，则会发现turnadmin是一个链接，指向turnserver。

1）添加lt-cred-mech用户：
./turnadmin -a -u demo -p 4080218913 -r demo

其中的参数含义
-a: 表示使用lt-cred-mech方式连接
-u：用户名
-p：密码
-r ：realm 域。自己试了，似乎随便写，没啥影响。

2）添加admins用户，应该是使用https时使用吧
./turnadmin -A -u demo -p 4080218913 -r demo

其中的参数含义同上，
-A：表示admin用户。

3）根据用户名、密码、realm计算出值
./turnadmin -k -u demo -p 4080218913 -r demo

此时会出来一串数字。
XXXXXXX

配置turnserver.conf文件

可以将example中的turnserver.conf文件拷贝到bin目录下。

将turnserver.conf文件按照如下配置编辑。

#如果多网卡，记得此处设置为和你所用监听的IP相对应的eth
listening-device=eth0 
listening-port=3478
relay-device=eth0
min-port=59000
max-port=65000
Verbose
fingerprint
#webrtc需要使用此选项
lt-cred-mech
use-auth-secret
static-auth-secret=4080218913
#之前turnadmin中-r参数的值，此处要对应
realm=demo
stale-nonce
#可以添加用户名和密码
user=demo:4080218913
#测试期间可以使用example/etc中的pem，自己计算的话需要用到openssl，方法为：
#sudo openssl req -x509 -newkey rsa:2048 -keyout /etc/turn_server_pkey.pem -out /etc/turn_server_cert.pem -days 99999 -nodes

#填写pem目录即可，如
cert=~/coturn/example/etc/turn_server_cert.pem
pkey==~/coturn/example/etc/turn_server_pkey.pem
no-loopback-peers
no-multicast-peers
mobility
no-cli
#各项参数含义，可以看turnserver.conf中的说明。
sudo turnserver 启动即可
turnserver会载入当前目录中的配置文件。


其他Webrtc Server
除了janus还可以选用medooze，最近在webrtchacks上有文章对janus mediasoup medooze等webrtc server 进行SFU负载测试（文章链接），medooze性能表现比较优秀，代码也比较易读，听webrtc中文群里大牛说medooze近期可能会有大的更新貌似，会有全端客户端开源，到时再学习 - -


