---
title: siftester
layout: post
author: felixdu
title: Siftester
date: 2020-05-23T09:52:20.613Z
thumbnail: /assets/img/posts/hello.jpg
category: [siftester]
summary: Siftester
permalink: /blog/work/siftester
---
# Integrate SIFTESTER into NTAS


## Information

| Items               | xx                                             |
| ------------------- | ---------------------------------------------- |
| FQDN                | ntas-siftester.dynamic.nsn-net.net             |
| IP                  | 10.181.137.133                                 |
| postgresql username | siftester                                      |
| postgresql password | siftesterPass!                                 |
| redis password      | P5D7Jb4DwAdezkj9cxg9Z                          |
| jenkins password    | NTAS-SIFTESTER/NTASSIFTESTER123 or nsn-intra   |
| Jenkins URL         | http://ntas-siftester.dynamic.nsn-net.net:8080 |
| mongodb browser     | http://ntas-siftester.dynamic.nsn-net.net:8081 |
| postgres browser    | http://ntas-siftester.dynamic.nsn-net.net:8082 |

## Infos for every part
### codeEntitySwitcher

| Items | xx                                                                                        |
| ----- | ----------------------------------------------------------------------------------------- |
| Image | mncc-ntas-ft-docker-local.esisoj70.emea.nsn-net.net/ft/siftester-switcher:v1.1.7_20190814 |
| code  | https://scm.cci.nokia.net/ntas/ft/siftester/-/tree/master/codeEntitySwitcher              |




## Deploy Postgresql Server on Ubuntu 18.04
[Deploy Postgresql Server on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04)
### Step 1 — Installing PostgreSQL
```
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### Step 2 — Creating a New Role
```
createuser --interactive
Enter name of role to add: siftester
Shall the new role be a superuser? (y/n) y
```

### Step 3 — Creating a New Database
If you are logged in as the postgres account, you would type something like:
```
createdb siftester
```

### Step 4 - Opening a Postgres Prompt with the New Role
```
sudo adduser siftester
```

### Step 5 - [Configure PostgreSQL to allow remote connection](https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html)
* Configuring postgresql.conf
replace `/etc/postgresql/10/main/postgresql.conf.conf` from `listen_addresses = 'localhost'` to `listen_addresses = '*'`

* Configuring pg_hba.conf
```
host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5
```

### Step 6 - MAKE SURE THAT the user that is connecting has a password
a. Run the following psql command with the postgres user account:
```
sudo -u postgres psql postgres
```
b. Set the password:
```
# \password postgres
```

### restart postgres
`systemctl restart postgresql`


## Deploy Redis Server on Ubuntu 18.04
[how-to-install-and-secure-redis-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04)
```
sudo apt update
sudo apt install redis-server
```

sudo vi /etc/redis/redis.conf
```
/etc/redis/redis.conf
bind 127.0.0.1 ::1 --> bind 0.0.0.0 ::0
```

restart redis
```
sudo systemctl restart redis
```

Configuring a Redis Password
```
openssl rand 60 | openssl base64 -A
```
P5D7Jb4DwAdezkj9cxg9Z
```
/etc/redis/redis.conf
# requirepass foobared
```


## Deploy nginx proxy
refer to [ngx_http_proxy_connect_module](https://github.com/chobits/ngx_http_proxy_connect_module)
```sh
cd /tmp
apk add linux-headers libressl-dev git gcc pcre-dev zlib-dev make musl-dev
git clone https://github.com/chobits/ngx_http_proxy_connect_module.git
wget http://nginx.org/download/nginx-1.17.4.tar.gz
tar xvzf nginx-1.17.4.tar.gz
cd nginx-1.17.4
patch -p1 < /tmp/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_101504.patch
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --with-perl_modules_path=/usr/lib/perl5/vendor_perl --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-Os -fomit-frame-pointer' --with-ld-opt=-Wl,--as-needed --add-module=/tmp/ngx_http_proxy_connect_module

#./configure --add-module=/tmp/ngx_http_proxy_connect_module --sbin-path=/usr/sbin/nginx --modules-path=/etc/nginx/modules --conf-path=/etc/nginx/nginx.conf --prefix=/etc/nginx
make && make install
```

```sh
[root@hz12sepvm05-ft-node tmp]# cat /tmp/siftester.conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log debug;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

