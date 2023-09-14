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
make KCFLAGS="-pipe" -j$(nproc)
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
make LLVM=1 menuconfig



3.处理（可能）错误
移除字段
-fexcess-precision=fast

4.编译内核
make LLVM=1 KCFLAGS="-pipe" -j$(nproc)

5.生成deb包
make LLVM=1 deb-pkg -j$(nproc)
```

### 四、性能测试
使用以下命令
```
jitterdebugger -D 30m -c 'stress-ng --cpu-method loop -c 8'
```

#### 对比不同调度器内核
1、oracle arm64 a1
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
2、aws x86_64 cascade lake
```
6.5.2-xanmod1-gcc
T: 0 (  611) A: 0 C:   1799995 Min:         5 Avg:    6.95 Max:        73 
T: 1 (  612) A: 1 C:   1799995 Min:         5 Avg:    6.89 Max:      2316 
T: 2 (  613) A: 2 C:   1799993 Min:         5 Avg:    6.36 Max:        70 
T: 3 (  614) A: 3 C:   1799990 Min:         5 Avg:    6.88 Max:       108 
T: 4 (  615) A: 4 C:   1799988 Min:         5 Avg:   49.65 Max:       862 
T: 5 (  616) A: 5 C:   1799985 Min:         5 Avg:    6.78 Max:        40 
T: 6 (  617) A: 6 C:   1799983 Min:         5 Avg:    6.96 Max:        47 
T: 7 (  618) A: 7 C:   1799981 Min:         5 Avg:    6.62 Max:       164

6.5.2-tkg-eevdf-gcc
T: 0 (  545) A: 0 C:   1799999 Min:         5 Avg:    6.75 Max:        96 
T: 1 (  546) A: 1 C:   1799997 Min:         5 Avg:    6.31 Max:       157 
T: 2 (  547) A: 2 C:   1799995 Min:         5 Avg:    6.29 Max:        31 
T: 3 (  548) A: 3 C:   1799992 Min:         5 Avg:    6.48 Max:       129 
T: 4 (  549) A: 4 C:   1799990 Min:         5 Avg:    9.80 Max:       127 
T: 5 (  550) A: 5 C:   1799988 Min:         5 Avg:    7.96 Max:        48 
T: 6 (  551) A: 6 C:   1799986 Min:         5 Avg:    6.47 Max:        51 
T: 7 (  552) A: 7 C:   1799984 Min:         5 Avg:    6.73 Max:        81

6.5.2-tkg-eevdf-llvm
T: 0 (  549) A: 0 C:   1799998 Min:         4 Avg:   11.84 Max:       179 
T: 1 (  550) A: 1 C:   1799997 Min:         5 Avg:    6.12 Max:       309 
T: 2 (  551) A: 2 C:   1799995 Min:         5 Avg:    6.09 Max:        46 
T: 3 (  552) A: 3 C:   1799994 Min:         5 Avg:    6.18 Max:        90 
T: 4 (  553) A: 4 C:   1799992 Min:         5 Avg:    6.17 Max:      2550 
T: 5 (  554) A: 5 C:   1799991 Min:         5 Avg:    6.12 Max:        56 
T: 6 (  555) A: 6 C:   1799989 Min:         5 Avg:    6.08 Max:        62 
T: 7 (  556) A: 7 C:   1799988 Min:         5 Avg:    6.08 Max:       146

6.5.2-tkg-bore-eevdf-llvm
T: 0 (  563) A: 0 C:   1799999 Min:         4 Avg:    6.13 Max:        56 
T: 1 (  564) A: 1 C:   1799997 Min:         4 Avg:    6.04 Max:       131 
T: 2 (  565) A: 2 C:   1799996 Min:         4 Avg:    6.10 Max:        64 
T: 3 (  566) A: 3 C:   1799994 Min:         5 Avg:    6.22 Max:        58 
T: 4 (  567) A: 4 C:   1799993 Min:         4 Avg:    6.15 Max:        45 
T: 5 (  568) A: 5 C:   1799992 Min:         4 Avg:    6.07 Max:        57 
T: 6 (  569) A: 6 C:   1799990 Min:         4 Avg:    6.14 Max:        61 
T: 7 (  570) A: 7 C:   1799989 Min:         5 Avg:    6.08 Max:       139

