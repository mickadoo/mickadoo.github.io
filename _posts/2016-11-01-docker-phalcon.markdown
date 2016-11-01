---
layout: post
title:  "Setting up Phalcon using Docker and PHPStorm"
date:   2016-11-01 11:40:00 +0100
categories: php,phalcon,docker,phpstorm
---

I've heard a bit of hype about Phalcon for it's high performance. I'm learning a bit of C now too 
so it seems like a good framework to have a look at since it's written in as an extension for PHP.

I'm using a PHP 7 container (of which I'm sure you can find many). Setting up phalcon and phalcon dev-tools was easy:

```
FROM mickadoo/php7 # or whatever other php7 image you want to use
MAINTAINER michaeldevery@gmail.com
RUN apt-get update

# phalcon
RUN curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.deb.sh | bash
RUN apt-get install -y php7.0-phalcon

# phalcon dev tools
RUN git clone git://github.com/phalcon/phalcon-devtools.git /opt/phalcon
RUN ln -s /opt/phalcon/phalcon.php /usr/bin/phalcon
RUN chmod ugo+x /usr/bin/phalcon
```

I then ran the container with the ports needed for using it as a remote interpreter from PHPStorm. You can see about setting up a remote interpreter 
in [my post on it](https://mickadoo.github.io/php,phpunit,docker/2016/10/12/phpunit-docker.html).

You'll want to run this from whichever directory you want to use for your Phalcon project.

```
#!/usr/bin/env bash

docker run -it \
    -p 22:22 \
    -p 80:80 \
    -p 3306:3306 \
    -v $PWD:/var/www \
    --network dockernet \
    --ip 172.18.0.2 \
    mickadoo/phalcon
```

One thing that was annoying me straight away when I started making the bootloader file was PHPStorm complaining about the missing classes. It seems like you can't
configure it to use remote external libraries but by copying the needed files from the container to the project directory you can get around this.

`docker cp <container-name> /path/to/phalcon-devtools/phalcon/ide/ lib/phalcon`

You can replace "lib/phalcon" with whatever you want or even put it outside your project.
I'd suggest not checking it into git since it's not really part of the project. Then just add that directory by right clicking "External Libraries" in your project pane
and add the new directory in the window from "Configure PHP include paths". Now the warnings should be gone at least.

Then to finally get up and running with Phalcon. Get a shell in your running container using `docker exec -it <container-name> /bin/bash`.
Then from the directory you mounted to (in my case /var/www) run the `phalcon project <name>` command. This should create all the files for a sample project
inside a subdirectory.

To access the site I didn't bother with Apache or Nginx, instead I just used the PHP webserver from inside the project directory:

`php -S 0.0.0.0:80 -t public/`

If all that is set up right you should be able to see the Phalcon welcome message from your host machine on localhost.