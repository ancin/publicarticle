# nginx 配置HTTPS， 转向http

nginx配置如下。

 location / {
         proxy_pass http://127.0.0.1:8080/;
             proxy_set_header Host $host:443;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_redirect http:// $scheme://;
        }

其他配置采用nginx给出的默认配置即可。

# Tomcat的配置如下。添加如下配置

<Valve className="org.apache.catalina.valves.RemoteIpValve" remoteIpHeader="X-Forwarded-For"
           protocolHeader="X-Forwarded-Proto" protocolHeaderHttpsValue="https"/>

