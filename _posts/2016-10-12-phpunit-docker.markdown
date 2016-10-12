---
layout: post
title:  "Running PHPUnit tests inside Docker container from PhpStorm"
date:   2016-10-12 18:40:00 +0100
categories: php,phpunit,docker
---

I like using docker containers to keep PHP separate from my host machine which allows for easy switching of version.

I really like PhpStorms quick menu for running tests or scripts so I set it up to work with docker containers.

Things you will need:

- PhpStorm
- A docker container with PHP on it
- A project with PHPUnit tests

First we need to set up a remote interpreter. PHPStorm doesn't support docker's exec function to run remote PHP scripts (yet) so we'll have to use SSH.

If you're making your own Dockerfile install openssh-server and add a user:  

```
RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd \
    && chmod 0755 /var/run/sshd \
    && /usr/sbin/sshd
RUN useradd --create-home --shell /bin/bash --groups sudo php-remote \
    &&  echo "php-remote:php-remote" | chpasswd
```

If it's already on your container image just add the user. Now would be a good time to test SSH. You'll need to map port 22 to make it accessible.

Another handy thing is to make sure your container will have a static ip so you won't have to check it every time. You'll need to create a docker subnet first with:

`docker network create --subnet=172.18.0.0/16 dockernet`

Then run your container:

`docker run -it -v $PWD:/var/www -p 22:22 --net dockernet --ip=172.18.0.2 my-image-name /bin/bash`

Start the ssh service with `service ssh start` and you should be able to connect to it from your host with ssh `php-remote@172.18.0.2` (using the password you created above).

If you don't like the interactivity of starting ssh yourself you could create an image with php-cli that runs sshd.

Alright, alright. Hopefully you have a running container that you can access from SSH. 

Next inside PhpStorm go to "Settings > Languages & Frameworks > PHP" and click on the '...' next to Interpreter.
 
Click the green plus icon to add a new one and select remote. Choose "SSH Credentials" and enter the ip address, username and password. If it all works it should auto detect the version.

Next let's set up the PHPUnit configuration. 

From "Settings > Languages & Frameworks > PHP > PHPUnit" click the plus icon and select "By Remote Interpreter", then choose the interpreter you just set up.

If you're mounting your project in your container set your path mappings.

You're free to chose which method to use for loading PHPUnit here. Personally I like to load it from the composer autoloader since it's easy to change version (and sometimes you don't want to break all your tests for an old project by updating PHPUnit globally).

Last step is to set up the run configuration. From the top right of your PhpStorm window select the little drop-down arrow and choose "edit configurations". 
 
Click the green plus to add a new one and choose PHPUnit. Choose to use an alternative configuration file and point to your phpunit.xml. You can use the path on your host machine here as PhpStorm is clever enough to replace it with the path mappings you chose. 

Give your configuration a fancy name and you should be good to go.