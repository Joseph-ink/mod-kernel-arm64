## 操作步骤如下，可以自行编译（建议在纯净系统上编译）

## 以xanmod内核编译为例：

### 一、条件准备

1.安装常用工具
```
apt-get update
apt-get install -y git jq htop iperf3 net-tools jitterdebugger stress-ng vim devscripts socat wget net-tools make pkg-config libmnl-dev libatm1-dev libbpf-dev libtirpc-dev libcap-dev libdb-dev software-properties-common dwarves
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
make KCFLAGS="-mcpu=neoverse-n1 -fomit-frame-pointer -pipe" -j$(nproc)
```

4.生成deb包
```
make deb-pkg -j$(nproc)
```

### 三、使用LLVM/Clang编译
```
1.安装最新LLVM/Clang
bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
设置环境变量优先调用
export PATH=/usr/lib/llvm-17/bin:$PATH

2.配置内核
make LLVM=1 LLVM_IAS=1 menuconfig



3.处理（可能）错误
移除字段
-fexcess-precision=fast

4.编译内核
make LLVM=1 LLVM_IAS=1 KCFLAGS="-mcpu=neoverse-n1 -fomit-frame-pointer -pipe" -j$(nproc)

5.生成deb包
make LLVM=1 LLVM_IAS=1 deb-pkg -j$(nproc)
```
### 四、性能测试
使用以下命令
```
jitterdebugger -D 30m -c 'stress-ng --cpu-method loop -c 8'
```
#### 对比不同调度器内核
一、oracle arm64 a1
```
6.4.12-tkg-cfs-llvm
T: 0 ( 1208) A: 0 C:   1799999 Min:         2 Avg:    9.63 Max:     50572 
T: 1 ( 1210) A: 1 C:   1799998 Min:         2 Avg:    7.59 Max:     42076 
T: 2 ( 1211) A: 2 C:   1799998 Min:         2 Avg:    9.69 Max:     38229

6.4.12-tkg-bore-llvm
T: 0 ( 1330) A: 0 C:   1799999 Min:         2 Avg:    6.29 Max:     19347 
T: 1 ( 1331) A: 1 C:   1799999 Min:         2 Avg:    5.82 Max:     18931 
T: 2 ( 1332) A: 2 C:   1799999 Min:         2 Avg:    6.19 Max:     19111


6.4.12-tkg-eevdf-llvm
T: 0 ( 1193) A: 0 C:   1799999 Min:         2 Avg:   14.78 Max:     47044 
T: 1 ( 1195) A: 1 C:   1799999 Min:         2 Avg:    8.69 Max:     37081 
T: 2 ( 1196) A: 2 C:   1799999 Min:         2 Avg:    8.00 Max:     40972

6.4.12-tkg-tt-llvm
T: 0 ( 1079) A: 0 C:   1800000 Min:         2 Avg:   10.85 Max:     48461 
T: 1 ( 1081) A: 1 C:   1799999 Min:         2 Avg:   12.28 Max:     39420 
T: 2 ( 1082) A: 2 C:   1799999 Min:         2 Avg:    6.61 Max:     38230
```
二、aws x86_64 cascade lake
```
linux 6.4.0
T: 0 ( 8353) A: 0 C:   1799996 Min:         5 Avg:    6.83 Max:      1178 
T: 1 ( 8354) A: 1 C:   1799993 Min:         5 Avg:    6.80 Max:       858

linux 6.4.11 bmq
T: 0 (  462) A: 0 C:   1799998 Min:         4 Avg:    5.58 Max:       125 
T: 1 (  463) A: 1 C:   1799996 Min:         4 Avg:    5.32 Max:       123

linux 6.4.11 pds
T: 0 (  461) A: 0 C:   1799998 Min:         4 Avg:    6.44 Max:        88 
T: 1 (  462) A: 1 C:   1799996 Min:         4 Avg:    5.35 Max:      2762


linux 6.4.11 tt
T: 0 (  470) A: 0 C:   1799998 Min:         5 Avg:    6.02 Max:      2781 
T: 1 (  471) A: 1 C:   1799996 Min:         4 Avg:    5.97 Max:       152


linux 6.4.11 eevdf
T: 0 (  472) A: 0 C:   1800001 Min:         4 Avg:    6.03 Max:      1265 
T: 1 (  473) A: 1 C:   1800000 Min:         4 Avg:    6.71 Max:       112


linux 6.4.11 bore
T: 0 (  547) A: 0 C:   1799997 Min:         6 Avg:    6.46 Max:       630 
T: 1 (  548) A: 1 C:   1799995 Min:         5 Avg:    6.31 Max:       100 

linux 6.4.11 xanmod
T: 0 (  547) A: 0 C:   1799996 Min:         5 Avg:    6.05 Max:       155 
T: 1 (  548) A: 1 C:   1799994 Min:         4 Avg:    6.02 Max:       161

linux 6.1.46 xanmod rt
T: 0 (  565) A: 0 C:   1799996 Min:         5 Avg:    6.05 Max:       332 
T: 1 (  566) A: 1 C:   1799994 Min:         5 Avg:    6.02 Max:       395
```

三、oci x86_64 zen
```
6.4.13-tkg-eevdf-llvm
T: 0 ( 1772) A: 0 C:   1800000 Min:         7 Avg:    9.08 Max:      3043 
T: 1 ( 1773) A: 1 C:   1799998 Min:         7 Avg:    9.08 Max:      6854 
T: 2 ( 1774) A: 2 C:   1799996 Min:         8 Avg:    9.06 Max:      2460 
T: 3 ( 1775) A: 3 C:   1799994 Min:         7 Avg:    9.43 Max:      5304

6.4.12-tkg-tt-llvm
T: 0 ( 1615) A: 0 C:   1799998 Min:         7 Avg:    8.87 Max:      3719 
T: 1 ( 1616) A: 1 C:   1799996 Min:         6 Avg:   86.72 Max:     39957 
T: 2 ( 1617) A: 2 C:   1799995 Min:         7 Avg:    8.86 Max:      4598 
T: 3 ( 1618) A: 3 C:   1799993 Min:         7 Avg:    9.79 Max:     14511

6.4.12-tkg-bore-llvm
T: 0 ( 1630) A: 0 C:   1799998 Min:         7 Avg:    9.13 Max:      2640 
T: 1 ( 1631) A: 1 C:   1799996 Min:         7 Avg:    9.12 Max:      1675 
T: 2 ( 1632) A: 2 C:   1799994 Min:         7 Avg:    9.13 Max:      1750 
T: 3 ( 1633) A: 3 C:   1799992 Min:         7 Avg:    9.42 Max:      4921

6.4.12-tkg-pds-llvm
T: 0 ( 1595) A: 0 C:   1799998 Min:         6 Avg:   12.16 Max:      4081 
T: 1 ( 1596) A: 1 C:   1799996 Min:         6 Avg:    8.51 Max:     24043 
T: 2 ( 1597) A: 2 C:   1799995 Min:         6 Avg:   66.65 Max:     40221 
T: 3 ( 1598) A: 3 C:   1799993 Min:         6 Avg:    8.25 Max:      4336

6.4.12-tkg-bmq-llvm
T: 0 ( 1624) A: 0 C:   1799998 Min:         6 Avg:    7.78 Max:      2282 
T: 1 ( 1625) A: 1 C:   1799996 Min:         6 Avg:    8.22 Max:      5463 
T: 2 ( 1626) A: 2 C:   1799995 Min:         6 Avg:    7.61 Max:      2994 
T: 3 ( 1627) A: 3 C:   1799993 Min:         6 Avg:    8.14 Max:      5417
```
