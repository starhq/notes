# openssl

```SHELL
# 生成 32 字节随机数
openssl rand -base64 32

# 指定长度生成
openssl rand -hex 16

# 生成有效期 365 天的证书
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

```