http {
    resolver 1.2.3.11;
    server {
        listen 8088;
        proxy_connect;
        proxy_connect_allow            443 563;
        proxy_connect_connect_timeout  10s;
        proxy_connect_read_timeout     10s;
        proxy_connect_send_timeout     10s;
        location / {
            proxy_pass $scheme://$http_host$request_uri;
        }
    }
}
```


## Known Issues
1. Jenkins job timeout is 30mins and robot.sh timeout is 25mins, but 25mins timeout takes no effect, so instead of kill the robot.sh process, the whole jenkins job is killed after 30mins timeout, so the mapping process at last can't run, some mapping data are lost.
2. dbagent pod may failed when pull image of siftester-dbagent, because the image is so large at more than 2G.
3. [Done] waiting the select test cases jenkins job finish before the FT select test cases
4. [TODO] Merge the siftester implementation back into official siftester

----------------------------------------------------------------------------------------------------
## Repo README

siftester
======

> This project contained siftester implementation for NTAS CI environment.

## Brief Description

Siftester implementation includes serval parts:
- codeEntityInclude
    this is the common header shared between codeEntityHandler & codeEntitySwitcher.
- codeEntityHandler
    this component will be built into a shared library linked into the SUT. it will
    collect the code entities like function addresses and memory mapping on the fly,
    then send them to dbagent for further analyze.
- codeEntitySwitcher
    a switch used to control the codeEntityHandler whether to send code entities to
    dbagent or not.
- DBAgent
    - receive the code entities and store them into local database;
    - receive the http message to do the mapping;
    - send the mapping data into central db.
- ctag_case_selection
    the functionalities used to select test cases for the branch based on the code
    and the function name to case name mapping generated from dbagent
- central_db
    the http server used to control the central db, like create the database; query
    the selected test cases etc.


the workflow as below:


## Steps

**Deploy Central DB Server**
On the central DB server
1. Create a central postgres database to store the mapping data;
2. Create a central redis server used as the cache and assistance database;
3. Start the central db service with systemd(default it will use TCP/5000)
4. In this implementation, I also created a jenkins service on the central db
   server to serve the case selection, you can also create another http server
   to serve it or integrate it into central db process.

> These steps has been integrated into [docker-compose](https://scm.cci.nokia.net/ntas/ft/siftester/-/blob/master/deploy/docker-compose.yml)
> can be started up with `docker-compose -p siftester up -d`

**Build the SUT with libcodeEntityHandler**
This step has already integrated into [ntas cmake system](https://gitlabe1.ext.net.nokia.com/tas/cmake/tree/ntas-19-0)
& [cci cmake](https://scm.cci.nokia.net/ntas/cmake-build), in order to enable it,
append the cmake command with -DENABLE_SIFTESTER:BOOL=ON. This will build

- libcodeEntityHandler.so
- codeEntitySwithcer
- link the SUT with libcodeEntityHandler.so
- #build the siftester-switcher docker image(manually)
- build the siftester-dbagent docker image based on debuginfo
- build other SUT docker images with the hacked SUT

**Modify the kubernetes manifacts to add siftester-switcher in each pod**
[kubernetes common](https://gitlabe1.ext.net.nokia.com/tas/kubernetes/blob/ntas-19-0/helm/nokia-tas/templates/_common.tpl#L557)
[each chart](https://gitlabe1.ext.net.nokia.com/tas/kubernetes/blob/ntas-19-0/helm/nokia-tas/charts/amc/templates/amc.yaml#L94)

**Run test cases to generate the mapping**
1. Send HTTP request to central db process to create the new database.
2. Deploy the lab with `deploy.sh xxxx --enable-siftester`, this will deploy the
   lab with siftester enable, additional FT pod dbagent(dbagent, postgresql, redis)
   will be created on TAS node, and port forward will be created on FT node to
   commonicate with central db and redis.
3. Run the test cases with `--enable-siftester`, this will send code entities to
   dbagent
4. when all the test cases on this executor finished, do mapping and send the mapping
   into central db
5. when all executors done, notify the centraldb process to do mapping_args action.

```
    when generating the mapping, the flow as below:
    +-----------------+
    | start to run FT |
    +-----------------+
            |
            | curl -d '{"common_tag":<common_tag>}' \
            |      -H "Content-Type: application/json" \
            |      -X POST \
            |      http://<ntas-siftester.dynamic.nsn-net.net>/<branch>/TestStart
            |
            | This action will create the new central database for siftester and set
            | the redis MAPPING_<base-branch> to new db name siftester_<common-tag>
            V
    +----------------------------------+
    | each case on each executor start |
    +----------------------------------+
            |
            | curl http://<dbagent>:8024/start?casename=<casename>
            |
            | begin to collect function addrs
            |
            V
    +---------------------------------+
    | each case on each executor stop |
    +---------------------------------+
            |
            | curl http://<dbagent>:8024/stop?casename=<casename>
            |
            | stop to collect function addrs
            V
    +------------------------------------+
    | all case finished on each executor |
    +------------------------------------+
            |
            | curl http://<dbagent>:8023/MappingStart?branch=<branch>
            |
            | begin to do mapping and combine the result into central db
            V
    +-------------------------------------------+
    | all the mapping collected into central db |
    +-------------------------------------------+
            |
            | curl -d '{"common_tag":<common_tag>}' \
            |      -H "Content-Type: application/json" \
            |      -X POST \
            |      http://<ntas-siftester.dynamic.nsn-net.net>/<branch>/MappingStart
            |
            | to do mapping_args, after this step will set redis
            | SIFTESTER_<base-branch> to new databse name siftester_<common-tag>
            |
    +-----------------------------------------+
    | Generate the finally mapping_args table |
    +-----------------------------------------+

    When select test case, will use the correct central databse according to
    the base branch.
