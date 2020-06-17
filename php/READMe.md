# PHP

https://hub.docker.com/_/php

## Simple
If you are going to be creating a PHP web application, the Docker Community maintains an image with Debian's Apache httpd in conjunction with PHP (as mod_php) and uses mpm_prefork by default.

```bash
docker build -t phpinfo .
docker run -d -p 8080:80 phpinfo
```
### Start Apache without a Dockerfile
```bash
docker run -d -p 80:80 -v "$PWD":/var/www/html php:7.4-apache
```
- Run from the location containing index.php
- Browse to http://localhost/
- You can make edits on your host and refresh the page to see changes.

## Install Extensions

## With MySQL Database