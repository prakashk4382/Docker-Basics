Cheatsheet
============================
1 Installation
-------------------


### ON UBUNTU 14-10

**Repo install usually back leveled** 

    sudo apt-get install -y docker.io  #
    sudo usermod -aG docker vagrant
    docker info
    docker -v 
    docker version
    sudo service docker restart

**Use (for latest)** 

    wget -qO- https://get.docker.com/ | sh

**Pre release**

    wget -qO- https://test.docker.com/ | sh

**Basic Commands**

    docker info
    docker -v 
    docker version
    sudo service docker restart

### Centos 7  

**Repo install usually back leveled** 

    sudo yum install docker
    sudo service docker start
    sudo chkconfig docker on #(start at boot)
    docker info
    docker version
    sudo service docker restart

**Alternativly (for latest)** 

    sudo wget https://get.docker.com/builds/Linux/x86_64/docker-latest -O /usr/bin/docker
    sudo chmod +x /usr/bin/docker
    sudo /usr/bin/docker -d &
    sudo docker info
    sudo docker version
    
**(it’s a plane install, no docker conf.. not startup configs..)**

### On Fedora 20 

    sudo yum -y remove docker
    sudo yum install docker-io
    sudo service docker start
    sudo chkconfig docker on (start at boot)
 
 2 - Run Command Basics
 ------------------- 
 
 
  
**Run first container** 

    docker ps -a
    docker run busybox 
    docker ps
    docker ps -a
    docker run -i busybox  (STDIN)
    ls
    pwd
    hostname

**(Crtl-d) to cancel**

    docker ps
    docker ps -a
    docker run -t busybox

**Note: a ctrl-c doesn't kill it** 

    docker ps
    docker ps -a
    docker run -it busybox

**(basic Shell)** 

    hostname
    ps -ef 
    exit 
    docker ps
    docker ps -a


    docker run -it busybox
    hostname
    ps -ef 

**ctrl-p-q (quits without killing the container)**

    docker ps
    docker ps -a
    docker run -d busybox

**(It’s gone)**

    docker ps
    docker ps -a


    docker run -itd busybox
    docker ps 
    docker attach xxxx
    ls

