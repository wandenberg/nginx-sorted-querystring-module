Nginx Sorted Querystring Module
===============================

A module to order the querystring parameters in a variable to be used as cache key.

Requests like /index.html?b=2&a=1&c=3, /index.html?b=2&c=3&a=1, /index.html?c=3&a=1&b=2, /index.html?c=3&b=2&a=1 will produce the same value for `$sorted_querystring_args` _'a=1&b=2&c=3'_.

It is also possible remove some undesired parameter defining its name with `sorted_querysting_filter_parameter`, like `sorted_querysting_filter_parameter b _ foo;` resulting in a `$sorted_querystring_args` as _'a=1&c=3'_

_This module is not distributed with the Nginx source. See [the installation instructions](#installation)._


Configuration
-------------

An example:

  pid         logs/nginx.pid;
  error_log   logs/nginx-main_error.log debug;

  worker_processes    2;

  events {
    worker_connections  1024;
    #use                 kqueue; # MacOS
    use                 epoll; # Linux
  }

  http {
    default_type    text/plain;

    types {
        text/html   html;
    }

    log_format main  '[$time_local] $host "$request" $request_time s '
                     '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
                     'cache_status: "$upstream_cache_status" args: "$args '
                     'sorted_args: "$sorted_querystring_args" ';

    access_log       logs/nginx-http_access.log;

    proxy_cache_path /tmp/cache levels=1:2 keys_zone=zone:10m inactive=10d max_size=100m;

    server {
      listen          8080;
      server_name     localhost;

      access_log       logs/nginx-http_access.log main;

      location /filtered {
        sorted_querysting_filter_parameter v _ time b;

        proxy_set_header Host "static_files_server";
        proxy_pass http://localhost:8081;

        proxy_cache zone;
        proxy_cache_key "$sorted_querystring_args";
        proxy_cache_valid 200 1m;
      }

      location / {
        proxy_pass http://localhost:8081;

        proxy_cache zone;
        proxy_cache_key "$sorted_querystring_args";
        proxy_cache_valid 200 10m;
      }
    }

    server {
      listen          8081;

      location / {
        return 200 "$args\n";
      }
    }
  }


Variables
---------

* **$sorted_querystring_args** - just list the IP considered as remote IP on the connection


Directives
----------

* **sorted_querystring_filter_parameter** - list parameters to be filtered while using the `$sorted_querystring_args` variable.


<a id="installation"></a>Installation instructions
--------------------------------------------------

[Download Nginx Stable](http://nginx.org/en/download.html) source and uncompress it (ex.: to ../nginx). You must then run ./configure with --add-module pointing to this project as usual. Something in the lines of:

    $ ./configure \
        --add-module=../nginx-sorted-querystring-module \
        --prefix=/home/user/dev-workspace/nginx
    $ make
    $ make install


Running tests
-------------

This project uses [nginx_test_helper](https://github.com/wandenberg/nginx_test_helper) on the test suite. So, after you've installed the module, you can just download the necessary gems:

    $ cd test
    $ bundle install

And run rspec pointing to where your Nginx binary is (default: /usr/local/nginx/sbin/nginx):

    $ NGINX_EXEC=../path/to/my/nginx rspec .


Changelog
---------

This is still a work in progress. Be the change. And take a look on the Changelog file.