6.5.2-tkg-bore-eevdf-gcc
T: 0 (  551) A: 0 C:   1799995 Min:         5 Avg:    6.59 Max:        93 
T: 1 (  552) A: 1 C:   1799995 Min:         5 Avg:    6.51 Max:       162 
T: 2 (  553) A: 2 C:   1799992 Min:         5 Avg:    6.89 Max:        46 
T: 3 (  554) A: 3 C:   1799990 Min:         5 Avg:   48.35 Max:       807 
T: 4 (  555) A: 4 C:   1799988 Min:         5 Avg:    6.50 Max:       141 
T: 5 (  556) A: 5 C:   1799986 Min:         5 Avg:    6.74 Max:        59 
T: 6 (  557) A: 6 C:   1799983 Min:         5 Avg:    6.73 Max:        41 
T: 7 (  558) A: 7 C:   1799981 Min:         5 Avg:    6.31 Max:       162

6.4.15-tkg-bmq-llvm
T: 0 (  570) A: 0 C:   1799998 Min:         4 Avg:    5.43 Max:       260 
T: 1 (  571) A: 1 C:   1799996 Min:         4 Avg:    7.20 Max:        58 
T: 2 (  572) A: 2 C:   1799995 Min:         4 Avg:    5.42 Max:       296 
T: 3 (  573) A: 3 C:   1799993 Min:         4 Avg:    7.15 Max:      2137 
T: 4 (  574) A: 4 C:   1799992 Min:         4 Avg:    5.57 Max:        72 
T: 5 (  575) A: 5 C:   1799991 Min:         4 Avg:    5.37 Max:        50 
T: 6 (  576) A: 6 C:   1799989 Min:         4 Avg:    5.03 Max:       329 
T: 7 (  577) A: 7 C:   1799988 Min:         4 Avg:    5.37 Max:        49

6.4.15-tkg-bmq-gcc
T: 0 ( 2906) A: 0 C:   1799997 Min:         5 Avg:   29.32 Max:       453 
T: 1 ( 2907) A: 1 C:   1799995 Min:         4 Avg:    7.37 Max:        74 
T: 2 ( 2908) A: 2 C:   1799993 Min:         4 Avg:    7.21 Max:        60 
T: 3 ( 2909) A: 3 C:   1799991 Min:         4 Avg:    7.13 Max:       133 
T: 4 ( 2910) A: 4 C:   1799988 Min:         4 Avg:    5.04 Max:       151 
T: 5 ( 2911) A: 5 C:   1799986 Min:         4 Avg:    7.47 Max:      2029 
T: 6 ( 2912) A: 6 C:   1799984 Min:         4 Avg:    5.42 Max:        45 
T: 7 ( 2913) A: 7 C:   1799982 Min:         4 Avg:    5.31 Max:       103

6.4.15-tkg-pds-gcc
T: 0 (  541) A: 0 C:   1799997 Min:         4 Avg:    5.13 Max:        83 
T: 1 (  542) A: 1 C:   1799995 Min:         4 Avg:    5.65 Max:        92 
T: 2 (  543) A: 2 C:   1799993 Min:         4 Avg:    5.67 Max:       152 
T: 3 (  544) A: 3 C:   1799990 Min:         4 Avg:    5.48 Max:        57 
T: 4 (  545) A: 4 C:   1799988 Min:         4 Avg:    5.30 Max:        51 
T: 5 (  546) A: 5 C:   1799986 Min:         4 Avg:    5.20 Max:        49 
T: 6 (  547) A: 6 C:   1799984 Min:         4 Avg:    5.65 Max:       133 
T: 7 (  548) A: 7 C:   1799982 Min:         4 Avg:    5.51 Max:       277

6.4.15-tkg-pds-llvm

T: 0 (  649) A: 0 C:   1799998 Min:         4 Avg:    5.32 Max:       749 
T: 1 (  650) A: 1 C:   1799996 Min:         4 Avg:    5.02 Max:        70 
T: 2 (  651) A: 2 C:   1799995 Min:         4 Avg:    5.48 Max:        61 
T: 3 (  652) A: 3 C:   1799994 Min:         4 Avg:    5.16 Max:       120 
T: 4 (  653) A: 4 C:   1799992 Min:         4 Avg:    5.51 Max:        57 
T: 5 (  654) A: 5 C:   1799991 Min:         4 Avg:    5.21 Max:        39 
T: 6 (  655) A: 6 C:   1799989 Min:         4 Avg:    5.60 Max:        51 
T: 7 (  656) A: 7 C:   1799988 Min:         4 Avg:    5.33 Max:        56
```

3、oci x86_64 zen
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
#### 结论：
1、使用LLVM Clang开启full LTO，对于linux内核相应速度略有提升。
2、默认内核调度器EEVDF较CFS更佳；
3、BMQ、PDS调度器延迟相应时间最低。
