# HW2 - Create GCP Instance from scratch and mongo v5.0 install:

1. [GCP] Create New Project:
My First Project --> New Project --> Select New Project

2. [GCP] Share Project:
Project Info block --> Add people to this project --> `Editor` role

3. [Local PC] Install compute engine api client to MacOS
(Refer to [article](https://tapendradev.medium.com/how-to-install-gcloud-sdk-on-the-macos-and-start-managing-gcp-through-cli-d14d2c3a8869)):

Download [archive](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-289.0.0-darwin-x86_64.tar.gz) --> 
```
./google-cloud-sdk/install.sh
```
--> Restart terminal or open the new tab for the changes to take effects.

4. [Local PC] Configuring the SDK:
```
gcloud init
```
Chose most cheapest region as default:
```
Do you want to configure a default Compute Region and Zone? (Y/n)?  Y
[11] us-west1-b
```

Configuretion will be stored to:
```
/Users/{USER}/.boto
```

5. [GCP] Create VM instance:
Compute --> Compute Engine --> VM Instances --> Enable `Compute Engine API` --> Create Instance --> Set deployment parameters:
- Name: `mongo-ek`
- Labels --> Add Labels --> Key: `instance_type` Value: `mongo_db`
--> Save
- Machine type: e2-medium (2 vCPU, 4GB memory)
-  `Boot disk` block --> Change "Debian" to "Ubuntu" --> Version: "Ubuntu 20.04 LTS"

 --> Create (or copy equivalent comandline and launch ot from terminal)

6. [GCP. Optional] Add ssh key to metadata:
Compute --> Compute Engine --> `Settings` block --> Metadata --> `SSH Keys` tab --> Add ssh_key.pub

(You can skip this step and add metadata from terminal later in the 8th step)

7. [Local PC] Remove dafault project:
```
gcloud projects list
PROJECT_ID           NAME                PROJECT_NUMBER
clear-shadow-333805  My First Project    912787810250
mongo2021-19890928   mongo2021-19890928  586552157981
```

```
gcloud projects delete clear-shadow-333805
```
8. [Local PC] Configure the security key for use by your VMs:

```
gcloud compute os-login ssh-keys add \
    --project PROJECT_ID \
    --key-file /Users/$USER/.ssh/id_ecdsa_sk.pub
```

9. [Local PC] List instances from local machine:
```
gcloud compute instances list
```

10. [Local PC] Login to the new instance through gcloud compute:
```
gcloud compute ssh mongo-ek --ssh-key-file=/Users/$USER/.ssh/id_ecdsa_sk
```

11. [Local PC. Optional] Configure gcloud compute ssh:
```
gcloud compute config-ssh --ssh-key-file=/Users/$USER/.ssh/id_ecdsa_sk
```
(all configurations will be stored to: `~/.ssh/config` file)

12. [Local PC] Login to the new instance through ssh:

```
ssh Instance_IP
```
In case when local user and remote user is not same, set remote user additionally:
```
ssh user@Instance_IP
```

13. [Remote GCP Instance] Install MongoDB ver 5.0 to Ubuntu 20.04 through official guidelines: [https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

- import the MongoDB public GPG Key
```
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
```
- Add mongo repository:
```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```
- Reload the local package database:
```
sudo apt-get update
```
- Install a specific release of MongoDB (freeze MongDB release version):
```
sudo apt-get install -y mongodb-org=5.0.2 mongodb-org-database=5.0.2 mongodb-org-server=5.0.2 mongodb-org-shell=5.0.2 mongodb-org-mongos=5.0.2 mongodb-org-tools=5.0.2
```

14. [Remote GCP Instance] Launch mongo:
- Create custom folders:
```
sudo mkdir /home/mongo && sudo mkdir /home/mongo/db1 && sudo chmod 777 /home/mongo/db1
```
- Launch mongo from terminal with listening all ip addresses (--bind_ip_all)
```
mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all --auth
```

15. [Remote GCP Instance] Create user:
```
mongo --port 27001
use admin;
db.createUser( { user: "mongo_db_admin", pwd: "mongo_db_admin_pwd", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
```

16. [Local PC] Install mongo client to local Mac OS for remote connection:
```
brew tap mongodb/brew
brew install mongodb-community-shell
```

17. [Local PC] Tag instance for applying Firewall rule to tags group:
```
gcloud compute instances add-tags mongo-ek --tags=mongo-db
```

18. [GCP] Open 27001 port on the mongo instance only for my IP:
Compute --> Compute Engine --> VM Instances --> Select `mongo-ek` instance --> More actions --> View network details --> Firewall (left menu) --> Create firewall rule --> 
- Name: `mongo-db-open-27001`
- Description: `Open 27001 for 109.86.165.62`
- Targets: `Specified target tags`
- Target tags: `mongo-db` (adding only 1 tag group added in the previous step)
- Source IPv4 ranges: `109.86.165.62/32`
- Specified protocols and ports: tcp:27001

19. [Local PC] Connect to mongo server remote:
```
mongo 34.82.82.252:27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin
```

20. [Remote GCP Instance] Stop current mongo process, wrap it to systemd process and relaunch:
-  Stop current mongo process:
```
sudo kill -9 $(pgrep mongod)
sudo rm -f /tmp/mongodb-*.sock
```
- Create new mongo config file:
```
sudo mv /etc/mongod.conf /etc/mongod.conf_origin
cat /etc/mongod.conf_origin |grep -v "#"
```
```
sudo su -
cat <<EOF >>/etc/mongod.conf
storage:
  dbPath: /home/mongo/db1
  journal:
    enabled: true

systemLog:
  destination: file
  logAppend: true
  path: /home/mongo/db1/db1.log

net:
  port: 27001
  bindIp: 0.0.0.0

processManagement:
  timeZoneInfo: /usr/share/zoneinfo
EOF
exit
```
- Change PID file path in the service:
```
sudo vi /lib/systemd/system/mongod.service
PIDFile=/var/run/mongodb/mongod.pid --> PIDFile=/home/mongo/db1/db1.pid
```
- Change mongo files owner from `kuts` to `mongo`:
```
sudo chown -R mongodb:mongodb /home/mongo/db1
```
- Start mongo through systemctl
```
sudo systemctl start mongod
```

21. [Local PC] Connect to mongo server remote:
```
mongo 34.82.82.252:27001 -u mongo_db_admin -p mongo_db_admin_pwd --authenticationDatabase admin
```

22. [Mongo] Create database:
```
use ek_test_db
```
For appearance database in the list we should fill database any data. Let's create 1st collection:
```
db.createCollection('1st_collection')
```
Show databases list:
```
show databases;
```

[Back to table of contents](../README.md)