```


**Select test cases**
1. send http request to jenkins to trigger the jenkins job to select the test cases, the
result will be stored on the central db server's redis
2. get the selected test cases by send http request to central db process.

```
    +----------------------------------------+
    | After Prepare package step             |
    +----------------------------------------+
            |
            | curl http://<siftester-jenkins>:8080/job/siftester-select-testcases/ \
            |      buildWithParameters\?token\=<token>\ \
            |      &BRANCH\=${BRANCH}\&ISMAINLINE\=${ISMAINLINE}\&\PARAMETERS\=${PARAM_STR}
            |
            |
    +--------------------------------------+
    | Begin to run FT job                  |
    +--------------------------------------+
            |
            | curl http://localhost/<branch>/testcases?common_tag=<common_tag>
            |
            | Then run the selected test cases
            |
```

## Touched repositories
[ci-scripts](https://gitlabe1.ext.net.nokia.com/tas/CIScripts)
- CiScripts/update_container_versions_siftester.py 
  update container_versions.json replace repository from ntas to siftester if siftester images build from CCI for this module.
- CiScripts/ci_rebase.sh 
  invoke `update_container_versions_siftester.py` for `ntas-19-0_siftester-map`
- scripts/prepare.sh 
  for `ntas-19-0_siftester-map` define para `ENABLE_SIFTESTER=ON`
  for feature branches, trigger select test cases job
- scripts/build-package.sh 
  for `ntas-19-0_siftester-map` `cmake ../ -G Ninja -DENABLE_SIFTESTER=ON` 
  for `ntas-19-0_sifteser-map` `ninja docker-siftester-dbagent` 

[taf-core](https://gitlabe1.ext.net.nokia.com/taf/core)
- templates/Deploy-And-FT/deploy_and_ft.gvy 
  send `TestStart` to central db to create database
  send `MappintDone` to central db to update database name for case selection  
- modules/test_cases/Collector.py 
  select test cases for each profile 
- scripts/ft.sh
  deploy the lab with `--enable-siftester` and run test cases with `--enable-siftester` 
- python_scripts/job_template 
  send `MappingDone` to each executor's dbagent to do mapping and combine the mapping into central db.

[cmake-build](https://gitlabe1.ext.net.nokia.com/tas/cmake)  
[kubernetes](https://gitlabe1.ext.net.nokia.com/tas/kubernetes)  
[ft/siftester](https://gitlabe1.ext.net.nokia.com/tas/siftester)  
[ft/single_vm_stuff](https://gitlabe1.ext.net.nokia.com/tas/single_vm_stuff)  
[ft/vtas-taf](https://gitlabe1.ext.net.nokia.com/tas/vtas-taf)  
[moby/siftester-dbagent](https://gitlabe1.ext.net.nokia.com/tas/docker-siftester-dbagent)  
[moby/siftester-switcher](https://gitlabe1.ext.net.nokia.com/tas/docker-siftester-switcher)  
[moby/build](https://gitlabe1.ext.net.nokia.com/tas/docker-build)  
  
[cci cmake-build](https://scm.cci.nokia.net/ntas/cmake-build)  
[cci ft/siftester](https://scm.cci.nokia.net/ntas/ft/siftester)  
[cci scm/cicd](https://scm.cci.nokia.net/ntas/scm/cicd)  
[cci moby/siftester-dbagent](https://scm.cci.nokia.net/ntas/moby/siftester-dbagent)  
[cci moby/siftester-switcher](https://scm.cci.nokia.net/ntas/moby/siftester-switcher)  
```sh
cd /home/felixdu/src/CIScripts && rg siftester
```
