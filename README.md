# Nginx-Load-Balancing

* ```Round robin```: In this method, the request is distributed in a roundrobin way. When no method is specified for distribution, it is used automatically.
* ```Ip hash```: It is a method in which hash functions are used to identify the server to be used for the next request based on the IP address of the clients. With the help of this method, you can opt for session persistence.
* ```Least connected```: Using this method, you can assign the upcoming request to the server that is not so busy, i.e., the server with few active connections.

## Install Nginx on Load Balancer Host
First, you will need to install the Nginx server on the Load Balancer host. You can install it by running the following command:

```
apt-get install nginx -y
```
Once the Nginx has been installed, start the Nginx service and enable it to start at system reboot:

```
systemctl start nginx
systemctl enable nginx
```
Now, you can verify the status of the Nginx service using the following command:

```
systemctl status nginx
```
If everything is fine, you will get the following output:

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-10-11 19:22:31 UTC; 26s ago
       Docs: man:nginx(8)
    Process: 62622 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 62623 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 62716 (nginx)
      Tasks: 2 (limit: 1041)
     Memory: 3.9M
     CGroup: /system.slice/nginx.service
             ├─62716 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             └─62719 nginx: worker process

Oct 11 19:22:31 ubuntu systemd[1]: Starting A high performance web server and a reverse proxy server...
Oct 11 19:22:31 ubuntu systemd[1]: Started A high performance web server and a reverse proxy server.
```
By default, Nginx web server listens on port 80. You can check it by running the following command:


ss -antpl
You should see the Nginx port 80 in the following output:



## Configure Nginx as a Load Balancer
Next, you will need to create an Nginx virtual host configuration file for the load balancing setup. You can set up the Nginx load balancing using the three methods.

### 1. Set Up Nginx Using Round robin Method
You will need to create an Nginx virtual host configuration file to implement the load balancer. You can create it with the following command:

```
nano /etc/nginx/conf.d/loadbalancer.conf
```
Add the following configuration for round robin method:

```
upstream backend {
        server 192.168.0.11;
        server 192.168.0.12;
    }

    server {
        listen      80;
        server_name 192.168.0.10;

        location / {
	        proxy_redirect      off;
	        proxy_set_header    X-Real-IP $remote_addr;
	        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header    Host $http_host;
		proxy_pass http://backend;
	}
}
```
Where: 192.168.0.10 is the IP address of load balancer, 192.168.0.11 is the IP address of the first backend server and 192.168.0.12 is the IP address of the second backend server.

Save and close the file when you are finished then restart the Nginx service to apply the changes:

```
systemctl restart nginx
```
In this method, the request is distributed in a round robin way.

### 2. Setup Nginx Using IP Hash Method

In this section, we will set up Nginx load balancing using the Ip hash method. This method uses an algorithm that takes the source and destination IP addresses of the client and server to generate a unique hash key. This key is used to allocate the client to a particular server.

First, create an Nginx virtual host configuration file with the following command:

```
nano /etc/nginx/conf.d/loadbalancer.conf
```
Add the following configuration:

```
upstream backend {
	ip_hash;
        server 192.168.0.11;
        server 192.168.0.12;
    }

    server {
        listen      80;
        server_name 192.168.0.10;

        location / {
	        proxy_redirect      off;
	        proxy_set_header    X-Real-IP $remote_addr;
	        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header    Host $http_host;
		proxy_pass http://backend;
	}
}
```
Save and close the file when you are finished then restart the Nginx service to apply the changes:

```
systemctl restart nginx
```

### 3. Set Up Nginx Using Least Connected Method
In this section, we will set up Nginx load balancing using the Least Connected method. This method directs the requests to the server with the least active connections at that time. First, create an Nginx virtual host configuration file with the following command:

```
nano /etc/nginx/conf.d/loadbalancer.conf
```
Add the following configuration:

```
upstream backend {
        least_conn;
        server 192.168.0.11;
        server 192.168.0.12;
    }

    server {
        listen      80;
        server_name 192.168.0.10;

        location / {
	        proxy_redirect      off;
	        proxy_set_header    X-Real-IP $remote_addr;
	        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header    Host $http_host;
		proxy_pass http://backend;
	}
}
```
Save and close the file when you are finished then restart the Nginx service to apply the changes:
```
systemctl restart nginx
```
