OS: Centos 6.10
#Link: https://ivanblagojevic.com/2017/12/how-to-install-wordpress-on-centos-6-server/
#install and run Apache web server
yum install httpd
service httpd start
#install mysql server
yum install mysql-server
service mysqld start
#Set the root password for MySQL
#After installing MySQL, you need to set the root password for MySQL. Open the console that is connected to the server via SSH and type:
/usr/bin/mysql_secure_installation
#Install PHP
yum install php php-mysql
# Install PHP extensions
yum install php-mysql php-pdo php-pear php-pecl php-xml php-gd php-zlib
# Let httpd and mysqld run together with the system
chkconfig httpd on
chkconfig mysqld on
#Make sure PHP is working
nano /var/www/html/info.php
# write and save
<?php
phpinfo();
?>
#restart php
service httpd restart
#goto http://yoursite/info.php
#download lastest vertion of wordpress
wget http://wordpress.org/latest.tar.gz
#unzip
tar -xzvf latest.tar.gz
#Create a new database with the name as you wish. I put wordpress as an example:
mysql -u root -p
CREATE DATABASE wordpress;
#create new username
CREATE USER nobjta_9x_tq@localhost;
#set passwd
SET PASSWORD FOR nobjta_9x_tq@localhost= PASSWORD("Canhth0ng");
#Give all the privileges to the user:
GRANT ALL PRIVILEGES ON wordpress.* TO nobjta_9x_tq@localhost IDENTIFIED BY 'Canhth0ng';
#Refresh MySQL:
FLUSH PRIVILEGES;
###  Set WordPress
cp ~/wordpress/wp-config-sample.php ~/wordpress/wp-config.php
#edit config following:
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');
/** MySQL database username */
define('DB_USER', 'user');
/** MySQL database password */
define('DB_PASSWORD', 'password');
/** MySQL hostname */
define('DB_HOST', 'localhost');

