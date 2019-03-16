
## Use Grafana to Setup a Dashboard

* * *

### Install Grafana on Ubuntu

```bash
wget https://dl.grafana.com/oss/release/grafana_6.0.1_amd64.deb 

sudo apt-get install -y adduser libfontconfig

sudo dpkg -i grafana_6.0.1_amd64.deb 
```

* * *

### Configure Grafana

Semicolons (the ; char) are the standard way to comment out lines in a .ini file.
Change the port number to 9530

```bash
vim /etc/grafana/grafana.ini

```
in grafana ini file, comment starts with ```#```, value comments with ```;```

```bash
# The http port  to use
http_port = 9530

# The full public facing url you use in browser, used for redirects and emails
# If you use reverse proxy and sub path specify full url (with sub path)
root_url = http://localhost:9530

```
save and restart grafana service

```
systemctl daemon-reload
systemctl start grafana-server
systemctl status grafana-server
systemctl restart grafana-server
```
check if grafana works: open browser to http://localhost:9530

[more about grafana configuration](http://docs.grafana.org/installation/configuration/)

* * *

### Set Grafana to HTTPS

1. generate certification file and key save to /etc/ssl/grafana-key.pem, /etc/ssl/grafana-cert.pem

    move to grafana configuration directory
```bash
cd /etc/grafana
```
    create a temporary self-signed certificate

```bash
openssl genrsa -out selfsigned-grafana.key 2048
openssl req -new -key selfsigned-grafana.key -out selfsigned-grafana.csr
openssl x509 -req -days 365 -in selfsigned-grafana.csr -signkey selfsigned-grafana.key -out selfsigned-grafana.crt
```

    Enter passphrase, and answers some stupid questions like 

    * Country Name
    * State
    * Locality
    * Orgnization 
    * Unit Name 

    change the certification and key file permission
```bash
chown grafana:grafana selfsigned-grafana.crt
chown grafana:grafana selfsigned-grafana.key
chmod 400 selfsigned-grafana.crt 
chmod 400 selfsigned-grafana.key
```

2. change grafana configuration file and restart service

```bash
vim /etc/grafana/grafana.ini
```

```shell
[server]
# Protocol (http, https, socket)
protocol = https

# The http port  to use
http_port = 9530

# enable gzip
enable_gzip = true

# https certs & key file
cert_file = "/etc/grafana/selfsigned-grafana.crt"
cert_key = "/etc/grafana/selfsigned-grafana.key"

# set to true if you host Grafana behind HTTPS. default is false.
cookie_secure = true

```

3. restart the grafana service 

```bash
systemctl status grafana-server
systemctl restart grafana-server
```

4. check the grafana logging
```bash
vim /var/log/grafana/grafana.log
```
5. verify https works, open browser and go to https://localhost:9530


* * *



### Install Influxdb


1. install influxdb

add apt repo from the [offical website](https://docs.influxdata.com/influxdb/v1.7/introduction/installation/)
```

wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

```

install from apt repo

```
sudo apt-get update && sudo apt-get install influxdb
sudo service influxdb start
```
everything looks fine 

```
Connected to http://localhost:8086 version 1.7.4
InfluxDB shell version: 1.7.4
Enter an InfluxQL query
> exit
```


2. Configure Influxdb

The configuration file for InfluxDB is ```influxdb.conf```, it has different location per different OS

* Linux: ```/etc/influxdb/influxdb.conf```

* MacOS: ```/usr/local/etc/influxdb.conf```

```bash
vim /etc/influxdb/influxdb.conf
```

enable https, and set the endpoint to https://localhost:8086

```
[http]

  [...]

  # Determines whether HTTPS is enabled.
  https-enabled = true

  [...]

  # The TLS or SSL certificate to use when HTTPS is enabled.
  https-certificate = "/etc/influxdb/selfsigned-influxdb.crt"

  # Use a separate private key location.
  https-private-key = "/etc/influxdb/selfsigned-influxdb.key"
```

generate certification file and key save to /etc/ssl/grafana-key.pem, /etc/ssl/grafana-cert.pem

    move to grafana configuration directory
```bash
cd /etc/influxdb
```
    create a temporary self-signed certificate

```bash
openssl genrsa -out selfsigned-influxdb.key 2048
openssl req -new -key selfsigned-influxdb.key -out selfsigned-influxdb.csr
openssl x509 -req -days 365 -in selfsigned-influxdb.csr -signkey selfsigned-influxdb.key -out selfsigned-influxdb.crt
```

    Enter passphrase, and answers some stupid questions like 

    * Country Name
    * State
    * Locality
    * Orgnization 
    * Unit Name 

    change the certification and key file permission
```bash
chown influxdb:influxdb selfsigned-influxdb.crt
chown influxdb:influxdb selfsigned-influxdb.key
chmod 400 selfsigned-influxdb.crt 
chmod 400 selfsigned-influxdb.key
```

check the influxdb 

```
sudo systemctl status influxdb

```

stop the influxdb 

```
sudo systemctl stop influxdb
```

resetart the infludb 

```
sudo systemctl restart influxdb
```

connect to https enabled influxdb, since we are using selfsigned certificate, make sure the ```unsafeSsl``` is used.

```
 influx  -ssl -unsafeSsl -host localhost

```

3. enable authentication

```bash

vim /etc/influxdb/influxdb.conf


```

```bash

[http]

# Determines whether user authentication is enabled over HTTP/HTTPS.
  auth-enabled = true
```

restart the influxdb

```
sudo systemctl stop influxdb
sudo systemctl restart influxdb
sudo systemctl status influxdb
```
4. create user for influxdb

```bash
influx  -ssl -unsafeSsl -host localhost
```
create admin, and a user with admin access

```bash
CREATE USER admin WITH PASSWORD '<password>' WITH ALL PRIVILEGES

```
log off and log in again
```
influx  -ssl -unsafeSsl -host localhost -username 'admin' -password '<password>'

```
create another user with admin access, and log in using the new user
```bash
CREATE USER caesar WITH PASSWORD '<password>' WITH ALL PRIVILEGES
exit
influx  -ssl -unsafeSsl -host localhost -username 'caesar' -password '<password>'
```
try new user's access, a dummy db is created and dropped.

```bash 
> SHOW  USERS

> SHOW DATABASES

> CREATE DATABASE dummy1

> SHOW DATABASES

name: databases
name
----
_internal
dummy1

> DROP DATABASE dummy1
> SHOW DATABASES
name: databases
name
----
_internal

> CREATE DATABASE dashboard01
```

### Update the Grafana Dashboard Configuration

- grafana dash board is defined in ```testnet-dashboard-stable.json```

- set Grafna API Token 

    create an orgnization in the dashboard
```bash

curl -k -X POST -H "Content-Type: application/json" -d '{"name":"dev-org"}' https://admin:<password>@localhost:9530/api/orgs

{"message":"Organization created","orgId":3}

```
an orgnization with id ``3`` is created

```
curl -k -X POST https://admin:<password>@localhost:9530/api/user/using/3

{"message":"Active organization changed"}

```

create the API token 

```
curl -k -X POST -H "Content-Type: application/json" -d '{"name":"apikeycurl", "role": "Admin"}' https://admin:zaq12wsx@localhost:9530/api/auth/keys

>{"name":"apikeycurl","key":"eyJrIjoiUWJmTk9hRkx2anVCekVRSWU3UGN3ZkhiM2kxV1I0bnEiLCJuIjoiYXBpa2V5Y3VybCIsImlkIjozfQ=="}

```

set the GRAFANA_API_TOKEN

```
$ export GRAFANA_API_TOKEN="eyJrIjoiUWJmTk9hRkx2anVCekVRSWU3UGN3ZkhiM2kxV1I0bnEiLCJuIjoiYXBpa2V5Y3VybCIsImlkIjozfQ=="
- set the configuration parameter 

```bash
export DASHBOARD_DB_CONFIG="host=localhost,db=dashboard01,u=caesar,p=<password>"
```
- run ```init.sh``` to initilize metrics database

- publish the dashboard

    * make sure you have the access to avoid ```git@github.com: Permission denied (publickey)``
```bash
./publish-metrics-dashboard.sh
```

### Install Rust
```bash
$ curl https://sh.rustup.rs -sSf | sh
$ source $HOME/.cargo/env
$ rustup component add rustfmt-preview
```
```bash
$ rustup update
```
```bash
$ sudo apt-get install libssl-dev pkg-config zlib1g-dev llvm clang
```
```bash
$ cargo build --all
```
Run the test suite:

```bash
$ cargo test --all
```

To emulate all the tests that will run on a Pull Request, run:

```bash
$ ./ci/run-local.sh
```

### malformed entry error

if erros like happened:
```bash
E: Malformed entry 63 in list file /etc/apt/sources.list (URI parse)
E: The list of sources could not be read.
E: Malformed entry 63 in list file /etc/apt/sources.list (URI parse)
E: The list of sources could not be read.
```
This error message told us the 63 line of the file ```/etc/apt/sources.list``` is invalid, so just commented out the line would be ok. 
use vim to open /etc/apt/sources.list, and comment out the 63 line, or any other number of line