## 操作步骤如下，可以自行编译（建议在纯净系统上编译）

## 以xanmod内核编译为例：

### 一、条件准备

1.安装常用工具
```
apt-get update
apt-get install -y git jq htop iperf3 net-tools stress-ng vim devscripts socat wget net-tools make pkg-config libmnl-dev libatm1-dev libbpf-dev libtirpc-dev libcap-dev libdb-dev software-properties-common dwarves
```

2.准备编译环境
```
apt-get install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev debhelper bc ccache cpio fakeroot kmod libncurses5-dev lz4 qtbase5-dev rsync schedtool zstd clang llvm lld
```

3.获取内核源代码（目前版本6.4.11-xanmod1）
```
git clone -b 6.4.11-xanmod1 https://github.com/xanmod/linux.git
cd linux
```

### 二、使用默认GNU/GCC编译

1.配置内核
```
make menuconfig
```

2.处理（可能）错误
```
vi .config
```
移除字段
```
debian/canonical-certs.pem
debian/canonical-revoked-certs.pem
```

3.编译内核
建议方法
```
make -j$(nproc)
```

4.生成deb包
```
make deb-pkg
```

### 三、使用LLVM/Clang编译
```
1.安装最新LLVM/Clang
bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
设置环境变量优先调用
export PATH=/usr/lib/llvm-17/bin:$PATH

2.配置内核
make LLVM=1 menuconfig

3.处理（可能）错误
移除字段
-fexcess-precision=fast

4.编译内核
make LLVM=1 -j$(nproc)

5.生成deb包
make LLVM=1 deb-pkg
```
