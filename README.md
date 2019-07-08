## 复制inconshreveable/ngrok，新增权限验证。
#### 个人内部使用，所有必须设置authtoken.conf，不然无法连接。


运行build.sh  

#!/bin/sh
cd /ngrok/  

NGROK_DOMAIN="example.com"  

openssl genrsa -out rootCA.key 2048  
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=${NGROK_DOMAIN}" -days 5000 -out rootCA.pem  
openssl genrsa -out device.key 2048  
openssl req -new -key device.key -subj "/CN=${NGROK_DOMAIN}" -out device.csr  
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000  
cp rootCA.pem assets/client/tls/ngrokroot.crt  
cp device.crt assets/server/tls/snakeoil.crt  
cp device.key assets/server/tls/snakeoil.key  

make release-server  

GOOS=linux GOARCH=386 make release-client  
GOOS=linux GOARCH=amd64 make release-client  
GOOS=windows GOARCH=386 make release-client  
GOOS=windows GOARCH=amd64 make release-client  
GOOS=darwin GOARCH=386 make release-client  
GOOS=darwin GOARCH=amd64 make release-client  
GOOS=linux GOARCH=arm make release-client  

mkdir -p bin/logs/  
touch bin/authtoken.conf  


authtoken.conf格式为：token=subdomain1,subdomain2,subdomain3......  
示例：  
  test-bUSZTh6j=test,test2  

服务端运行: cd /ngrok/bin/ && ./ngrokd -domain="example.com"  -log=logs/run.log  


客户端在/ngrok/bin/下对应平台的子目录中。客户端配置文件示例：config.yml  
server_addr: example.com:4443  
auth_token: test-bUSZTh6j  
tunnels:  
  &nbsp;&nbsp;webapp:  
    &nbsp;&nbsp;&nbsp;&nbsp;subdomain: test  
    &nbsp;&nbsp;&nbsp;&nbsp;proto:  
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;http: "80"  

  CMD下运行 ngrok.exe -config=config.yml start-all
  
