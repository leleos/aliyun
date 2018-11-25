# 安装7zip

### 安装C++环境
```shell
yum install -y gcc-c++ 

yum install -y bzip2

yum install -y wget
```
### 安装7zip
```shell
wget  https://managedway.dl.sourceforge.net/project/p7zip/p7zip/16.02/p7zip_16.02_src_all.tar.bz2

tar -xjvf p7zip_16.02_src_all.tar.bz2
cd  p7zip_16.02
make
make install
```