**ctrl-p-q (quits without killing docker attach the container***

    docker ps --no-trunc=true
 
 **(long CID)** 

    cid=$(docker run -itd busybox)
    echo $cid
    docker inspect $cid
    docker inspect --format '{{.NetworkSettings.IPAddress}}' $cid
    docker stop $cid

**Or.. .use the name** 

    docker run --name john1 -itd busybox
    docker attach john1
    docker stop john1
    docker run -itd busy box
    docker run -itd busy box
    docker run -itd busy box
    docker run -itd busy box
    docker run --name john2 -itd busybox
    docker run --name john3 -itd busybox
    docker ps -q 


     docker ps 
     docker rm <cid>
     docker rm $(docker ps -aq)
     

 3 - Demystifying Volumes
 ------------------- 
 

 
**setup**

```
FROM ubuntu:14.04

MAINTAINER John Willis  <john@socketplane.io>

VOLUME ["/john99"]

CMD ["/bin/sh"]
```
    docker build -f myimage.dockerfile -t myimage .
    
**Volumes**

(***Simple Run***)

    docker run -it -v /john1 busybox
    cd john1
    touch file1
    ctrl-p-q   # Keep in running 
    
    docker restart <cid>
    docker exec <cid> ls /john1

  The file stays until we remove the container  
  however, if we start a new container based on the orgi busybox image

    docker run -it -v /john1 busybox
    cd john1
    ls  (not there)   This was a new run..
    exit

  We can also have volumes that have volumes defined in the imagee build.. 

    docker images   (show my image)  
    docker inspect <imageid>
    docker history <imagename>

    docker run -itd myimage
    cd mydir 
    ls    (file1 will alwys be there)
    touch file2  (same rues apply) 
    ls
    exit

    docker run -it -v /john1 myimage
    ls   (both directories) 
   
   cd into both… 

   on docker host.. 
```   
mkdir john3
cd john3
touch file3
touch file4
   
docker run -it -v /vagrant/john3:/john3 myimage    (host needs to be abs path)
```    
  ## good for testing src code… 
```
cd john3
ls
touch file5
ls
exit
```
  on docker host.. 
```
cd john3
ls   (see files 3 and 4 and 5)
 
docker run -it -v ~/john3:/john3:ro myimage   (point out ~) 
cd john3
vi file5
save???

docker run -it -v ~/.bash_history:/.bash_history myimage

docker ps -a   (see what's running)
docker kill $(docker ps -q)
docker rm $(docker ps -aq)   
docker ps -a

docker run -it --name john1 -v ~/john3:/john3 myimage
ls  (directories are there.. and files trust me) 
### ctrl-pq   (keep running) 

docker ps

docker run -it --name john2 --volumes-from john1 myimage
ls
###  ctrlpq

docker run -it --name john3 --volumes-from john2 myimage

***(make a backup)**

docker run --rm --volumes-from john1 -v $(pwd):/backup busybox tar cvf /backup/john2.tar /john
```

4 - More Run Commands 
------------------- 



Look at docker hub
check out ubuntu, centos and fedora

**Search dockerhub** 

    docker -v 
    history
    docker search ubuntu
    docker search -s 10 ubuntu
    docker search --no-trunc=true -s 10 ubuntu
    
    docker images
    docker pull ubuntu
    docker pull ubuntu:14.04
    docker pull ubuntu:trusty
    docker images

( explain all 5 tags and tagging)

    docke
    docker ps    (look at what happened.. ran /bin/bash)

    docker run -itd ubuntu /bin/sh
    docker ps  (look at what happened.. ran /bin/sh)
    
    docker history <image id>   (Show the CMD) 

    docker run -itd ubuntu uname -a
    docker ps -a  (look at what happened..no in the CMD STDIN gone)
    docker logs <cid>

    docker run -itd ubuntu sleep 10 && watch docker ps

**cleanup**

    docker kill $(docker ps -q) && docker rm $(docker ps -aq)
   
    docker run -itd --name job1 ubuntu /bin/sh -c "while true; do echo Docker Rocks; sleep 1; done"
    
    docker logs -h
    docker logs -ft job1
 
    docker kill job1
    docker rm job1 
    docker ps -a

    docker run -itd --name job2 ubuntu /bin/sh -c "while true; do echo Docker Rocks; sleep 1; done"
    docker kill $(docker ps -lq)  (stop the last container) 
    docker rm $(docker ps -lq)
    
**Fun with PIDS**

    docker run -itd --name job2 ubuntu /bin/sh -c "while true; do echo Docker Rocks; sleep 1; done"
    docker stats job2
    watch docker top job2 -ef  (watch the pid change) 
    docker exec -itd job2 sleep 20
    watch docker top job2 -ef
    
**Using Inspect ** 

    docker inspect -h 
    docker inspect job2
    docker inspect --format '{{.Name}} {{.State.Running}}' job2
     
**Fun with 1.6 **

    docker ps -a
    docker run  -itd --name job4 --label=NodeNumber=3 --label=NodeType=cluster ubuntu
    docker inspect job4 > jmw.out  (find /Labels) 
    docker inspect --format '{{.Name}} {{.Config.Labels.NodeType}}' job4
    
    cid=$(docker run -itd ubuntu)
    docker attach $cid
    ulimit -a
    cid=$(docker run -itd --ulimit nofile=1024:1024 ubuntu)
    docker attach $cid
    ulimit -a
    
```    
cid=$(docker run -itd --ulimit nofile=1024:1024 --ulimit core=102400 --ulimit nproc=1000 --ulimit nice=100 --ulimit memlock=8196 --ulimit fsize=8192 --ulimit rss=4096 --ulimit cpu=4 --ulimit locks=1000 --ulimit sigpending=100 --ulimit msgqueue=1000 --ulimit nice=100 --ulimit rtprio=100 ubuntu)

docker attach $cid
ulimit -a
```
    
5 - Basic Networking
------------------- 



##  Note: Networking Changed in 1.9 but this tutorial still covers the default Bridge networking...

** Setup **


     sudo apt-get install -y bridge-utils curl
     docker run -d --name db -p 3306:3306 mysql
     docker run -d --name wp1 -p 80:80 wordpress
     docker run -d --name wp2 -p 81:80 wordpress

**Commands **

     ip a  ( explain eth0=nat eth1 my ip, docker0= bridge)  (docker0 sunset /16 65k)

    ip a show docker0

**explain docker0 (this is a virtual bridge and the IP is the gateway for all contianers on this docker host) **
**… 172.17.42.1/16**

     docker run -itd —name u1 ubuntu
     docker exec u1 ip a

     ip a 

     brctl show docker0  (explain veth each container has a eth0 veth pair)

     docker run -it —name u2 ubuntu
     sudo apt-get update
     sudo apt-get install -y inetutils-traceroute

     traceroute docker.com

**(look at iptables nat configuration)**
     sudo iptables -t nat -L -n  (docker also setup masquerade rule for all container traffic). 
**allows any outbound but by default no inbound…(show the masa rule)**

**(these rules are configured dynamically by docker based on how you use the EXPOSE** 
     docker history httpd   (look at the expose cmd for port 80)

     docker run -itd --name web1 -P httpd

     docker exec web1 ip a   (look at the ip address)

     sudo iptables -t nat -L -n

     curl localhost:32768

     docker run -itd --name web2 -p 80 httpd

     sudo iptables -t nat -L -n   (look at the new DNAT rule)

     docker run -itd --name web3 -p 8080:80 httpd

     sudo iptables -t nat -L -n   (look at the new DNAT rule)

     curl localhost::8080

**on the LAMP stack setup docker host**

     docker ps (look at my three machines… ) 

     ip a  ( explain eth0=nat eth1 my ip, docker0= bridge)  (docker0 sunset /16 65k)

     brctl show docker0  (explain veth )

     sudo iptables -t nat -L -n  (docker also setup masquerade rule for all container traffic). 

**allows any outbound but by default no inbound…(show the masa rule)**

     ip addr show eth1

**hit the web page… at 80 then at 8080 then 1936**

     cat /etc/haproxy/haproxy.cfg

     docker run -d --name wp3 -p 82:80 wordpress

     docker exec wp3 ip a

     sudo vi /etc/haproxy/haproxy.cfg   (add new interface)

     sudo service haproxy reload 

**hit 1936*

6 Dockerfile (Part 1) 
------------------- 


### ON UBUNTU 14-10

**Basic Three Commands FROM, RUN and CMD** 

apache-ex1
```
FROM ubuntu:14.04

RUN apt-get -y install apache2 

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
Commands
```
docker build -f apache-dockerfile-ex1 -t apache-ex1 .

docker images
    
cid=$(docker run -itd apache-ex1)
    
docker exec $cid ip a

nid=$(docker inspect --format '{{.NetworkSettings.IPAddress}}' $cid)
    
curl $nid
```
**Caching** 
```
docker build -f apache-dockerfile-ex1 -t apache-ex1 .
    
docker build --no-cache=true -f apache-dockerfile-ex1 -t apache-ex1 .
```

**Alternate Syntax** 

apache-ex2
```
FROM ubuntu:14.04

RUN ["apt-get","install","-y","apache2"]

CMD /usr/sbin/apache2ctl -D FOREGROUND
```

**Finding index.html** 
```
docker run -it apache-ex1 /bin/sh

find / -name index.html
```

**More Commands ADD and EXPOSE** 

apache-ex3
```
FROM ubuntu:14.04

RUN \
  apt-get update && \
  apt-get -y install apache2 

ADD index.html /var/www/html/index.html

EXPOSE 80 

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

index.html
```
<h1>Docker Rocks!</h1>
```
Commands
```
cat index.html 
    
docker build -f apache-dockerfile-ex3 -t apache-ex3 .
    
cid=$(docker run -itd -P apache-ex3)
    
nid=$(docker inspect --format '{{.NetworkSettings.IPAddress}}' $cid)
    
curl $nid
```
Show from Web Browser 
```
ip a show eth1

docker ps -a
```

**Volume Command** 

apache-ex4
```
FROM ubuntu:14.04

VOLUME [ "/var/www/html" ]

ADD index.html /var/www/html/index.html

RUN \
  apt-get update && \
  apt-get -y install apache2 

EXPOSE 80 

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
Commands
```
docker build -f apache-dockerfile-ex4 -t apache-ex4 .
    
cid=$(docker run -itd -v ~/test/index.html:/var/www/html/index.html wordpress-ex4)
    
nid=$(docker inspect --format '{{.NetworkSettings.IPAddress}}' $cid)
    
curl $nid
```

Modify index.html  

    curl $nid
    
**Final Touches** 

apache-ex5
```
FROM ubuntu:14.04

MAINTAINER John Willis  <john@socketplane.io>
ENV REFRESHED_AT 2016-04-20 

VOLUME [ "/var/www/html" ]
WORKDIR /var/www/html

ADD index.html /var/www/html/index.html

RUN \
  apt-get update && \
  apt-get -y install apache2 

EXPOSE 80 

ENTRYPOINT ["/usr/sbin/apache2ctl"]
CMD ["-D", "FOREGROUND"]
```
Show Using Variable for Refresh
```
docker build -f apache-dockerfile-ex5 -t apache-ex5 .
    
docker build -f apache-dockerfile-ex5 -t apache-ex5 .
```

Modify REFRESH VARIABLE in the Dockerfile
```
docker build -f apache-dockerfile-ex5 -t apache-ex5 .
```    

8 – Troubleshooting with Sysdig
------------------- 


9 Docker Machine
------------------- 



## Note ... Installation on MAC and Windows at 1.8 or above should be done via the Docker Toolbox

### Installation on OSX

**Install Machine** 
```
sudo wget --no-check-certificate -O /usr/local/bin/docker-machine https://github.com/docker/machine/releases/download/v0.3.0/docker-machine_darwin-amd64

sudo chmod +x /usr/local/bin/docker-machine

docker-machine -v
```
**Install Docker (client)** 
```
sudo wget --no-check-certificate -O /usr/local/bin/docker https://get.docker.com/builds/Darwin/x86_64/docker-latest

sudo chmod +x /usr/local/bin/docker

docker -v
```

### Running Machine on Virtualbox

```
docker-machine create --driver virtualbox dev1

docker-machine ls

eval "$(docker-machine env dev1)"

docker run busybox echo hello world

docker ps -a

```  

### Running Machine on Digital Ocean

```
docker-machine create --driver digitalocean --digitalocean-access-token=$DIGITAL_OCEAN_TOKEN dev2

docker-machine ls

eval "$(docker-machine env dev2)"

docker-machine ls   (show the active machine)

docker run busybox echo hello world

docker ps -a
```  

### Running Machine on Amazon EC2

```
docker-machine -D create --driver amazonec2 --amazonec2-access-key $AWS_ACCESS_KEY_ID --amazonec2-secret-key $AWS_SECRET_ACCESS_KEY --amazonec2-vpc-id $AWS_VPC_ID --amazonec2-zone b dev3

docker-machine ls

eval "$(docker-machine env dev3)"

docker-machine ls   (show the active machine)

docker run busybox echo hello world

docker ps -a
``` 

10 -  Docker Compose
------------------- 


## Note ... Installation on MAC and Windows at 1.8 or above should be done via the Docker Toolbox

### Installation on OS X

**Install Compose** 
```
sudo wget --no-check-certificate https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m`

sudo mv docker-compose-`uname -s`-`uname -m` /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```
**Install Docker (client)  (if not already installed)** 
```
sudo wget --no-check-certificate -O /usr/local/bin/docker https://get.docker.com/builds/Darwin/x86_64/docker-latest

sudo chmod +x /usr/local/bin/docker

docker -v
```
**Canonical Docker-Compose (Python/Redis) example**

docker-compose.yml
```
web:
  build: .
  command: python app.py
  ports:
   - "5000:5000"
  volumes:
   - .:/code
  links:
   - redis
 redis:
  image: redis
```

Dockerfile
```
FROM python:2.7
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
```

app.py
```
from flask import Flask
from redis import Redis
import os
app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello World! I have been seen %s times.' % redis.get('hits')

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

requirements.txt 
```
flask
redis
```

Commands
```
docker-compose up -d

docker-compose ps

docker-compose logs

curl localhost:5000

docker-compose stop
```

**Tomcat Sample example**
```
docker pull tomcat

docker history tomcat | grep -i expose

cid=$(docker run -d -P tomcat)

docker ps

curl localhost:32xxx 

docker kill $cid

docker rm $cid
```

**Now let's Compose it...**

compose-ex1.yml
```
tomcatapp:
  image: tomcat
  ports:
   - "8080"
```

Commands
```
docker-compose -f compose-ex1.yml up -d

docker-compose -f compose-ex1.yml ps

docker-compose -f compose-ex1.yml logs

curl localhost:32xxx 

docker-compose -f compose-ex1.yml stop

docker-compose -f compose-ex1.yml rm
```

**Now let's Compose it... (add a sample.war file)**

Figure out where the webapps directory is
```
docker run -it tomcat bash 

ls   
```
nginx.conf
```
worker_processes 1;

events { worker_connections 1024; }

http {

    sendfile on;

    gzip              on;
    gzip_http_version 1.0;
    gzip_proxied      any;
    gzip_min_length   500;
    gzip_disable      "MSIE [1-6]\.";
    gzip_types        text/plain text/xml text/css
                      text/comma-separated-values
                      text/javascript
                      application/x-javascript
                      application/atom+xml;

    # List of application servers
    upstream app_servers {

        server tomcatapp1:8080;
        server tomcatapp2:8080;
        server tomcatapp3:8080;

    }

    # Configuration for the server
    server {

        # Running port
        listen [::]:80;
        listen 80;

        # Proxying the connections connections
        location / {

            proxy_pass         http://app_servers;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;

        }
    }
}
```

compose-ex2.yml
```
nginx:
  image: nginx
  links:
   - tomcatapp1:tomcatapp1
   - tomcatapp2:tomcatapp2
   - tomcatapp3:tomcatapp3
  ports:
   - "80:80"
  volumes:
   - nginx.conf:/etc/nginx/nginx.conf
tomcatapp1:
  image: tomcat
  volumes:
   - sample.war:/usr/local/tomcat/webapps/sample.war
tomcatapp2:
  image: tomcat
  volumes:
   - sample.war:/usr/local/tomcat/webapps/sample.war
tomcatapp3:
  image: tomcat
  volumes:
   - sample.war:/usr/local/tomcat/webapps/sample.war
```

Commands
```
export COMPOSE_FILE=compose-ex2.yml

docker-compose up -d

docker-compose ps 

docker exec composetest_nginx_1 cat /etc/hosts

docker exec composetest_tomcatapp1_1 ip a 

docker exec composetest_tomcatapp2_1 ip a

docker exec composetest_tomcatapp3_1 ip a

curl http://localhost/sample/

docker-compose stop
```

11 -  Docker Swarm
------------------- 



**Setup** 
```
docker-machine create --driver virtualbox dev1

eval "$(docker-machine env dev1)"

docker pull swarm

docker history swarm

docker run swarm     #(get help)
```

**Create a Cluster** 
```
sid=$(docker run swarm create) 

files $ echo $sid
1ffdb70193793b943df9456a35c24817
```

**Create the Swarm Manager**
```
docker-machine create -d virtualbox --swarm --swarm-master --swarm-discovery token://$sid swarm-master

docker-machine ls

eval "$(docker-machine env swarm-master)"

docker-machine ls

docker info
```

**Create Swarm Nodes**
```
docker-machine create -d virtualbox --engine-label itype=frontend --swarm --swarm-discovery token://$sid swarm-node-01

docker-machine create -d virtualbox --swarm --swarm-discovery token://$sid swarm-node-02

docker-machine create -d virtualbox --swarm --swarm-discovery token://$sid swarm-node-03

docker-machine ls

docker-machine env --swarm swarm-master   # (checkout the different port)

eval "$(docker-machine env --swarm swarm-master)"

docker-machine ls    (Notice non of the docker machines have the asterick) 

docker info

docker run swarm list token://$sid

docker ps     #(no containers are running in the swarm)
```
**Look at the Four Nodes**
```
docker-machine ls

eval "$(docker-machine env swarm-master)"

docker ps

eval "$(docker-machine env swarm-node-01)"

docker ps

eval "$(docker-machine env swarm-node-02)"

docker ps

eval "$(docker-machine env swarm-node-03)"

docker ps
```

**Running Docker Instances with Swarm  (explain Spead vs Binpack**
```
eval "$(docker-machine env --swarm swarm-master)"

docker ps 

docker run -itd --name engmgr ubuntu 

docker ps 

for i in `seq 1 6`; do docker run -itd -e constraint:itype!=frontend --name eng$i ubuntu; done

docker ps 

docker run -itd --name engmgr-c -e affinity:container==engmgr ubuntu
```
**Cleanup**
```
docker-machine kill $(docker-machine ls -q)

docker-machine rm $(docker-machine ls -q)
```

