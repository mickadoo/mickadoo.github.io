---
layout: post
title:  "Using Elastic Beanstalk to store Symfony Parameters"
date:   2016-09-21 15:00:00 +0100
categories: symfony,aws
---

You can see an tutorial for getting Elastic Beanstalk up and running [here][eb-tutorial].
 
 I prefer using the EB console to manage my parameters just because it doesn't require 
 any more configuration files or running of scripts if I want to change something there.

I followed Symfony's explanation of [how environment variables are handled][symfony-env-variables].

You can set environment variables in the aws console from the Elastic Bedanstalk area by clicking on the gear next to software configuration.

Symfony will use variables prefixed with `SYMFONY__`. This prefix is stripped and 
the remaining string is converted to lowercase. Then `__` is converted to `.`

This means for a parameter `database.host` you'll need to set an environment variable `SYMFONY__DATABASE__HOST`.

There were some problems I encountered though. 

- `composer install` will create a `parameters.yml` file based on `parameters.yml.dist`
- If you don't define a value for a parameter in the dist file (which of course you don't want to for privacy) the value in `parameters.yml` will be null
- If you don't define a key in the dist file it won't be included in `parameters.yml`
- Symfony gives priority to the values in `parameters.yml` over environment variables
- If you remove the .dist file composer install will fail (The dist file "app/config/parameters.yml.dist" does not exist.)

I had some head scratching here before I realized I was an idiot and that I could just
 prefix the environment variables with something and then use them inside the `parameters.yml.dist`
 
So the result is this:

```
#config.yml
...
doctrine:
    dbal:
        host:     "%database_host%"
...        
```

```
#parameters.yml.dist

parameters:
    database.host: '%env.database.host%'
    ...
```

which generates this on `composer install`:

```
#parameters.yml

parameters:
    database.host: '%env.database.host%'
    ...
```

... and all of which depends on an environment variable `SYMFONY__ENV__DATABASE__HOST` being defined.

Hopefully this will make managing your EB environment easier. 

[eb-tutorial]: http://www.ifdattic.com/how-to-deploy-symfony-application-to-aws-elasticbeanstalk
[symfony-env-variables]: http://symfony.com/doc/current/configuration/external_parameters.html
