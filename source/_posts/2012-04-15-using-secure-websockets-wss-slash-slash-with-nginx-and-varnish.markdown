---
layout: post
title: "Using secure websockets (wss://) with nginx and varnish"
date: 2012-04-14 10:17
comments: true
categories:  [socket.io, stunnel, nginx, node, varnish]
---

How to get socket.io communicating over wss://<!-- more -->

## Preamble

The inital joy of working with socket.io and node to pass data between client and server melted away a little bit when I switched from using a regular websocket connection (ws://) to a secure one (wss://). N.B - If you're not familiar with websockets, think of this as the difference between http:// and https://.

Initally, I had varnish listening on port 80, then forwarding traffic to my node server, and this was working fine until I added wss:// into the mix. Nothing was listening on 443, and varnish doesn't know what to do with encrypted traffic anyway. I figured I could set nginx up on 443 to deal with this traffic but it turns out I was wrong...
 
Websockets work by upgrading a regular HTTP/1.0 connection to HTTP/1.1, which means a single connection between the client and server can be kept open, rather than opening a new connection for every request. This is great and all that (think _seriously amazing_ in terms of what you can do with this), but it's fairly well documented that current stable versions of nginx can't handle HTTP/1.1 connections.

So basically, you need to run something else on 443 if you want to use wss://.

Here's what I ended up doing, and a quickstart guide on how to do it yourself:


## Problem (tl;dr)
- nginx doesn't understand HTTP/1.1 (necessary for websockets to work in the first place)
- varnish doesn't understand encrypted traffic
- how to get socket.io communicating over wss:// ?

## Solution
My solution ended up looking like this:

	(:443) --> stunnel --+ 				+--> [socket.io] --> node
		                 | 				| 					   ^
						 +--> varnish --+ 	   				   |
						 | 				| 	   				   |	
	(:80) ---------------+				+--> [static] -----> nginx

Unencrypted traffic comes straight through to varnish, which listens on port 80. Encrypted traffic commes through port 443 to stunnel, which then decrypts it before passing it through to the same instance of varnish. Varnish then either serves content from the cache or chooses the correct backend to respond to the request (nginx for static content or node for websockets). Nginx is configured to serve static content or pass requests through to node if necessary.

## Quickstart
The following instructions assume you're using Amazon Linux. Other types of linux may require further steps and use different commands.

### Stunnel
[Stunnel](http://stunnel.org) allows you to encrypt arbitrary tcp connections inside ssl (so you're going to need an ssl certificate). If you don't have one, you can get a [free certificate here](https://www.startssl.com/). To install stunnel:

	sudo yum install stunnel

You then need to edit the stunnel config file, which is at /etc/stunnel/stunnel.conf:

``` bash /etc/stunnel/stunnel.conf

cert = /path/to/your.crt
key = /path/to/your.key

debug = 5
output = /var/log/stunnel/stunnel.log

[https]
accept = 443
connect = 80
TIMEOUTclose = 0
```

Then launch stunnel:

	sudo stunnel

### Varnish
[Varnish](https://www.varnish-cache.org/) is many things - it caches content, can act as a load balancer and generally it acts to speed things up. To install:

	sudo yum install varnish

I'm not going to get into [configuring varnish](https://www.varnish-cache.org/docs/3.0/tutorial/) in too much depth here since it's a big subject. Here is a simplified version of the relevant parts of my config file though (/etc/varnish/default.vcl):

``` bash /etc/varnish/default.vcl
# you can define multiple backends using the following syntax:

backend node {
	.host = "127.0.0.1";
	.port = "3000";
}

backend nginx {
	.host = "127.0.0.1";
	.port = "81";
}


# incoming traffic gets dealt with inside the following block

sub vcl_recv {

	# choose backend based on request
	if (req.url ~ "\.(js|css|img)$") {
		set req.backend = nginx;
	} else {
		set req.backend = node;
	}

	# pipe websocket connections directly to node app
		if (req.http.Upgrade ~ "(?i)websocket" || req.url ~ "^/socket.io/") {
			return (pipe);
	}

}


# outgoing traffic gets dealt with below

sub vcl_pipe {

	# copy the upgrade header so websocket connections get upgraded to 1.1
	if (req.http.upgrade) {
		set bereq.http.upgrade = req.http.upgrade;
	}

	return (pipe);

}
```

To start varnish listening on port 80, use the following (you can set the cache size by changing 1G to your own value):

	sudo varnishd -f /etc/varnish/default.vcl -s malloc,1G -T 127.0.0.1:2000

To stop varnish, use the following

	sudo pkill varnishd

### Nginx

[Nginx](http://nginx.org/) is a lightweight, super-fast web server. The dev version already has support for HTTP/1.1, but the current stable version does not. Hence this article... To install:

	sudo yum install nginx

Varnish is set up to pass requests for uncached static files through to nginx (listening on port 81) so it can serve them directly (it's insanely fast at doing this), but there are times when it may not be able to handle a request it is passed. For these requests, you can set your node app up to be an upstream server, so for anything nginx can't handle, it can just pass through to node. You can configure this in /etc/nginx/conf.d/default.conf:

``` nginx /etc/nginx/conf.d/default.conf

upstream node {
	server 127.0.0.1:3000;
}

server {
	listen *:81;
	server_name example.com;
	root /srv/www/example.com;
	error_log /var/log/nginx/example.com/error.log info;
	access_log /var/log/nginx/example.com/access.log main;
														
	location = / {
		proxy_pass http://node;
	}
																						
	location / {
		try_files $uri $uri/ @proxy;
	}
				
	location @proxy{
		proxy_pass http://node;
	}
}
```

You can then start nginx using:

	sudo /etc/init.d/nginx start

... and check it's running on the correct port using:

	sudo netstat -anp | grep nginx

## Summary

This set up is working for me now and seems to be serving both secure and non-secure traffic pretty quickly. Hopefully the next version of nginx with support for HTTP/1.1 isn't too far off.



