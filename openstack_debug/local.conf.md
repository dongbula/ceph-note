```
#rpm不gpgcheck
(root)$ sudo sed -i  s'/gpgcheck=1/gpgcheck=0'/g  /etc/yum.repos.d/*.repo
# 配置PIP
(root)$ mkdir  ~/.pip/ && cat >>  ~/.pip/pip.conf <<EOF
[global]
proxy = http://localhost:8123
timeout = 300
index-url = http://pypi.python.org/simple

[install]
trusted-host = pypi.python.org
EOF
# 配置curl代理
cat > ~/.curlrc <<EOF
proxy = http://localhost:8123
EOF

# 配置git代理(用于无法访问外网的机器,能连接外网的机器不用配置)
cat > ~/.gitconfig <<EOF
[http]
   proxy = http://localhost:8123
[https]
   proxy = http://localhost:8123
EOF

# 配置yum代理(用于无法访问外网的机器,能连接外网的机器不用配置)
cat >> /etc/yum.conf  <<EOF
proxy = http://localhost:8123
EOF

# 配置wget代理(用于无法访问外网的机器,能连接外网的机器不用配置)
cat >> /etc/wgetrc <<EOF
http_proxy = http://localhost:8123
https_proxy = http://localhost:8123
ftp_proxy = http://localhost:8123
EOF

# 安装sock5 -> http代理
yum install -y https://copr-be.cloud.fedoraproject.org/results/jasonbrooks/polipo/epel-7-x86_64/polipo-1.1.1-2.fc22/polipo-1.1.1-2.el7.centos.x86_64.rpm
#启动代理转换,localhost:8123
polipo socksParentProxy=127.0.0.1:1080
# 测试代理是否可用
http_proxy=http://localhost:8123 https_proxy=http://localhost:8123 wget http://www.youtube.com


###vi /opt/devstack/stackrc
###export http_proxy=http://127.0.0.1:8123
###export https_proxy=http://127.0.0.1:8123
###export no_proxy=127.0.0.1,192.168.153.159,git.trystack.cn


(root)$ cd /opt/
(root)$ git clone  http://git.trystack.cn/openstack-dev/devstack.git  
(root)$ devstack/tools/create-stack-user.sh
(root)$ chown -R stack:stack devstack
(root)$ su - stack
(stack)$ cd /opt/devstack
(stack)$ sudo yum install python-pip openssl-devel gcc -y
(stack)$ sudo pip install --upgrade pip
(stack)$ sudo pip install -U os-testr
(stack)$ echo "127.0.0.1 `hostname`" | sudo tee /etc/hosts

#fix error
#Traceback (most recent call last):
#  File "/bin/generate-subunit", line 7, in <module>
#    from os_testr.generate_subunit import main
#ImportError: No module named os_testr.generate_subunit
(stack)$ sudo setfacl -m u:stack:rwx -R  /usr/lib/python2.7/site-packages/*
(stack)$ sudo setfacl -m u:stack:rwx -R  /usr/lib64/python2.7/site-packages/*

#userdel --remove stack 删除stack用户，如果失败想重来的话
```
部署单节点swift

(stack)$vi /opt/devstack/local.conf
```
[[local|localrc]]
GIT_BASE=http://git.trystack.cn
VERSION=master
SWIFT_REPO=$GIT_BASE/openstack/swift.git
SWIFT_BRANCH=$VERSION
#OFFLINE=True  
#RECLONE=no
ADMIN_PASSWORD=devstack
MYSQL_PASSWORD=devstack
SERVICE_PASSWORD=devstack
SERVICE_TOKEN=devstack
DATABASE_PASSWORD=stackdb
RABBIT_PASSWORD=stackqueue
LOGFILE=$DEST/logs/stack.sh.log
LOGDAYS=2
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5
SWIFT_REPLICAS=1
disable_all_services
enable_service key swift mysql
SWIFT_DATA_DIR=$DEST/data
```
```
开始部署
./stack.sh

sudo swift -A http://localhost:8080/auth/v1.0 -U swiftprojecttest1:swiftusertest3 -K testing3 stat
                        Account: TEMPAUTH_swiftprojecttest1
                     Containers: 2
                        Objects: 0
                          Bytes: 0
Containers in policy "policy-0": 2
   Objects in policy "policy-0": 0
     Bytes in policy "policy-0": 0
         X-Openstack-Request-Id: tx2c74163dd1c540fb86e11-005966209b
                    X-Timestamp: 1499864992.62258
                     X-Trans-Id: tx2c74163dd1c540fb86e11-005966209b
                   Content-Type: text/plain; charset=utf-8
                  Accept-Ranges: bytes
swift 命令参考 ：http://blog.bit-isle.jp/bird/2013/01/50
```


