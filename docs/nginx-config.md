# NGINX config

Below is the base config used to config NGINX running on Alpine. You could create a docker config for all NGINX deployments using the config below. In fact, all the Docker stacks that utilize NGINX in this repo use the same base config of `conf/nginx/nginx.conf`.

The benefits of this are:
* Specifying default good security values like using TLS v1.2+, only accepting certain ciphers
* The `ssl_certificate`, `ssl_certificate_key`, `ssl_dhparam` could be replaced with values to Docker secrets
* This config will load any NGINX config added to `/etc/nginx/conf.d/*.conf`

```
user nginx;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 1024;
}

http {
	##
	# Basic Settings
	##
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##
	ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
	ssl_ecdh_curve secp384r1; 
	
	# SSL cert
	ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;	

	ssl_session_timeout  10m;
	ssl_session_cache shared:SSL:10m;
	ssl_session_tickets off; 
	ssl_stapling on; 
	ssl_stapling_verify on; 

	add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
	add_header X-Frame-Options DENY;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";
   

	##
	# Logging Settings
	##
	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##
	gzip on;
	gzip_disable "msie6";

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
}
```