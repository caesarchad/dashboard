
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
enable https, and set the endpoint to https://localhost:8086

3. create user for influxdb

4. create a database



### Update the Grafana Dashboard Configuration

1. grafana dash board is defined in ```testnet-dashboard-stable.json```

2. set the configuration parameter 

```bash
export DASHBOARD_DB_CONFIG="host=<metrics host>,db=<database name>,u=<username>,p=<password>"
```
3. run ```init.sh``` to initilize metrics database

4. publish the dashboard

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


### Solutions for error likes: git@github.com: Permission denied (publickey).

A "Permission denied" error means that the server rejected your connection. on your local machine, you haven't made any SSH keys.

Here's how to fix:

Open git bash (Use the Windows search. To find it, type "git bash") or the Mac Terminal. Pro Tip: You can use any *nix based command prompt (but not the default Windows Command Prompt!)
Type cd ~/.ssh. This will take you to the root directory for Git (Likely C:\Users\[YOUR-USER-NAME]\.ssh\ on Windows)
Within the .ssh folder, there should be these two files: id_rsa and id_rsa.pub. These are the files that tell your computer how to communicate with GitHub, BitBucket, or any other Git based service. Type ls to see a directory listing. If those two files don't show up, proceed to the next step. NOTE: Your SSH keys must be named id_rsa and id_rsa.pub in order for Git, GitHub, and BitBucket to recognize them by default.
To create the SSH keys, type ssh-keygen -t rsa -C "your_email@example.com". This will create both id_rsa and id_rsa.pub files.
Now, go and open id_rsa.pub in your favorite text editor (you can do this via Windows Explorer or the OSX Finder if you like, tpying open . will open the folder).
Copy the contents--exactly as it appears, with no extra spaces or lines--of id_rsa.pub and paste it into GitHub and/or BitBucket under the Account Settings > SSH Keys. NOTE: I like to give the SSH key a descriptive name, usually with the name of the workstation I'm on along with the date.
Now that you've added your public key to Github and/or BitBucket, try to git push again and see if it works. It should!
