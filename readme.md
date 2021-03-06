### acme-lego

Go 版本 ACME 客户端，支持 ACMEv2 协议，支持 ECC 证书

##### 为什么是 Go 版本

Go 版本用来写类似的工具使用起来及其方便，只需将编译后的二进制文件下载放在服务器执行就可以，不额外依赖其他环境，不需要安装 Go 语言环境

##### 开始使用

1、下载源码，拉取依赖并编译二进制（比较懒，后面会提供编译好的二进制 release 版本供下载）

```bash
go build -o bin/lego main.go
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o bin/lego main.go
```

2、建立 Lego 默认配置目录 `/etc/lego/`，或者建立在其他目录的话，后面执行 lego 调用需要传入 path 参数为配置目录（或者说 path 参数默认为 `/etc/lego/` ）

在配置目录下建立 `config.json` 配置文件，如下

```json
{
    "email": "acme@example.com", // 用于账户注册的邮箱
    "key-type": ["rsa2048", "ec256"], // 全局支持的证书类型，
    "challenge": "http-path", // 全局支持的验证方式
    "domain-group": { // 域名配置
        "a.example.com": { // 验证的域名
            "domains": ["a12.example.com", "a22.example.com"], // 支持多个域名申请一个证书
            "key-type": ["rsa2048", "ec256"], // 针对当前域名的证书类型
            "challenge": "http-path", // 针对当前域名的验证方式
            "http-path": "/web-path/certificate/acme" // 如果是 http-path 验证，临时文件的位置
        },
        "b.example.com": {
            ...
        }
    },
    "expire-days": 30, // 在临过期多久前执行续签
    "after-renew": "systemctl reload nginx" // 续签成功后执行的命令
}
```

3、建立配置文件后，依次执行

```bash
lego reg # 注册账户
lego run # 执行申请证书
```

就可以将配置的域名申请证书

4、也可以在配置文件不存在情况下手动执行

```bash
lego reg --email="acme@example.com" # 用传入的邮箱执行账户申请
```

5、或者忽略配置，为单个域名执行单个证书的申请

```bash
lego run --domain="c.example.com" --type ecc # 执行 c.example.com 域名的 ECC 证书申请
```

6、域名续签，由于执行文件没有做为服务在后台守护，所以需要手动添加 crontab 任务定时执行下面命令

```bash
lego renew
```

或者单个域名续签

```bash
lego renew --domain="c.example.com"
```

##### 配置目录结构

```
lego/
    account/
        account.json # 账户信息
        account.key # 账户私钥
    certificates/
        a.example.com/ # 每个域名单独目录
            fullchain.ecdsa-256.crt # ecc 公钥
            fullchain.rsa-2048.crt # rsa 公钥
            meta.ecdsa-256.json # ecc 数据文件
            meta.rsa-2048.json # rsa 数据文件
            privkey.ecdsa-256.key # ecc 私钥
            privkey.rsa-2048.key # rsa 私钥
        b.example.com/

```

### 感谢

基于 [github.com/xenolf/lego](https://github.com/xenolf/lego) 开发
