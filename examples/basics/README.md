## The History and Architecture of NGINX

NGINX | NGINX
:-------------------------:|:-------------------------:
![NGINX Plus](images/nginx-plus-icon.png)|![NGINX Logo](images/nginx-logo.png)

NGINX was written in 2002 by Igor Sysov while he was working at rambler.ru, a web company providing Internet Search content.  As Rambler continued to grow, Igor kept hitting the practical limit of 10,000 simultaneous HTTP requests with Apache HTTP server.  The only way to handle more traffic was to buy and run more servers.  So he wrote NGINX to solve the `C10k concurrency problem` - how do you handle more than 10,000 concurrent requests on a single Linux server.  

Igor created a new TCP connection and request handling concept called the `NGINX Worker`.  The Workers are Linux processes that continually wait for incoming TCP connections, and immediately handle the Request, and deliver the Response.  It is based on event-driven computer program logic written in the `native C programming language`, which is well-known for its speed and power.  Importantly, NGINX Workers can use any CPU, and can scale in performance as the compute hardware scales up, providing a nearly linear performance curve.  There are many articles written and available about this NGINX Worker architecture if you are interested in reading more about it.  

Another architecural concept in NGINX worth noting, is the `NGINX Master` process.  The master process interacts with the Linux OS, controls the Workers, reads and validates config files before using them, writes to error and logging files, and performs other NGINX and Linux management tasks.  It is considered the Control plane process, while the Workers are considered the Data plane processes.  The `separation of Control functions from Data handling functions` is also very beneficial to handling high concurrency, high volume web traffic.

NGINX also uses a `Shared Memory model`, where common elements are equally accessed by all Workers.  This reduces the overall memory footprint considerably, making NGINX very lightweight, and ideal for containers and other small compute environments.  You can literally run NGINX off a legacy floppy disk!

In the `NGINX Architectural` diagram below, you can see these different core components of NGINX, and how they relate to each other.  You will notice that Control and Management type functions are separate and independent from the Data flow functions of the Workers that are handling the traffic.  You will find links to NGINX core architectures and concepts in the References section.

>> It is this unique architecture that makes NGINX so powerful and efficient.

![NGINX Architecture](images/lab1_nginx-architecture.png)

- In 2004, NGINX was released as open source software (OSS).  It rapidly gained popularity and has been adopted by millions of websites.

- In 2013, NGINX Plus was released, providing additional features and Commercial Support for Enterprise customers.

<br/>

## So what is NGINX Plus?  

NGINX Plus 
:-------------------------:
![NGINX Plus](images/nginx-plus-icon.png)

NGINX Plus is the `Commercial version of NGINX`, with additional Enterprise features on top of the base NGINX Opensource OSS build. Here is a Summary list of the Plus features:

- Dynamic reconfiguration reloads with no downtime
- Dynamic NGINX software updates with no downtime
- Dynamic DNS resolution and DNS Service discovery
- Active Health Checks
- NGINX Plus API w/statistics and dashboard, over 240 metrics
- NGINX JavaScript Prometheus exporter libraries
- Dynamic Upstreams
- Key Value store
- Cache Purge API controls
- NGINX Clustering for High Availability
- JWT processing with OIDC for user authentication
- NGINX App Protect Firewall WAF

# (Install NGINX Plus) SKIP - DONE Already!!!
	
	sudo mkdir -p /etc/ssl/nginx
 	sudo cp <downloaded-file-name>.crt /etc/ssl/nginx/nginx-repo.crt
	sudo cp <downloaded-file-name>.key /etc/ssl/nginx/nginx-repo.key
 	sudo apt update && \
	sudo apt install apt-transport-https \
                 lsb-release \
                 ca-certificates \
                 wget \
                 gnupg2 \
                 ubuntu-keyring
 	wget -qO - https://cs.nginx.com/static/keys/nginx_signing.key \
    	| gpg --dearmor \
    	| sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
 		
   	#Add the NGINX Plus repository
  	printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
	https://pkgs.nginx.com/plus/ubuntu `lsb_release -cs` nginx-plus\n" \
	| sudo tee /etc/apt/sources.list.d/nginx-plus.list

  	#Download the nginx-plus apt configuration to /etc/apt/apt.conf.d
   	sudo wget -P /etc/apt/apt.conf.d https://cs.nginx.com/static/files/90pkgs-nginx
	#Update the repository information
 	sudo apt update
  	#Install the nginx-plus package
   	sudo apt install -y nginx-plus
	#Check installed version
 	nginx -v
	
# LAB-1: Explore - Review NGINX Plus Default Configuration
## UDF > Components > Nginx-plus-apigw > Access > Web Shell

### Exploration Commands ###

	# Check running processes status
 	ps aux | grep nginx
  	# Check nginx service status
   	systemctl status nginx

  	# Check NGINX Deployment
 	cd /etc/nginx/
 	cat nginx.conf
	cd conf.d/
	cat default.conf
	sudo mv default.conf default.conf.bak #ignore if not present
	sudo nginx -t
	sudo nginx -T
	sudo nginx -v
	sudo nginx -V
	sudo nginx -s reload

