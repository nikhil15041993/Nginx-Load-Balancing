## Configure Nginx as Reverse Proxy for WebSocket

The WebSocket is a protocol which provides a way of creating web applications that supports real-time bi-directional communication between both clients and servers. WebSocket makes it much easier to develop these types of applications. Most modern browsers support WebSocket including Firefox, Internet Explorer, Chrome, Safari, and Opera, and more and more server application frameworks are now supporting WebSocket as well.


For a production environment, where multiple WebSocket servers are needed for getting a good performance and high availability of the website or application, a load balancing layer which understands the WebSocket protocol is required, NGINX supports the use of WebSocket from NGINX version 1.3 and can act as a reverse proxy for doing load balancing of applications.


This example helps in WebSocket implementation built on Node.js
If you don’t have Node.js and npm installed, then run the following command:
```
sudo apt install nodejs npm
```
Node.js is installed as nodejs on Ubuntu and as node on CentOS. The example uses node, so on Ubuntu we need to create a symbolic link from nodejs to node:
```
ln -s /usr/bin/nodejs /usr/local/bin/node
```

To install Websocket, run the following command:
```
sudo npm install ws
```
### Create a file called a server.js with these contents
```
port = 8080
var Msg = '';
var WebSocketServer = require('ws').Server
    , wss = new WebSocketServer({port});
    wss.on('connection', function(ws) {
        ws.on('message', function(message) {
        console.log('Received from client: %s', message);
        ws.send('Server received from client: ' + message);
    });
 });
console.log("Websocket Server started on port " + port);
```
And execute the server with:
```
node server.js
```

### NGINX Configuration
on /etc/nginx/nginx.conf
```
worker_processes 1;
worker_rlimit_nofile 8192;

events {
  worker_connections  1024;
}

http {
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }
 
    upstream websocket {
        server 192.168.2.111:8080;
    }
 
    server {
        listen 8300;
        location / {
            proxy_pass http://websocket;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            proxy_set_header Host $host;
        }
    }
}
```

WS Client
We can now install a websocket client called wscat. I can use the program to connect to the server:
```
npm install wscat -g

wscat --connect ws://192.168.2.111:8300
Connected (press CTRL+C to quit)
> Hai Nikhil!
< Server received from client: Hai Nikhil!
```

on server we can see the below message 
```
node server.js
Websocket Server started on Port 8080
Received from client: Konbanwa!
```
