# WAL-G built for AWS Graviton2

1. Launch a new ARM64 instance:
    * Ubuntu Server 20.04 LTS (HVM), SSD Volume Type
    * 64-bit (Arm)
    * t4g.2xlarge
    * 40 GB EBS volume
    * tags:
        * `Name: mediacloud-herewegoagain-build-arm64`
        * `project: mediacloud-herewegoagain`

2. SSH into the instance and run:

```bash
sudo hostnamectl set-hostname build-arm64

wget https://go.dev/dl/go1.17.5.linux-arm64.tar.gz
tar -zxf go1.17.5.linux-arm64.tar.gz
rm go1.17.5.linux-arm64.tar.gz
echo "export PATH=$HOME/go/bin:$PATH" >> ~/.bashrc
echo "export GOPATH=$HOME/go" >> ~/.bashrc
echo "export USE_LIBSODIUM=1" >> ~/.bashrc
source ~/.bashrc

sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get -y install gcc-11 g++-11

sudo update-alternatives --remove-all gcc
sudo update-alternatives --remove-all g++
sudo update-alternatives --remove-all cc
sudo update-alternatives --remove-all c++
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 10
sudo update-alternatives --set cc /usr/bin/gcc
sudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 10
sudo update-alternatives --set cc /usr/bin/gcc
sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 10
sudo update-alternatives --set c++ /usr/bin/g++

git clone https://github.com/wal-g/wal-g.git
cd wal-g/
git checkout v1.1
sudo apt-get install -y liblzo2-dev brotli libsodium-dev build-essential make
go get -v -t -d ./...
export CFLAGS="-march=armv8.2-a+fp16+rcpc+dotprod+crypto -mcpu=neoverse-n1 -fsigned-char"
make deps
sudo ln -s /usr/lib/aarch64-linux-gnu/libbrotlidec.a /usr/lib/aarch64-linux-gnu/libbrotlidec-static.a
sudo ln -s /usr/lib/aarch64-linux-gnu/libbrotlicommon.a /usr/lib/aarch64-linux-gnu/libbrotlicommon-static.a
sudo ln -s /usr/lib/aarch64-linux-gnu/libbrotlienc.a /usr/lib/aarch64-linux-gnu/libbrotlienc-static.a

make pg_build
mv main/pg/wal-g wal-g-pg-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-pg-ubuntu-20.04-arm64-graviton2 > wal-g-pg-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-pg-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-pg-ubuntu-20.04-arm64-graviton2.xz > wal-g-pg-ubuntu-20.04-arm64-graviton2.xz.sha256

make mysql_build
mv main/mysql/wal-g wal-g-mysql-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-mysql-ubuntu-20.04-arm64-graviton2 > wal-g-mysql-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-mysql-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-mysql-ubuntu-20.04-arm64-graviton2.xz > wal-g-mysql-ubuntu-20.04-arm64-graviton2.xz.sha256

make sqlserver_build
mv main/sqlserver/wal-g wal-g-sqlserver-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-sqlserver-ubuntu-20.04-arm64-graviton2 > wal-g-sqlserver-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-sqlserver-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-sqlserver-ubuntu-20.04-arm64-graviton2.xz > wal-g-sqlserver-ubuntu-20.04-arm64-graviton2.xz.sha256

make redis_build
mv main/redis/wal-g wal-g-redis-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-redis-ubuntu-20.04-arm64-graviton2 > wal-g-redis-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-redis-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-redis-ubuntu-20.04-arm64-graviton2.xz > wal-g-redis-ubuntu-20.04-arm64-graviton2.xz.sha256

make mongo_build
mv main/mongo/wal-g wal-g-mongo-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-mongo-ubuntu-20.04-arm64-graviton2 > wal-g-mongo-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-mongo-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-mongo-ubuntu-20.04-arm64-graviton2.xz > wal-g-mongo-ubuntu-20.04-arm64-graviton2.xz.sha256

make fdb_build
mv main/fdb/wal-g wal-g-fdb-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-fdb-ubuntu-20.04-arm64-graviton2 > wal-g-fdb-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-fdb-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-fdb-ubuntu-20.04-arm64-graviton2.xz > wal-g-fdb-ubuntu-20.04-arm64-graviton2.xz.sha256

make gp_build
mv main/gp/wal-g wal-g-gp-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-gp-ubuntu-20.04-arm64-graviton2 > wal-g-gp-ubuntu-20.04-arm64-graviton2.sha256
xz -kv9 wal-g-gp-ubuntu-20.04-arm64-graviton2
sha256sum wal-g-gp-ubuntu-20.04-arm64-graviton2.xz > wal-g-gp-ubuntu-20.04-arm64-graviton2.xz.sha256
```


3. Copy binaries to local machine:

```bash
scp mediacloud-herewegoagain-build-arm64:wal-g/'wal-g-*-ubuntu-20.04-arm64-graviton2*' .
```


4. Create a new GitHub release


## Links

* https://github.com/aws/aws-graviton-getting-started/blob/main/golang.md
