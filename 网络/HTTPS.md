# HTTPS
## https 传输过程
- 客户端先从服务器获取到证书，证书中包含公钥
- 客户端对证书进行校验
- 客户端生成对称秘钥，用证书中的公钥进行加密，发送给服务器
- 服务器得到这个请求后用私钥进行解密，得到秘钥
- 客户端发出的请求都用对称秘钥进行加密
- 服务器收到这个密文用这个秘钥解密