##Step 11: Transfer Wordpres files to /var/www/html
cp -r ~/wordpress/* /var/www/html
service httpd restart

# FINAL install wordpress
# goto: yoursite/wp-admin/install.php

#install git and add all in html to git
# REMOVE info.php
# D O N E

# Copy source to /var/www/html
# Edit config in wp-config.php for connect to new host
# Dump database wordpress to file.sql
# Copy to server and run:
mysql -u nobjta_9x_tq -p wordpress < Dump20190302.sql
# IF error collection utf8:
## Replace
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_520_ci;
## utf8mb4 and utf8mb4_unicode_520_ci --> utf8 and utf8_general_ci
ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;

# Save and run msql again

#### reset httpd and enjoy
service httpd restart

#############################################################################################

############################ WEBRTC DELOYMENT SERVER ############################

# 1 Clone project 
git clone https://github.com/webrtc/apprtc.git

# 2 Do all the steps in the Collider instructions then continue on step 3.

## Collider: signal server in Go language 

# 2.1 clone (done in #1)
# 2.1.1 download go language and install
    # download go files: https://golang.org/dl/ and unzip into /root/uav-project/
    # ATENTION: 32bit and 64bit version: https://dl.google.com/go/go1.12.linux-386.tar.gz
    export GOROOT=/root/uav-project/go

    [root@ctr486333 go]# pwd
    /root/uav-project/go
    [root@ctr486333 go]# tree -L 1
    .
    ├── api
    ├── AUTHORS
    ├── bin
    ├── CONTRIBUTING.md
    ├── CONTRIBUTORS
    ├── doc
    ├── favicon.ico
    ├── lib
    ├── LICENSE
    ├── misc
    ├── PATENTS
    ├── pkg
    ├── README.md
    ├── robots.txt
    ├── src
    ├── test
    └── VERSION
# 2.2 add $GOPATH to bashrc with path of /root/uav-project/apprtc/src/collider
mkdir -p /root/uav-project/goWorkspace/src
vi ~/.bashrc

    export GOROOT=/root/uav-project/go
    export GOPATH=/root/uav-project/goWorkspace
    export GOBIN=$GOROOT/bin
    export PATH=$PATH:$GOBIN:$GOPATH


tree ./

    [root@ctr486333 collider]# tree ./
    ./
    ├── collider
    │   ├── client.go
    │   ├── client_test.go
    │   ├── collider.go
    │   ├── collider_test.go
    │   ├── dashboard.go
    │   ├── dashboard_test.go
    │   ├── messages.go
    │   ├── room.go
    │   ├── roomTable.go
    │   └── room_test.go
    ├── collidermain
    │   └── main.go
    ├── collidertest
    │   └── mockrwc.go
    └── README.md

# test :
[root@ctr486333 uav-project]# go
    Go is a tool for managing Go source code.

    Usage:

        go <command> [arguments]

    The commands are:
    ...


# 2.3 mkdir collinder dir and link the collider directories into $GOPATH/src 
cd $GOROOT/src/cmd/vendor/golang.org/x
git clone https://github.com/gorilla/websocket
cd $GOPATH
ln -s /root/uav-project/apprtc/src/collider/src/collider  $GOPATH/src
ln -s /root/uav-project/apprtc/src/collider/src/collidermain $GOPATH/src
ln -s /root/uav-project/apprtc/src/collider/src/collidertest $GOPATH/src

# NOTICE: collidermain, collider, collidertest link must be inside /root/uav-project/go/src/ folder

# 2.4 get depedencies, install collidermain
cd $GOPATH/src
# go get -v golang.org/x/net/websocket
go get collidermain
go install collidermain
# 2.5 Running

$GOPATH/bin/collidermain -port=8089 -tls=true

# DEPLOY collinder

mkdir /cert
cat server.crt server.key > cert.pem
# server.key file is start with: -----BEGIN RSA PRIVATE KEY
cat server.key > key.pem
# copy cert.pem, key.pem to /cert/

# Create service for collinder
vi $GOPATH/start.sh

#!/bin/sh
/root/uav-project/goWorkspace/bin/collidermain 2>> /root/uav-project/goWorkspace/collider.log

chmod 744 start.sh

vi /etc/inittab

# add following line:
coll:2:respawn:/root/uav-project/goWorkspace/start.sh
# run to apply the inittab change without rebooting:
init q

# edit port in source code collidermain/main.go and rebuild:
cd collidermain
go build
mv collidermain $GOPATH/bin

# run for this session without reboot
nohup ./start.sh &

# cheking out Collinder ran:
#  1. Using netstat to see the listening processes
    netstat -tulnp
#  2. Using ss to see the listening processes
    ss -nutlp

############## DONE COLLINDER signaling server ################

#############  Coturn Server ##################################

https://github.com/coturn/coturn/wiki/CoturnConfig
# 1.1 Install libs

$ wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
$ tar xvfz libevent-2.0.21-stable.tar.gz
$ cd libevent-2.0.21-stable
$ ./configure
$ make
$ make install

# 1.2 Install Coturn server (Turnserver)

# download at: https://github.com/coturn/coturn/wiki/Downloads
# https://coturn.net/turnserver/v4.5.0.8/
$ yum install openssl-devel
$ tar xvfz turnserver-4.5.0.8.tar.gz
$ ./configure
$ make
$ make install

####
1) If your system supports automatic start-up system daemon services, 
then to enable the turnserver as a system service that is automatically
started, you have to:

	a) Create and edit /etc/turnserver.conf or 
	/usr/local/etc/turnserver.conf . 
	Use /usr/local/etc/turnserver.conf.default as an example.

	b) For user accounts settings: set up SQLite or PostgreSQL or 
	MySQL or MongoDB or Redis database for user accounts.
	Use /usr/local/share/turnserver/schema.sql as SQL database schema,
	or use /usr/local/share/turnserver/schema.userdb.redis as Redis
	database schema description and/or 
	/usr/local/share/turnserver/schema.stats.redis
	as Redis status & statistics database schema description.
	
	If you are using SQLite, the default database location is in 
	/var/db/turndb or in /usr/local/var/db/turndb or in /var/lib/turn/turndb.
	 
	c) add whatever is necessary to enable start-up daemon for the 
	/usr/local/bin/turnserver.
     
2) If you do not want the turnserver to be a system service, 
   then you can start/stop it "manually", using the "turnserver" 
   executable with appropriate options (see the documentation).
   
3) To create database schema, use schema in file 
/usr/local/share/turnserver/schema.sql.
   
4) For additional information, run:
 
   $ man turnserver
   $ man turnadmin
   $ man turnutils
	
# 1.1 config
cat /usr/local/etc/turnserver.conf.default  > /etc/turnserver.conf
vi !$



# Coturn TURN SERVER configuration file
#
# Boolean values note: where boolean value is supposed to be used,
# you can use '0', 'off', 'no', 'false', 'f' as 'false, 
# and you can use '1', 'on', 'yes', 'true', 't' as 'true' 
# If the value is missed, then it means 'true'.
#

# Listener interface device (optional, Linux only).
# NOT RECOMMENDED. 
#
listening-device=eth0

# TURN listener port for UDP and TCP (Default: 3478).
# Note: actually, TLS & DTLS sessions can connect to the 
# "plain" TCP & UDP port(s), too - if allowed by configuration.
#
listening-port=3478

# TURN listener port for TLS (Default: 5349).
# Note: actually, "plain" TCP & UDP sessions can connect to the TLS & DTLS
# port(s), too - if allowed by configuration. The TURN server 
# "automatically" recognizes the type of traffic. Actually, two listening
# endpoints (the "plain" one and the "tls" one) are equivalent in terms of
# functionality; but we keep both endpoints to satisfy the RFC 5766 specs.
# For secure TCP connections, we currently support SSL version 3 and 
# TLS version 1.0, 1.1 and 1.2.
# For secure UDP connections, we support DTLS version 1.
#
tls-listening-port=5349

# Alternative listening port for UDP and TCP listeners;
# default (or zero) value means "listening port plus one". 
# This is needed for RFC 5780 support
# (STUN extension specs, NAT behavior discovery). The TURN Server 
# supports RFC 5780 only if it is started with more than one 
# listening IP address of the same family (IPv4 or IPv6).
# RFC 5780 is supported only by UDP protocol, other protocols
# are listening to that endpoint only for "symmetry".
#
#alt-listening-port=0
							 
# Alternative listening port for TLS and DTLS protocols.
# Default (or zero) value means "TLS listening port plus one".
#
#alt-tls-listening-port=0
	
# Listener IP address of relay server. Multiple listeners can be specified.
# If no IP(s) specified in the config file or in the command line options, 
# then all IPv4 and IPv6 system IPs will be used for listening.
#
#listening-ip=172.17.19.101
#listening-ip=10.207.21.238
#listening-ip=2607:f0d0:1002:51::4

# Auxiliary STUN/TURN server listening endpoint.
# Aux servers have almost full TURN and STUN functionality.
# The (minor) limitations are:
#
# 1) Auxiliary servers do not have alternative ports and
# they do not support STUN RFC 5780 functionality (CHANGE REQUEST).
#
# 2) Auxiliary servers also are never returning ALTERNATIVE-SERVER reply.
# 
# Valid formats are 1.2.3.4:5555 for IPv4 and [1:2::3:4]:5555 for IPv6.
#
# There may be multiple aux-server options, each will be used for listening
# to client requests.
#
#aux-server=172.17.19.110:33478
#aux-server=[2607:f0d0:1002:51::4]:33478

# (recommended for older Linuxes only)
# Automatically balance UDP traffic over auxiliary servers (if configured).
# The load balancing is using the ALTERNATE-SERVER mechanism.
# The TURN client must support 300 ALTERNATE-SERVER response for this 
# functionality.
#
#udp-self-balance

# Relay interface device for relay sockets (optional, Linux only).
# NOT RECOMMENDED.
#
#relay-device=eth1

# Relay address (the local IP address that will be used to relay the 
# packets to the peer).
# Multiple relay addresses may be used.
# The same IP(s) can be used as both listening IP(s) and relay IP(s).
#
# If no relay IP(s) specified, then the turnserver will apply the default
# policy: it will decide itself which relay addresses to be used, and it 
# will always be using the client socket IP address as the relay IP address
# of the TURN session (if the requested relay address family is the same
# as the family of the client socket).
#
#relay-ip=172.17.19.105
#relay-ip=2607:f0d0:1002:51::5

# For Amazon EC2 users:
#
# TURN Server public/private address mapping, if the server is behind NAT.
# In that situation, if a -X is used in form "-X <ip>" then that ip will be reported
# as relay IP address of all allocations. This scenario works only in a simple case
# when one single relay address is be used, and no RFC5780 functionality is required.
# That single relay address must be mapped by NAT to the 'external' IP.
# The "external-ip" value, if not empty, is returned in XOR-RELAYED-ADDRESS field.
# For that 'external' IP, NAT must forward ports directly (relayed port 12345
# must be always mapped to the same 'external' port 12345).
#
# In more complex case when more than one IP address is involved,
# that option must be used several times, each entry must
# have form "-X <public-ip/private-ip>", to map all involved addresses.
# RFC5780 NAT discovery STUN functionality will work correctly,
# if the addresses are mapped properly, even when the TURN server itself 
# is behind A NAT.
#
# By default, this value is empty, and no address mapping is used.
#
#external-ip=60.70.80.91
#
#OR:
#
#external-ip=60.70.80.91/172.17.19.101
#external-ip=60.70.80.92/172.17.19.102


# Number of the relay threads to handle the established connections
# (in addition to authentication thread and the listener thread).
# If explicitly set to 0 then application runs relay process in a 
# single thread, in the same thread with the listener process 
# (the authentication thread will still be a separate thread).
#
# If this parameter is not set, then the default OS-dependent 
# thread pattern algorithm will be employed. Usually the default
# algorithm is the most optimal, so you have to change this option
# only if you want to make some fine tweaks. 
#
# In the older systems (Linux kernel before 3.9),
# the number of UDP threads is always one thread per network listening
# endpoint - including the auxiliary endpoints - unless 0 (zero) or 
# 1 (one) value is set.
#
#relay-threads=0

# Lower and upper bounds of the UDP relay endpoints:
# (default values are 49152 and 65535)
#
#min-port=49152
#max-port=65535
	
# Uncomment to run TURN server in 'normal' 'moderate' verbose mode.
# By default the verbose mode is off.
verbose
	
# Uncomment to run TURN server in 'extra' verbose mode.
# This mode is very annoying and produces lots of output.
# Not recommended under any normal circumstances.
#	
#Verbose

# Uncomment to use fingerprints in the TURN messages.
# By default the fingerprints are off.
#
#fingerprint

# Uncomment to use long-term credential mechanism.
# By default no credentials mechanism is used (any user allowed).
#
#lt-cred-mech

# This option is opposite to lt-cred-mech. 
# (TURN Server with no-auth option allows anonymous access).
# If neither option is defined, and no users are defined,
# then no-auth is default. If at least one user is defined, 
# in this file or in command line or in usersdb file, then
# lt-cred-mech is default.
#
#no-auth

# TURN REST API flag.
# (Time Limited Long Term Credential)
# Flag that sets a special authorization option that is based upon authentication secret.
#
# This feature's purpose is to support "TURN Server REST API", see
# "TURN REST API" link in the project's page 
# https://github.com/coturn/coturn/
#
# This option is used with timestamp:
# 
# usercombo -> "timestamp:userid"
# turn user -> usercombo
# turn password -> base64(hmac(secret key, usercombo))
#
# This allows TURN credentials to be accounted for a specific user id.
# If you don't have a suitable id, the timestamp alone can be used.
# This option is just turning on secret-based authentication.
# The actual value of the secret is defined either by option static-auth-secret,
# or can be found in the turn_secret table in the database (see below).
# 
# Read more about it:
#  - https://tools.ietf.org/html/draft-uberti-behave-turn-rest-00
#  - https://www.ietf.org/proceedings/87/slides/slides-87-behave-10.pdf
#
# Be aware that use-auth-secret overrides some part of lt-cred-mech.
# Notice that this feature depends internally on lt-cred-mech, so if you set
# use-auth-secret then it enables internally automatically lt-cred-mech option
# like if you enable both.
#
# You can use only one of the to auth mechanisms in the same time because,
# both mechanism use the username and password validation in different way.
#
# This way be aware that you can't use both auth mechnaism in the same time!
# Use in config either the lt-cred-mech or the use-auth-secret
# to avoid any confusion.
#
#use-auth-secret

# 'Static' authentication secret value (a string) for TURN REST API only. 
# If not set, then the turn server
# will try to use the 'dynamic' value in turn_secret table
# in user database (if present). The database-stored  value can be changed on-the-fly
# by a separate program, so this is why that other mode is 'dynamic'.
#
#static-auth-secret=north

# Server name used for
# the oAuth authentication purposes.
# The default value is the realm name.
#
#server-name=blackdow.carleon.gov

# Flag that allows oAuth authentication.
#
#oauth

# 'Static' user accounts for long term credentials mechanism, only.
# This option cannot be used with TURN REST API.
# 'Static' user accounts are NOT dynamically checked by the turnserver process, 
# so that they can NOT be changed while the turnserver is running.
#
#user=username1:key1
#user=username2:key2
# OR:
user=nobjta_9x_tq:canhthong
#user=username2:password2
#
# Keys must be generated by turnadmin utility. The key value depends
# on user name, realm, and password:
#
# Example:
# $ turnadmin -k -u ninefingers -r north.gov -p youhavetoberealistic
# Output: 0xbc807ee29df3c9ffa736523fb2c4e8ee
# ('0x' in the beginning of the key is what differentiates the key from
# password. If it has 0x then it is a key, otherwise it is a password).
#
# The corresponding user account entry in the config file will be:
# 
#user=ninefingers:0xbc807ee29df3c9ffa736523fb2c4e8ee
# Or, equivalently, with open clear password (less secure):
#user=ninefingers:youhavetoberealistic
#

# SQLite database file name.
#
# Default file name is /var/db/turndb or /usr/local/var/db/turndb or
# /var/lib/turn/turndb.
# 
#userdb=/var/db/turndb

# PostgreSQL database connection string in the case that we are using PostgreSQL
# as the user database.
# This database can be used for long-term credential mechanism
# and it can store the secret value for secret-based timed authentication in TURN RESP API. 
# See http://www.postgresql.org/docs/8.4/static/libpq-connect.html for 8.x PostgreSQL
# versions connection string format, see 
# http://www.postgresql.org/docs/9.2/static/libpq-connect.html#LIBPQ-CONNSTRING
# for 9.x and newer connection string formats.
#
#psql-userdb="host=<host> dbname=<database-name> user=<database-user> password=<database-user-password> connect_timeout=30"

# MySQL database connection string in the case that we are using MySQL
# as the user database.
# This database can be used for long-term credential mechanism
# and it can store the secret value for secret-based timed authentication in TURN RESP API.
#
# Optional connection string parameters for the secure communications (SSL): 
# ca, capath, cert, key, cipher 
# (see http://dev.mysql.com/doc/refman/5.1/en/ssl-options.html for the 
# command options description).
#
# Use string format as below (space separated parameters, all optional):
#
mysql-userdb="host=localhost dbname=coturn user=root password=Canhth0ng port=3306 connect_timeout=20 read_timeout=10"

# If you want to use in the MySQL connection string the password in encrypted format,

# then set in this option the MySQL password encryption secret key file.
#
# Warning: If this option is set, then mysql password must be set in "mysql-userdb" in encrypted format! 
# If you want to use cleartext password then do not set this option!
#
# This is the file path which contain secret key of aes encryption while using password encryption.
#
secret-key-file=/root/coturn.key

# MongoDB database connection string in the case that we are using MongoDB
# as the user database.
# This database can be used for long-term credential mechanism
# and it can store the secret value for secret-based timed authentication in TURN RESP API. 
# Use string format is described at http://hergert.me/docs/mongo-c-driver/mongoc_uri.html
#
#mongo-userdb="mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]"

# Redis database connection string in the case that we are using Redis
# as the user database.
# This database can be used for long-term credential mechanism
# and it can store the secret value for secret-based timed authentication in TURN RESP API. 
# Use string format as below (space separated parameters, all optional):
#
#redis-userdb="ip=<ip-address> dbname=<database-number> password=<database-user-password> port=<port> connect_timeout=<seconds>"

# Redis status and statistics database connection string, if used (default - empty, no Redis stats DB used).
# This database keeps allocations status information, and it can be also used for publishing
# and delivering traffic and allocation event notifications.
# The connection string has the same parameters as redis-userdb connection string. 
# Use string format as below (space separated parameters, all optional):
#
#redis-statsdb="ip=<ip-address> dbname=<database-number> password=<database-user-password> port=<port> connect_timeout=<seconds>"

# The default realm to be used for the users when no explicit 
# origin/realm relationship was found in the database, or if the TURN
# server is not using any database (just the commands-line settings
# and the userdb file). Must be used with long-term credentials 
# mechanism or with TURN REST API.
#
# Note: If default realm is not specified at all, then realm falls back to the host domain name.
#       If domain name is empty string, or '(None)', then it is initialized to am empty string.
#
#realm=mycompany.org

# The flag that sets the origin consistency 
# check: across the session, all requests must have the same
# main ORIGIN attribute value (if the ORIGIN was
# initially used by the session).
#
#check-origin-consistency

# Per-user allocation quota.
# default value is 0 (no quota, unlimited number of sessions per user).
# This option can also be set through the database, for a particular realm.
#
#user-quota=0

# Total allocation quota.
# default value is 0 (no quota).
# This option can also be set through the database, for a particular realm.
#
#total-quota=0

# Max bytes-per-second bandwidth a TURN session is allowed to handle
# (input and output network streams are treated separately). Anything above
# that limit will be dropped or temporary suppressed (within
# the available buffer limits).
# This option can also be set through the database, for a particular realm.
#
#max-bps=0

#
# Maximum server capacity.
# Total bytes-per-second bandwidth the TURN server is allowed to allocate
# for the sessions, combined (input and output network streams are treated separately).
#
# bps-capacity=0

# Uncomment if no UDP client listener is desired.
# By default UDP client listener is always started.
#
#no-udp

# Uncomment if no TCP client listener is desired.
# By default TCP client listener is always started.
#
#no-tcp

# Uncomment if no TLS client listener is desired.
# By default TLS client listener is always started.
#
#no-tls

# Uncomment if no DTLS client listener is desired.
# By default DTLS client listener is always started.
#
#no-dtls

# Uncomment if no UDP relay endpoints are allowed.
# By default UDP relay endpoints are enabled (like in RFC 5766).
#
#no-udp-relay

# Uncomment if no TCP relay endpoints are allowed.
# By default TCP relay endpoints are enabled (like in RFC 6062).
#
#no-tcp-relay

# Uncomment if extra security is desired,
# with nonce value having limited lifetime.
# By default, the nonce value is unique for a session,
# and has unlimited lifetime. 
# Set this option to limit the nonce lifetime. 
# It defaults to 600 secs (10 min) if no value is provided. After that delay, 
# the client will get 438 error and will have to re-authenticate itself.
#
#stale-nonce=600

# Uncomment if you want to set the maximum allocation
# time before it has to be refreshed.
# Default is 3600s.
#
#max-allocate-lifetime=3600


# Uncomment to set the lifetime for the channel.
# Default value is 600 secs (10 minutes).
# This value MUST not be changed for production purposes.
#
#channel-lifetime=600

# Uncomment to set the permission lifetime.
# Default to 300 secs (5 minutes).
# In production this value MUST not be changed,
# however it can be useful for test purposes.
#
#permission-lifetime=300

# Certificate file.
# Use an absolute path or path relative to the 
# configuration file.
#
cert=/root/cert.pem

# Private key file.
# Use an absolute path or path relative to the 
# configuration file.
# Use PEM file format.
#
pkey=/root/key.pem

# Private key file password, if it is in encoded format.
# This option has no default value.
#
#pkey-pwd=...

# Allowed OpenSSL cipher list for TLS/DTLS connections.
# Default value is "DEFAULT".
#
#cipher-list="DEFAULT"

# CA file in OpenSSL format. 
# Forces TURN server to verify the client SSL certificates.
# By default it is not set: there is no default value and the client
# certificate is not checked.
#
# Example:
#CA-file=/etc/ssh/id_rsa.cert

# Curve name for EC ciphers, if supported by OpenSSL 
# library (TLS and DTLS). The default value is prime256v1, 
# if pre-OpenSSL 1.0.2 is used. With OpenSSL 1.0.2+,
# an optimal curve will be automatically calculated, if not defined
# by this option.
#
#ec-curve-name=prime256v1

# Use 566 bits predefined DH TLS key. Default size of the key is 1066.
#
#dh566

# Use 2066 bits predefined DH TLS key. Default size of the key is 1066.
#
#dh2066

# Use custom DH TLS key, stored in PEM format in the file.
# Flags --dh566 and --dh2066 are ignored when the DH key is taken from a file.
#
#dh-file=<DH-PEM-file-name>

# Flag to prevent stdout log messages.
# By default, all log messages are going to both stdout and to 
# the configured log file. With this option everything will be 
# going to the configured log only (unless the log file itself is stdout).
#
#no-stdout-log

# Option to set the log file name.
# By default, the turnserver tries to open a log file in 
# /var/log, /var/tmp, /tmp and current directories directories
# (which open operation succeeds first that file will be used).
# With this option you can set the definite log file name.
# The special names are "stdout" and "-" - they will force everything 
# to the stdout. Also, the "syslog" name will force everything to
# the system log (syslog). 
# In the runtime, the logfile can be reset with the SIGHUP signal 
# to the turnserver process.
#
#log-file=/var/tmp/turn.log

# Option to redirect all log output into system log (syslog).
#
#syslog

# This flag means that no log file rollover will be used, and the log file
# name will be constructed as-is, without PID and date appendage.
# This option can be used, for example, together with the logrotate tool.
#
#simple-log

# Option to set the "redirection" mode. The value of this option
# will be the address of the alternate server for UDP & TCP service in form of 
# <ip>[:<port>]. The server will send this value in the attribute
# ALTERNATE-SERVER, with error 300, on ALLOCATE request, to the client.
# Client will receive only values with the same address family
# as the client network endpoint address family. 
# See RFC 5389 and RFC 5766 for ALTERNATE-SERVER functionality description. 
# The client must use the obtained value for subsequent TURN communications.
# If more than one --alternate-server options are provided, then the functionality
# can be more accurately described as "load-balancing" than a mere "redirection". 
# If the port number is omitted, then the default port 
# number 3478 for the UDP/TCP protocols will be used.
# Colon (:) characters in IPv6 addresses may conflict with the syntax of 
# the option. To alleviate this conflict, literal IPv6 addresses are enclosed 
# in square brackets in such resource identifiers, for example: 
# [2001:db8:85a3:8d3:1319:8a2e:370:7348]:3478 . 
# Multiple alternate servers can be set. They will be used in the
# round-robin manner. All servers in the pool are considered of equal weight and 
# the load will be distributed equally. For example, if we have 4 alternate servers, 
# then each server will receive 25% of ALLOCATE requests. A alternate TURN server 
# address can be used more than one time with the alternate-server option, so this 
# can emulate "weighting" of the servers.
#
# Examples: 
#alternate-server=1.2.3.4:5678
#alternate-server=11.22.33.44:56789
#alternate-server=5.6.7.8
#alternate-server=[2001:db8:85a3:8d3:1319:8a2e:370:7348]:3478
			
# Option to set alternative server for TLS & DTLS services in form of 
# <ip>:<port>. If the port number is omitted, then the default port 
# number 5349 for the TLS/DTLS protocols will be used. See the previous 
# option for the functionality description.
#
# Examples: 
#tls-alternate-server=1.2.3.4:5678
#tls-alternate-server=11.22.33.44:56789
#tls-alternate-server=[2001:db8:85a3:8d3:1319:8a2e:370:7348]:3478

# Option to suppress TURN functionality, only STUN requests will be processed.
# Run as STUN server only, all TURN requests will be ignored.
# By default, this option is NOT set.
#
#stun-only

# Option to suppress STUN functionality, only TURN requests will be processed.
# Run as TURN server only, all STUN requests will be ignored.
# By default, this option is NOT set.
#
#no-stun

# This is the timestamp/username separator symbol (character) in TURN REST API.
# The default value is ':'.
# rest-api-separator=:	

# Flag that can be used to disallow peers on the loopback addresses (127.x.x.x and ::1).
# This is an extra security measure.
#
#no-loopback-peers

# Flag that can be used to disallow peers on well-known broadcast addresses (224.0.0.0 and above, and FFXX:*).
# This is an extra security measure.
#
#no-multicast-peers

# Option to set the max time, in seconds, allowed for full allocation establishment. 
# Default is 60 seconds.
#
#max-allocate-timeout=60

# Option to allow or ban specific ip addresses or ranges of ip addresses. 
# If an ip address is specified as both allowed and denied, then the ip address is 
# considered to be allowed. This is useful when you wish to ban a range of ip 
# addresses, except for a few specific ips within that range.
#
# This can be used when you do not want users of the turn server to be able to access
# machines reachable by the turn server, but would otherwise be unreachable from the 
# internet (e.g. when the turn server is sitting behind a NAT)
#
# Examples:
# denied-peer-ip=83.166.64.0-83.166.95.255
# allowed-peer-ip=83.166.68.45

# File name to store the pid of the process.
# Default is /var/run/turnserver.pid (if superuser account is used) or
# /var/tmp/turnserver.pid .
#
#pidfile="/var/run/turnserver.pid"

# Require authentication of the STUN Binding request.
# By default, the clients are allowed anonymous access to the STUN Binding functionality.
#
#secure-stun

# Mobility with ICE (MICE) specs support.
#
mobility

# Allocate Address Family according 
# If enabled then TURN server allocates address family according  the TURN 
# Client <=> Server communication address family.
# (By default coTURN works according RFC 6156.)
# !!Warning: Enabling this option breaks RFC6156 section-4.2 (violates use default IPv4)!!
#
#keep-address-family


# User name to run the process. After the initialization, the turnserver process
# will make an attempt to change the current user ID to that user.
#
#proc-user=<user-name>

# Group name to run the process. After the initialization, the turnserver process
# will make an attempt to change the current group ID to that group.
#
#proc-group=<group-name>

# Turn OFF the CLI support.
# By default it is always ON.
# See also options cli-ip and cli-port.
#
#no-cli

#Local system IP address to be used for CLI server endpoint. Default value
# is 127.0.0.1.
#
#cli-ip=127.0.0.1

# CLI server port. Default is 5766.
#
#cli-port=5766

# CLI access password. Default is empty (no password).
# For the security reasons, it is recommended to use the encrypted
# for of the password (see the -P command in the turnadmin utility).
#
# Secure form for password 'qwerty':
#
#cli-password=$5$79a316b350311570$81df9cfb9af7f5e5a76eada31e7097b663a0670f99a3c07ded3f1c8e59c5658a
#
# Or unsecure form for the same password:
#
#cli-password=qwerty

# Server relay. NON-STANDARD AND DANGEROUS OPTION. 
# Only for those applications when we want to run 
# server applications on the relay endpoints.
# This option eliminates the IP permissions check on 
# the packets incoming to the relay endpoints.
#
#server-relay

# Maximum number of output sessions in ps CLI command.
# This value can be changed on-the-fly in CLI. The default value is 256.
#
#cli-max-output-sessions

# Set network engine type for the process (for internal purposes).
#
#ne=[1|2|3]

# Do not allow an TLS/DTLS version of protocol
#
#no-tlsv1
#no-tlsv1_1
#no-tlsv1_2




# 1.3 Create database name: coturn

# 1.4 ????



#### APP RTC ####################################################

# Open src/app_engine/constants.py and do the following:

#1.1 Collider
$ vi app_engine/constants.py 
    WSS_INSTANCE_HOST_KEY = 'localhost:8443'
    WSS_INSTANCE_NAME_KEY = 'uav_sercret'

# save and run build: 

# install npm
# Enable Fedora Extra Packages for Enterprise Linux (EPEL) repos
$ rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

# Install npm from the epel repos
$ sudo yum install npm --enablerepo=epel
# install g++ 7.1.0
Install gcc compiler
First thing to to is to use terminal to install the most updated compiler. It will be used to compile the new compiler. Yes. gcc is compiled using its gcc compiler.
sudo yum install gcc-c++

Install glibc
sudo yum install -y gcc texinfo-tex flex zip libgcc.i686 glibc-devel.i686

Download gcc source code
wget ftp://ftp.gnu.org/gnu/gcc/gcc-7.1.0/gcc-7.1.0.tar.gz

Download mpc, mpfr, gmp package
tar zxf gcc-7.1.0.tar.gz
cd gcc-7.1.0
./contrib/download_prerequisites

Compile gcc
mkdir gcc-build-7.1.0
cd gcc-build-7.1.0
../configure --prefix=/usr
sudo make && make install

Check your gcc versions
gcc --version
gcc (GCC) 7.1.0
g++ --version
g++ (GCC) 7.1.0
which gcc
/usr/bin/gcc
which g++
/usr/bin/g++

#https://stackoverflow.com/questions/9253695/building-gcc-requires-gmp-4-2-mpfr-2-3-1-and-mpc-0-8-0
# in root folder: /root/uav-project/apprtc
$ yum install gcc openssl-devel bzip2-devel
$ wget https://www.python.org/ftp/python/2.7.15/Python-2.7.15.tgz
$ npm install

## IF google-cloud-sdk cannot install, install it by PIP python 2.7
## IF APPRTC CANNOT BUILD BY GRUNT, COPY BUILDED VERSION FROM OTHER COMPUTER LIKE UBUNTU 18.04

## thôi đến đây tự mò tiếp đi, đọc hướng dẫn ấy, tốt nhất là cài UBUNTU BẢN MỚI NHẤT ĐỂ  DỄ  BUILD 
## DÙNG STUNNEL https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ssl-tunnel-using-stunnel-on-ubuntu
# để khiến apprtc chạy https bằng cách cho nó chạy ở port 8080 rồi redirect sang 443

# Run apprtc
nohup /root/uav-project/gooogleSDK/google-cloud-sdk/bin/dev_appserver.py --host admasterlife.com --port 8080 /root/apprtc/out/app_engine/ &

yum -y install stunnel

Create a new file in /etc/stunnel/stunnel.conf with the following contents, edited as needed for your requirements:

client = no
[apprtc]
accept = 443
sslVersion = TLSv1.1
connect = admasterlife.com:8080
cert = /etc/stunnel/cert.pem

/usr/bin/stunnel5 /etc/stunnel/stunnel.conf 

# checking stunnel
netstat -lntu
[root@ctr486333 ~]# net
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      
tcp        0      0 103.101.163.159:8080        0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      
tcp        0      0 127.0.0.1:37176             0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:443                 0.0.0.0:*                   LISTEN      
tcp        0      0 127.0.0.1:8000              0.0.0.0:*                   LISTEN      
tcp        0      0 :::22                       :::*                        LISTEN      
udp        0      0 0.0.0.0:68                  0.0.0.0:*                               
                       
# truy cập https://admasterlife.com
                       


My web browser cannot talk to stunnel
If you get the following error message in stunnel:
2003.01.18 17:46:07 LOG3[6093:32770]: SSL_accept: 1407609C: error:1407609C:SSL routines:SSL23_GET_CLIENT_HELLO:http request

then your stunnel runs in server mode (without client = yes) and your web browser is connecting to it as if it is a normal webserver, ala http://example.com/.

If stunnel is supposed to be running as a client, then fix your stunnel.conf. If you do mean to be running as an SSL server then point your browser at https://host:port/ instead of http://host:port/.

https://www.stunnel.org/faq.html



# https://github.com/webrtc/apprtc/issues/615
# Ha ha, I found the error when connecting via MOBILE NETWORK during two years ago to config apprtc:
# Just config ICE servers like this:

    ICE_SERVER_OVERRIDE = [
        {
            "urls": "turn:103.101.163.159:3478?transport=udp",
            "username": "nobjta_9x_tq",
            "credential": "canhthong"
        },
        {
            "urls": "turn:103.101.163.159:3478?transport=tcp",
            "username": "nobjta_9x_tq",
            "credential": "canhthong"
        }
    ]

    ICE_SERVER_BASE_URL = ''
    ICE_SERVER_URL_TEMPLATE = ''
    ICE_SERVER_API_KEY = ''

# and in /etc/turnserver.conf

    cert=/root/cert.pem
    pkey=/root/key.pem

    listening-port=3478
    tls-listening-port=5349

    listening-ip=103.101.163.159

    relay-ip=103.101.163.159
    external-ip=103.101.163.159

    realm=admasterlife.com
    server-name=admasterlife.com

    #lt-cred-mech
    userdb=/etc/turnuserdb.conf
    oauth
    user=my_account:my_password
    no-stdout-log

# Reson of error is: when I config "lt-cred-mech" authentication, It was failed.
# So, I change it to "oauth": It Worked.

# test Turn Server (Collinder) in this website:
https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/

# The result very fast (like sturn url of Google):
0.005	1	host	3868393361	udp	192.168.1.157	35353	126 | 30 | 255
0.006	1	host	891932622	udp	2405:4800:12c7:190f:247e:5443:3c18:885f	51606	126 | 40 | 255
0.009	1	srflx	842163049	udp	42.114.30.95	3341	100 | 30 | 255
0.062	1	relay	3031532034	udp	103.101.163.159	62030	2 | 30 | 255
0.105	Done
0.109


#More infomation about Turn server Collinder error:
https://github.com/webrtc/apprtc/issues/266#issuecomment-235185590
https://www.webrtc-experiment.com/docs/TURN-server-installation-guide.html
# =>> 10: dùng oauth