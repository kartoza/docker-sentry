## Dockerfile for getsentry.com

Based on crosbymichael/sentry, this is a simple way
to get sentry running under docker easily with postgres. 

# Getting the image

There are several ways to get the image onto your system:

## Use the docker repository:

This will consume the most bandwidth for the initial build but 
will be easy to update thereafter. 

```
docker pull kartoza/sentry
```

## Build with docker build against the github repo


to build the image do:

```
docker build -t kartoza/sentry git://github.com/kartoza/docker-sentry
```

## Build locally 


```
git clone git://github.com/kartoza/docker-django-base
docker build -t kartoza/django-base .
```

# Usage (fig)


Here is a sample fig configuration (save in fig.yml):

```
db:
  image: kartoza/postgis
  volumes:
    - ./pg_sentry/postgres_data:/var/lib/postgresql
  environment:
    - USERNAME=docker
    - PASS=docker

sentry:
  image: kartoza/sentry
  hostname: sentry
  links:
    - db:db

createsuperuser:
  image: kartoza/sentry
  hostname: sentry
  links:
    - db:db
  command: createsuperuser
```

Create a super user:

```
fig run createsuperuser
```

Now run it:

```
fig up -d sentry
```

The running instance will be exposed on port 9000 on the host.

Finally you will probably want to set up a reverse proxy in 
nginx pointing at the sentry container. Here is an 
example configuration for nginx:

```
upstream sentry {
    server 127.0.0.1:9000;
}

server {

    # OTF gzip compression
    gzip on;
    gzip_min_length 860;
    gzip_comp_level 5;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain application/xml application/x-javascript text/xml text/css application/json;
    gzip_disable “MSIE [1-6].(?!.*SV1)”;

    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name sentry.yourhost.com;
    charset     utf-8;

    # max upload size, adjust to taste
    client_max_body_size 15M;

    location / {
        proxy_pass http://sentry;
    }
}

```
