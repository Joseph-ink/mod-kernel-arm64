编译于oracle甲骨文arm机型

经测试延迟更低（但不排除样本量小和网络正常波动）；

**不建议主力生产机器使用，风险自行承担。**

操作步骤如下，可以自行编译（建议在纯净系统上编译内核）：

一、安装常用工具
```
apt-get update
apt-get install git jq htop iperf3 net-tools -y
```

二、准备编译环境
```
apt-get install build-essential libncurses-dev bison flex libssl-dev libelf-dev debhelper
```

三、获取内核源代码（目前版本6.3.9-xanmod1）
```
git clone -b 6.3.9-xanmod1 https://github.com/xanmod/linux.git
cd linux
```

四、配置内核
```
make menuconfig
```

五、（可能）处理错误
```
vi .config
```
移除字段
```
debian/canonical-certs.pem
debian/canonical-revoked-certs.pem
```

六、编译内核
```
make -j$(nproc)
```

七、生成deb包
```
make deb-pkg
```
