# HAProxy Load Balancer - Basic

## Setup HAProxy service
```
$ ansible-playbook --inventory hosts haproxy-setup.yml
```

## HAProxy blog (basic)

### What is Load Balancing
	https://www.haproxy.com/blog/what-is-load-balancing/
	https://www.haproxy.com/blog/layer-4-and-layer-7-proxy-mode/
	
### Http load balancing
	https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers/
	https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy/
	https://www.haproxy.com/blog/introduction-to-haproxy-acls/
	https://www.haproxy.com/blog/how-to-map-domain-names-to-backend-server-pools-with-haproxy/
	
### Http traffic with ssl/tls
	https://www.haproxy.com/blog/haproxy-ssl-termination/
	https://www.haproxy.com/blog/redirect-http-to-https-with-haproxy/
	
### Http rewrite
	https://acloudguru.com/hands-on-labs/using-http-rewrites-with-haproxy
	
### Monitoring
https://www.haproxy.com/blog/exploring-the-haproxy-stats-page/![image](https://user-images.githubusercontent.com/3235983/182773416-58062823-f777-4e7e-b096-3bed57216404.png)

## Create Some Test Files  

We're going to need some test files for our web server containers. We're going to use 6 containers in 2 groups
of 3. We'll use the figlet command to spice up ourtext files a bit!  

Let's create the files:  

```
mkdir -p ~/testfiles
```

```
for site in `seq 1 2`; \
  do for server in `seq 1 3`; \
    do figlet -f big SITE$site - WEB$server > ~/testfiles/site$site\_server$server.txt ; \
  done ; \
done
```

## Start Some Web Server Containers

Ok, so now that we've got the odds and ends covered, let's stand up our web server containers. We're going to
start a total of 6 `nginx` containers using `Docker`, simulating 2 sites, with 3 web servers per site. Our web
servers will be available on ports `8081` through `8086`.  

See [how to install Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04).  

Let's start our containers:  

```
port=1 ; \
for site in `seq 1 2`; \
  do for server in `seq 1 3`; \
    do sudo docker run -dt --name site$site\_server$server -p 808$(($port)):80 docker.io/library/nginx ; \
    port=$(($port + 1 )) ; \
  done ; \
done
```

Copy our Test files:  

```
for site in `seq 1 2`; \
  do for server in `seq 1 3`; \
    do sudo docker cp ~/testfiles/site$site\_server$server.txt site$site\_server$server:/usr/share/nginx/html/test.txt ; \
  done ; \
done
```

Test our Web services:  

```
port=1 ; \
for site in `seq 1 2`; \
  do for server in `seq 1 3`; \
    do curl -s http://127.0.0.1:808$port/test.txt ; \
    port=$(($port + 1 )) ; \
  done ; \
done | more
```

## Manage HAProxy with Runtime API

Once the Runtime API is enabled, it can be accessed manually by using the command “socat”: 

[See](https://www.haproxy.com/blog/dynamic-scaling-for-microservices-with-runtime-api/#haproxy) the article about it.

```
$ sudo apt install socat
$ echo "help" | sudo socat stdio /run/haproxy/admin.sock

## 'set server <srv> state' expects 'ready', 'drain' and 'maint'
$ echo "set server site1/site1-web1 state maint" | sudo socat stdio /run/haproxy/admin.sock
$ echo "set server site1/site1-web1 state ready" | sudo socat stdio /run/haproxy/admin.sock
```
