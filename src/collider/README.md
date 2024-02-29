# Collider

A websocket-based signaling server in Go.

## Building

1. Install the Go tools and workspaces as documented at http://golang.org/doc/install and http://golang.org/doc/code.html

2. Checkout the `apprtc` repository

```bash
git clone https://github.com/webrtc/apprtc.git
sudo apt install golang
which go
##> /usr/bin/go
```

3. Make sure to set the $GOPATH according to the Go instructions in step 1
```bash
mkdir $HOME/goWorkspace/
  E.g. `export GOPATH=$HOME/goWorkspace/`
mkdir $GOPATH/src
cd $GOPATH/src
```

4. Link the collider directories into `$GOPATH/src
> go version
go version go1.18.1 linux/amd64
> go env
```log
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/hieutt/.cache/go-build"
GOENV="/home/hieutt/.config/go/env"
GOEXE=""
GOEXPERIMENT=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GOMODCACHE="/home/hieutt/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/hieutt/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/lib/go-1.18"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/lib/go-1.18/pkg/tool/linux_amd64"
GOVCS=""
GOVERSION="go1.18.1"
GCCGO="gccgo"
GOAMD64="v1"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/home/hieutt/goWorkspace/src/collider/go.mod"
GOWORK=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build3534521085=/tmp/go-build -gno-record-gcc-switches"
```
- old lib:
> https://github.com/golang/go/tree/master/src/crypto/tls

```bash 
cd apprtc
ln -s `pwd`/src/collider $GOPATH/src
```

5. Install dependencies
```bash
cd $GOPATH/src/collider
## download lib
go mod download
## build:
go install
```

## Running test

> /home/hieutt/go/bin/collider -port=8089 -tls=false

2024/02/29 18:20:09 Starting collider: tls = false, port = 8089, room-server=https://appr.tc

## Deployment
These instructions assume you are using Ubuntu 20.04 and go1.18.1 linux/amd64

1. Change [roomSrv](https://github.com/webrtc/apprtc/blob/master/src/collider/collider/main.go#L16) to your AppRTC server instance e.g.

```go
var roomSrv = flag.String("room-server", "https://your.apprtc.server", "The origin of the room server")
```

2. Then repeat step 6 in the Building section.

### Install Collider
1. Login on the machine that is going to run Collider.
2. Create a Collider directory, this guide assumes it's created in the root (`/collider`).
3. Create a certificate directory, this guide assumes it's created in the root (`/cert`).
4. Copy `$GOPATH/bin/collider ` from your development machine to the `/collider` directory on your Collider machine.

### Certificates
If you are deploying this in production, you should use certificates so that you can use secure websockets. Place the `cert.pem` and `key.pem` files in `/cert/`. E.g. `/cert/cert.pem` and `/cert/key.pem`

### Auto restart
1. Add a `/collider/start.sh` file:

```bash
#!/bin/sh -
/collider/collider 2>> /collider/collider.log
```

2. Make it executable by running `chmod 744 start.sh`.

#### If using inittab otherwise jump to step 5:

3. Add the following line to `/etc/inittab` to allow automatic restart of the Collider process (make sure to either add `coll` as an user or replace it below with the user that should run collider):
```bash
coll:2:respawn:/collider/start.sh
```
4. Run `init q` to apply the inittab change without rebooting.

#### If using systemd:

5. Create a service by doing `sudo nano /lib/systemd/system/collider.service` and adding the following:

```
[Unit]
Description=AppRTC signalling server (Collider)
 
[Service]
ExecStart=/collider/start.sh
StandardOutput=null
 
[Install]
WantedBy=multi-user.target
Alias=collider.service
```
6. Enable the service: `sudo systemctl enable collider.service`

7. Verify it's up and running: `sudo systemctl status collider.service`


#### Rotating Logs
To enable rotation of the `/collider/collider.log` file add the following contents to the `/etc/logrotate.d/collider` file:

```
/collider/collider.log {
    daily
    compress
    copytruncate
    dateext
    missingok
    notifempty
    rotate 10
    sharedscripts
}
```

The log is rotated daily and removed after 10 days. Archived logs are in `/collider`.