# LAB-2: Web Server Configuration
![NGINX as WS](images/nginx-web-server-architecture.png)
## UDF > Components > Nginx-plus-apigw > Access > Web Shell	
### Create/Copy files to be served by NGINX Web Server ###
	cd /opt/services/
	cd App1/
	ls
	cat index.html
	cd /etc/nginx/conf.d/
	sudo vi web.conf

 	cd /opt/services/
	cd App2/
	ls
	cat index.html
	cd /etc/nginx/conf.d/
	sudo vi web.conf

	cd /opt/services/
	cd App3/
	ls
	cat index.html
	cd /etc/nginx/conf.d/
	sudo vi web.conf
 
### web.conf ###
server {
    
    listen       9001;
    index  index.html;
   
    location / {
    root   /opt/services/App1;
    }
}

server {
    
    listen       9002;
    index  index.html;

    location / {
    root   /opt/services/App2;
    }
}

server {
    
    listen       9003;
    index  index.html;

    location / {
    root   /opt/services/App3;
    }
}
### ###

	sudo nginx -t
	sudo nginx -s reload
	curl 10.1.1.11:9001
	curl 10.1.1.11:9002
	curl 10.1.1.11:9003


# LAB-3: Configuring Load Balancer
![NGINX as LB](images/nginx-as-rp.png)
## UDF > Components > NIM > ACCESS > NIM UI
### Let's use NGINX Instance Manager (NIM) for this part!
NIM is the awesome NGINX Central Management tool, you can find details here:
[**NIM**](https://docs.nginx.com/nginx-instance-manager/)

### Acces the NIM UI and find your NGINX instances ###

To sign in use the username and password available in UDF Description. 

![NIM-Access](images/NIM-UDF-sign-in.webp)

Discover the instances in NGINX Instance Manager.

After you are done with exploring the intuitive NIM UI, create the lb.conf in the directory /etc/nginx/conf.d/ :

NIM UI > Instance Groups > Click "Add File" and Enter the file path and name: lb.conf 
![NIM-add-file](images/nim-oltra-ph-lb-conf-create.png)
	

	### lb.conf ###
	upstream backend_servers {
    zone backend_server_zone 64k;
    server 127.0.0.1:9001;
    server 127.0.0.1:9002;
	}

	server {
    listen 9000;
    autoindex on;

    location / {
    proxy_pass http://backend_servers/;
    #health_check;

    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP  $remote_addr;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    }
}

### Test Load Balancer ###

	sudo nginx -t
	sudo nginx -s reload
	curl localhost:9000
	curl localhost:9000
	curl localhost:9000


# LAB-4: Configuring NGINX Plus Dashboard

### dashboard.conf ###

server {

    listen       8080;

        location /api {
        api write=on;
        allow all;
    }

    location / {
    root /usr/share/nginx/html;
    index   dashboard.html;
    }
}

### ###


# LAB-5: Configuring NGINX Plus as an API Gateway

In this LAB, we will configure the NGINX as an API Gateway for an Httpbin API. Httpbin api is deployed in a modern environment, i.e., the Kubernetes Cluster named "rancher2". We will enable Rate Limiting for any request to the API. For the Commands listed below, use the VSCode Terminal available at UDF > Client-vscode > VSCODE > Terminal

![find-vscode](images/UDF-client-machine-vscode.png)

![find-vscode-terminal](images/vscode-terminal.png)

## Explore the API Deployment ##
### Check current Context ###
	kubectl config get-contexts

### Switch to Rancher 2 Cluster ###
	kubectl config use-context rancher2


### Test the API directly ###
	curl http://10.1.20.22:30080/get

### Configure the NGINX for a simple API Gateway functionality ###

Copy the [httpbin-ngx.conf](httpbin/httpbin-ngx.conf) to the Nginx instance named "Nginx-plus-apigw" in the UDF. 
You can use Nginx-plus-apigw > ACCESS > WEB SHELL and create the file in /etc/nginx/conf.d/httpbin-ngx.conf OR alternatively, you can use NIM and follow the process like you did for Load Balancer configuration, above. 

Before configuring, let us examine the simple API Gateway configuration file. 

### Test through NGINX ###
	curl http://10.1.1.11:8000/get

### Rate Limiting test with no violation ###
	for i in {1..10}; do   curl -i -s -o /dev/null -w "%{http_code}\n" http://10.1.1.11:8000/get;   sleep 1; done

### Rate Limiting test with violation ###
	for i in {1..10}; do   curl -i -s -o /dev/null -w "%{http_code}\n" http://10.1.1.11:8000/get;  done


## End of LAB Session 1 ##
### For LAB Session 2: NGINX as Kubernetes Ingress Controller, Pls USE: https://github.com/tareqaminul/oltra-ph-25/tree/main/examples/nic ###
