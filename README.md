## sys_init
**常见系统初始化 和 软件初始化**  

### 1.系统初始化
1. sys init (CentOS)
```bash
# 开机启动项文件
chmod +x /etc/rc.d/rc.local

# limits.conf 
rm -rf /etc/security/limits.d/*
echo '* - nproc  65535'|tee -a /etc/security/limits.conf
echo '* - nofile 65535'|tee -a /etc/security/limits.conf

# firewalld off
systemctl stop firewalled && systemctl disable firewalled
iptables -F

setenforce 0
sed -i s/'SELINUX=enforcing'/'SELINUX=disabled'/g /etc/selinux/config

# YUM INSTALL
yum clean all
yum makecache
 
yum -y groupinstall "Development Tools"
yum -y install make cmake bison-devel bzip2-devel zlib zlib-devel \
openssl openssl-devel openssl-libs openssl-static pcre pcre-devel pcre-static \
ncurses ncurses-devel ncurses-libs curl-devel expat-devel gettext-devel \
openldap openldap-devel readline readline-devel readline-static libssh2 libssh2-devel \
unixODBC unixODBC-devel sqlite sqlite-devel tcl tcl-devel perl-Digest-SHA1 \
python-libs python-devel python2-pip python-crypto perl-libs perl-ExtUtils-MakeMaker \
GeoIP GeoIP-devel gperftools gperftools-devel gperftools-libs libatomic_ops-devel \
gtest gtest-devel gdk-pixbuf2 gdk-pixbuf2-deve libffi libffi-devel libcurl libcurl-devel http-parser http-parser-devel \
libxml2* libmcrypt* libtool-ltdl-devel* 
yum -y install bash-completion fop lftp ntp ntpdate vim wget telnet dstat \
tree lrzsz net-tools nmap-ncat nmap sysstat dmidecode bc screen psmisc dos2unix \
lzop lftp p7zip expect supervisor mysql nscd

# NTP
systemctl stop ntpd

[ -f /etc/localtime ] && cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[ -f /etc/sysconfig/clock ] && echo 'ZONE="Asia/Shanghai"' | tee /etc/sysconfig/clock
[ -f /etc/timezone ] && echo 'Asia/Shanghai' | tee /etc/timezone
[ -f /etc/sysconfig/ntpd ] && echo 'SYNC_HWCLOCK=yes' | tee -a /etc/sysconfig/ntpd

ntpdate cn.pool.ntp.org

cp -f /etc/{ntp.conf,ntp.conf.bak}
cat > /etc/ntp.conf <<EOF
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
server cn.pool.ntp.org prefer
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
EOF

cp -f /etc/ntp/{step-tickers,step-tickers.bak}
cat > /etc/ntp/step-tickers <<EOF
cn.pool.ntp.org
0.centos.pool.ntp.org
1.centos.pool.ntp.org
2.centos.pool.ntp.org
3.centos.pool.ntp.org
EOF

systemctl start ntpd && systemctl enable ntpd

# nscd(缓存服务)
systemctl start nscd && systemctl enable nscd
nscd -g
```

2. sys init (Mac OS)
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew install mariadb
brew install redis
brew install tree

mysql.server start
mysql -uroot

redis-server /usr/local/etc/redis.conf
redis-cli
```


### 2.命令别名
1.命令别名存放位置
```text
请将别名放到~/.bash_profile、~/.bashrc文件内，即当前用户生效。
请将别名放到/etc/profile、/etc/profile.d/alias.sh文件内，即全局生效。

请根据操作系统类型，选择相应的全局或用户环境变量文件。
```

2.常用别名命令
```bash
cat >> ～/.bash_profile <<EOF
alias tree='tree -N'
alias ll='ls -l'
EOF
```

### 3.vim init
```bash
echo 'set ts=4' | tee -a /etc/vimrc
echo 'set expandtab' | tee -a /etc/vimrc  
```

### 4.git init
```bash
git config --global user.name "penn"
git config --global user.email "smallasa@sina.com"
git config --global push.default simple
git config --global core.quotepath false
git config --global credential.helper store --file=.git-credentials
git config --global core.editor vim
git config --global merge.tool vimdiff
```

### 5.python
```bash
pyenv:
git clone https://github.com/yyuu/pyenv.git ~/.pyenv
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' | tee -a /etc/profile.d/pyenv.sh
echo 'eval "$(pyenv init -)"' | tee -a /etc/profile.d/pyenv.sh
source /etc/profile
exec $SHELL

git clone https://github.com/yyuu/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' | tee -a /etc/profile.d/pyenv.sh
source /etc/profile
exec $SHELL

pyevn install --list
pyenv install 3.6.6
pyenv global 3.6.6


pip:
mkdir -p ~/.pip
cat > ~/.pip/pip.conf <<EOF
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple/
[list]
format=columns
EOF
```

### 6.Java
```bash
tar xzf jdk-8u161-linux-x64.tar.gz -C /usr/local
mv /usr/local/jdk1.8.0_161 /usr/local/java
chown -R root.root /usr/local/java
rm -f jdk-8u161-linux-x64.tar.gz

echo 'export JAVA_HOME=/usr/local/java' | tee  /etc/profile.d/java.sh
echo 'export JRE_HOME=${JAVA_HOME}/jre' | tee -a /etc/profile.d/java.sh
echo 'export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib' | tee -a /etc/profile.d/java.sh
echo 'export PATH=${JAVA_HOME}/bin:$PATH' | tee -a /etc/profile.d/java.sh
source /etc/profile

java -version
```

### 7.Go
```bash
tar xzf go1.10.3.linux-amd64.tar.gz -C /usr/local/
chown -R root:root /usr/local/go
rm -f go1.10.3.linux-amd64.tar.gz

echo 'export GOROOT=/usr/local/go'|tee /etc/profile.d/go.sh
echo 'export GOPATH=$HOME/work'|tee -a /etc/profile.d/go.sh
echo 'export PATH=$GOROOT/bin:$PATH' |tee -a /etc/profile.d/go.sh
source /etc/profile

go version
```

### 8.Node
```bash
git clone https://github.com/creationix/nvm.git /usr/local/nvm

cat > /etc/profile.d/nvm.sh <<EOF
export NVM_DIR="/usr/local/nvm"
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
[ -s "\$NVM_DIR/nvm.sh" ] && \. "\$NVM_DIR/nvm.sh"
[ -s "\$NVM_DIR/bash_completion" ] && \. "\$NVM_DIR/bash_completion"
EOF
source /etc/profile

nvm install v8.11.2
nvm use v8.11.2

npm install -g cnpm
```

### 9.scala
```bash
tar xzf /usr/local/scala.2.12.4.tar.gz -C /usr/local/
chown -R root:root /usr/local/scala
rm -f /usr/local/scala.2.12.4.tar.gz

echo 'export SCALA_HOME=/usr/local/scala'|tee /etc/profile.d/scala.sh
echo 'export PATH=${SCALA_HOME}/bin:$PATH'|tee -a /etc/profile.d/scala.sh 
source /etc/profile
 
scala -version
```

### 10.ruby
```bash
rvm:
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
sed -i 's!ftp.ruby-lang.org/pub/ruby!cache.ruby-china.org/pub/ruby!g' $rvm_path/config/db

ruby:
rvm list known
rvm install 2.4
rvm docs generate-ri
rvm --default use 2.4
ruby -v

gem:
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
gem sources -l
gem update
```