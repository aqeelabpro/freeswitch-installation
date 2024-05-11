# FreeSWITCH 1.10 installation guide from source on Debian 11


### Create SignalWire Account by visting https://signalwire.com/ and Giving some Details

## Get a token from SignalWire

Go to https://mentor.signalwire.com/dashboard and scroll Down to ``` Personal Access Token ```

<img width="1440" alt="signalwire_pat" src="https://github.com/aqeelabpro/freeswitch-installation/assets/93031839/17c6b75d-b950-46df-a856-1da481165e82">  


```
TOKEN=pat_FUpmJtRf8ZVBuaAhKP2AyERK 
apt-get update && apt-get install -yq gnupg2 wget lsb-release
wget --http-user=signalwire --http-password=$TOKEN -O /usr/share/keyrings/signalwire-freeswitch-repo.gpg https://freeswitch.signalwire.com/repo/deb/debian-release/signalwire-freeswitch-repo.gpg
echo "machine freeswitch.signalwire.com login signalwire password $TOKEN" > /etc/apt/auth.conf
echo "deb [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list
echo "deb-src [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list
apt-get -y update
apt-get -y build-dep freeswitch
cd /usr/src
git clone https://github.com/signalwire/freeswitch.git -bv1.10 freeswitch
cd freeswitch
git config pull.rebase true
./bootstrap.sh -j
./configure
make
make install
make cd-sounds-install cd-moh-install
```

# Post-installation
# Set Owner and Permissions
```
cd /usr/local
groupadd freeswitch
adduser --quiet --system --home /usr/local/freeswitch --gecos "FreeSWITCH open source softswitch" --ingroup freeswitch freeswitch --disabled-password
chown -R freeswitch:freeswitch /usr/local/freeswitch/
chmod -R ug=rwX,o= /usr/local/freeswitch/
chmod -R u=rwx,g=rx /usr/local/freeswitch/bin/*
```
# configure systems
```
 nano /etc/systemd/system/freeswitch.service
```

```
[Service]
; service
Type=forking
PIDFile=/usr/local/freeswitch/run/freeswitch.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /usr/local/freeswitch/run
ExecStartPre=/bin/chown freeswitch:daemon /usr/local/freeswitch/run
ExecStart=/usr/local/freeswitch/bin/freeswitch -ncwait -nonat
TimeoutSec=45s
Restart=always
; exec
WorkingDirectory=/usr/local/freeswitch/run
User=freeswitch
Group=daemon
LimitCORE=infinity
LimitNOFILE=100000
LimitNPROC=60000
;LimitSTACK=240
LimitRTPRIO=infinity
LimitRTTIME=7000000
IOSchedulingClass=realtime
IOSchedulingPriority=2
CPUSchedulingPolicy=rr
CPUSchedulingPriority=89
UMask=0007
[Install]
WantedBy=multi-user.target
```

enable and start freeswitch service

```
systemctl daemon-reload
systemctl start freeswitch
systemctl enable freeswitch
```

check if it is running

```
ps aux | grep freeswitch
```

# Set fs_cli bash profile

```
nano ~/.bash_profile
```

```
PATH=$PATH:$HOME/bin
PATH=$PATH:/usr/local/freeswitch/bin
export PATH
unset USername
```

# DNS cashing
```
apt -y install unbound
systemctl start unbound
systemctl enable unbound
```

# Proper entropy source

```
apt -y install haveged
systemctl start haveged
systemctl enable haveged
```


# Automatic time synchronization
```
apt -y install chrony   # installation
systemctl start chrony  # start
systemctl enable chrony # auto start
```
