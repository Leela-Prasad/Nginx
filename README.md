Nginx vs Apache:
In Apache we have pre configured number of threads so that it can handle requests which are equal number of threads that are configured.
But in Nginx we have a single process thread which will take all requests in an asynchronous way and forwards it to the respective back end system and give it back to client.

Since Nginx doesn't have a server side processing its resource usage is less 
where as in Apache it has the server side processing so the resource usage is more.

For serving static content in apache it has to go the thread and process it so that it can be given back to the client.
But in Nginx it will serve the static content from Nginx itself.
So Static Content serving is faster in Nginx.

Nginx can handle High number of Concurrency requests where in Apache it is limited to the number of threads it is configured.

Installation:
Download latest version from https://nginx.org/en/download.html

brew update

./configure

we might get error like
./configure: error: the HTTP rewrite module requires the PCRE library.
for this we need to install PCRE library which is used for regular experession

brew install pcre 
brew install pcre-devel
brew install zlib
brew install zlib-devel
brew install openssl
brew install openssl-devel

Now Run ./configure

./configure --sbin-path=/usr/bin/nginx --conf-path=/usr/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module

Now Configuration of Nginx is completed, so we can compile the source code.
make

make install

Starting Nginx
nginx

stopping nginx
nginx -s stop


Starting, Stopping, Checking status nginx through systemctl
systemctl start nginx
systemctl stop nginx
systemctl status nginx


Configuring nginx as service
copy below script in the location /lib/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/bin/nginx -t
ExecStart=/usr/bin/nginx
ExecReload=/usr/bin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target



===
Then execute below command
systemctl enable nginx

Nginx Terminology:
There are 2 terms
1. Context
2. Directive

context is like a block where directives are defined

    # Prefix Match 
    # anything starts with /greet will be matched
    # eg: /greet, /greeting, /greeting/more
    #location /greet {
    #  return 200 'Hello from NGINX "/greet" location.';
    #}

    # EXACT MATCH
    #location = /greet {
    #  return 200 'Hello from NGINX "/greet" location. - EXACT MATCH';
    #} 

    # REGEX MATCH
    location ~ /greet[0-9] {
      return 200 'Hello from NGINX "/greet" location. - REGEX MATCH';
    }

Preference for location block
1. EXACT Match
2. REGEX Match
3. Prefix Match


There are 2 types of rewrites
return
rewrite

location /logo {
  return 307 "/thumb.png";
}

Here /logo will be redirected /thumb.png
since we use "return" for redirection so the target url will be "localhost/thumb.png" instead of "localhost/logo"

If we dont want to change the target url then we should use "rewrite"
rewrite /logo /thumb.png;

Here the target url will not be changed.
*** and more thing when a rewrite is evaluated then it will run from the start of the file.


rewrite ^/user/(\w+) /greet/$1;
rewrite /greet/john /thumb.png;
Here first /user/john will be redirected to /greet/john
It will execute from the start of the file
so below line will match
rewrite /greet/john /thumb.png;
so it will display thumb.png.


To avoid multiple redirect we have to use "last"
rewrite ^/user/(\w+) /greet/$1 last;
rewrite /greet/john /thumb.png;

so when /user/john then it will be redirected to /greet/john
No more redirects.

try files:
try_files path1 path2 final;
Here path1 path2 are relative to root directive but final is rewrite.

When a specific url is not handled in config file
eg: localhost/notexist

then try files will try to render path1 
if path1 doesn't exist then it will try to render path2
if path2 doesn't exist then it will try to rewrite final


try_files /thumb.png /greet;
so for above example it will try for {root.dir}/thumb.png 
if thumb.png  then it will try for /greet rewrite
Note /greet it is not from root directory

try_files $uri /friendly_404
so if url not found then it will render sorry message


try_files $uri /cat.png /greet /friendly_404;
http://localhost/notfound

this will search {root.dir}/$uri -> (root.dir)/notfound - doesn't exist
will search {root.dir}/cat.png - doesn't exist
will search {root.dir}/greet - doesn't exist - this is relative path to root directory
will do a rewrite /friendly_404 - this will be rendered.


worker_processes
we can run ngnix process on each of the cpu core 
so one core can run one ngnix process

we can specify that
worker_processes 3

Nginx worker will run on 3 cores

worker_processes auto
this will run n number of workers 
n - is number of workers.

Each worker we can define maximum number of connections.
worker_connections 1024;

so total number of connections will be 
n * 1024
where n is number of cores.


Adding Custom Modules
for this we need to rebuild the source

To get the modules we can execute
./configure --help

./configure --sbin-path=/usr/bin/nginx --conf-path=/usr/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --modules-path=/usr/nginx/modules

we may get below error
./configure: error: the HTTP image filter module requires the GD library.

Run below command
brew install libgd


make
make install
nginx -V

static modules will be loaded automatically, but dynamic modules we have to load
load_module modules/ngx_http_image_filter_module.so
