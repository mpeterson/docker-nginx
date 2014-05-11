# mpeterson/nginx
## Features
  * Uses latest nginx available on the [Ubuntu PPA repository](https://launchpad.net/~nginx/+archive/stable).
  * Imports all docker links environmental variables to the nginx main configuration.
  * **Allows to use the docker links environmental variables inside sites configurations (for creating the connection to php5-fpm for example)**.
  * Allow to override or add files to the image when building it.

## Usage
This example considers that you have a [data-only container](http://docs.docker.io/use/working_with_volumes/) and a php container running and linked to an [ambassador](http://docs.docker.io/use/ambassador_pattern_linking/) however it's not needed for it to run correctly.

```bash
$ sudo docker run -d --volumes-from www_data --name httpd --link php_ambassador:php -p 80:80 -p 443:443 -p 127.0.0.1::22 mpeterson/nginx
```

*__Note:__ Notice that since the image is based on [phusion/baseimage-docker](https://github.com/phusion/baseimage-docker) it has a SSH service listening on 22 which in the example above is mapped so we can access the image in the case we wanted to.*

### Volumes
  * ```/etc/nginx/sites-enabled``` contains all nginx configurations to be used to configure the httpd server
  * ```/data``` volume is where your sites should be contained. This path has a symlink as ```/var/www``` to respect standards. Can be overriden via environmental variables.

It is recommended to use the [data-only container](http://docs.docker.io/use/working_with_volumes/) pattern and if you choose to do so then the volumes that it needs to have is ```/etc/nginix/sites-enabled``` and ```/data```.

### Override files
In the case that the user wants to add files or override them in the image it can be done stated on this section. This is particularly useful for example to add a cronjob or add certificates.

Since docker 0.10 removed the command ```docker insert``` the image needs to be built from source.

For this a folder ```overrides/``` inside the path ```image/``` can be created. All files from there will be copied to the root of the system. So, for example, putting the following file ```image/overrides/etc/ssl/certs/cloud.crt``` would result in that file being put on ```/etc/ssl/certs/cloud.crt``` on the final image.

After that just run ```sudo make``` and the image will be created.

### Docker links
The idea was to have a container that only contained nginx and not a full stack to allow flexibility. Because of that there was a need to create a way to reference the links from within the nginx configurations but [that's not possible](http://stackoverflow.com/questions/21866477/nginx-use-environment-variables) so we needed a way to be able to allow such behavior. Because of that a mechanism for this particular docker, it's usage can be examplified with this excerpt:

```nginx
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass $ENV{"PHP_PORT_9000_TCP_ADDR"}:$ENV{"PHP_PORT_9000_TCP_PORT"};
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
```

## Configuration
Configuration options are set by setting environment variables when running the image. This options should be passed to the container using docker
```-e <variable>```. What follows is a table of the supported variables:

Variable     | Function
------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------
DATA_DIR     | Allows to configure where the path that will hold the files. Bear in mind that since the Dockerfile has this hardcoded so it might be neccesary to build from source
