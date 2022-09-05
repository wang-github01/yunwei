```
## Configuration file of Harbor

# hostname设置访问地址，可以使用ip、域名，不可以设置为127.0.0.1或localhost，此处我设置为本地ip

hostname = 172.16.50.37

# Harbor启动后，管理员UI登录的密码，默认是Harbor12345

harbor_admin_password = Harbor12345

# 认证方式，这里支持多种认证方式，如LADP、本次存储、数据库认证。默认是db_auth，mysql数据库认证

auth_mode = db_auth

# 是否开启自注册 

self_registration = on

# Token有效时间，默认30分钟

token_expiration = 30
```