以下是部署整个openstack的local.conf
```
[[local|localrc]]
GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
SPICE_REPO=http://git.trystack.cn/git/spice/spice-html5.git

ADMIN_PASSWORD=stack
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
MYSQL_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=111222333444

HOST_IP=192.168.153.200
SERVICE_HOST=$HOST_IP

OS_PROJECT_NAME=demo
OS_USERNAME=demo
OS_PASSWORD=password
OS_AUTH_URL=http://$SERVICE_HOST:5000/v2.0

DEST=/opt/stack
#RECLONE=yes
PIP_UPGRADE=True
#OFFLINE=True

VERSION=master
NOVNC_BRANCH=v0.6.2
#VERSION=stable/ocata
KEYSTONE_REPO=$GIT_BASE/openstack/keystone.git
KEYSTONE_BRANCH=$VERSION

HORIZON_REPO=$GIT_BASE/openstack/horizon.git
HORIZON_BRANCH=$VERSION

NOVA_REPO=$GIT_BASE/openstack/nova.git
NOVA_BRANCH=$VERSION

NEUTRON_REPO=$GIT_BASE/openstack/neutron.git
NEUTRON_BRANCH=$VERSION

GLANCE_REPO=$GIT_BASE/openstack/glance.git
GLANCE_BRANCH=$VERSION

CINDER_REPO=$GIT_BASE/openstack/cinder.git
CINDER_BRANCH=$VERSION

disable_all_services
enable_service mysql
enable_service rabbit
enable_service key
REGION_NAME=RegionOne

# Disabling Identity API v2
#ENABLE_IDENTITY_V2=False

##### Horizon - Dashboard Service #####
#--------------------------------------
enable_service horizon
#enable_service +=,n-api,n-crt,n-obj,n-cpu,n-cond,n-sch,n-novnc,n-cauth,placement-api
enable_service +=,n-api,n-crt,n-obj,n-cpu,n-cond,n-sch,n-novnc,n-cauth
disable_service n-net
enable_service neutron
# Neutron options
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta

Q_USE_SECGROUP=True
FLOATING_RANGE=192.168.130.0/24
FIXED_RANGE=10.0.0.0/24
Q_FLOATING_ALLOCATION_POOL=start=192.168.130.201,end=192.168.130.210
PUBLIC_NETWORK_GATEWAY=192.168.130.1
PUBLIC_INTERFACE=ens35
Q_USE_PROVIDERNET_FOR_PUBLIC=True

# Neutron ML2 with OpenvSwitch
Q_PLUGIN=ml2
Q_AGENT=openvswitch
OVS_PHYSICAL_BRIDGE=br-ex
PUBLIC_BRIDGE=br-ex
OVS_BRIDGE_MAPPINGS=public:br-ex

IMAGE_URLS=http://download.cirros-cloud.net/0.3.2/cirros-0.3.2-x86_64-uec.tar.gz
enable_service g-api
enable_service g-reg
enable_service +=,cinder,c-api,c-vol,c-sch,c-bak
##### Logging #####
#------------------
LOGFILE=$DEST/logs/stack.sh.log
LOGDIR=$DEST/logs
LOGDAYS=1
LOG_COLOR=False

```
