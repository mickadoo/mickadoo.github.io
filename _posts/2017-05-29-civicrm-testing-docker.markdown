---
layout: post
title:  "Running CiviCRM in Docker"
date:   2017-05-29 21:40:00 +0100
categories: php,civicrm,docker
---

[CiviCRM][civicrm] is an open-source manager for constituent relations,
mostly used by charities, non-profit organizations and NGOs. I'm getting
started with it, but thought it might be helpful to create a [Dockerfile]
[docker-repo] to get it running locally and set up for tests.

It's supposed to work with Drupal, Joomla and Wordpress, although I've
only ever tried it with Drupal. The community is very active and it's
been my first real dive-in to contributing to an open source project.

One of the first hurdles I faced was getting the development environment
set up. I try and avoid installing stuff on my host machine where 
possible, and since PHP 7 isn't officially supported yet it seemed like a
good opportunity to try setting up an image with PHP 5.6 just for CiviCRM.

It's also very important to be sure before you create any pull requests
on [CiviCRM Core][civicrm-core] that you know the tests are passing and
the code is matching their coding standards (a modified version of 
the [Drupal set][drupal-cs] ). There are already tools for checking your 
changes, like [civilint], which is something similar to PHP code
sniffer.

These tools are included in the Docker image, including everything needed
to run the tests. So instead of having Jenkins do a 30 minute build and 
test I should be able to know whether the tests will pass locally now.

So if you want to just have a look at what CiviCRM looks like locally, or
you want to try running the tests on a pull request, the 
[Dockerfile][docker-repo] should help to get up and running quickly.

There are more instructions in the [readme] but this is all you should
need to get it running locally (assuming you have Docker set up):

```bash
git clone git@github.com:mickadoo/civicrm-docker.git
cd civicrm-docker
docker build -t civicrm-test .
docker run -p 8000:8000 civicrm-test
```

And it should be viewable at [http://localhost:8000][local-build]

[civicrm]: https://civicrm.org/
[civicrm-core]: https://github.com/civicrm/civicrm-core
[drupal-cs]: https://www.drupal.org/docs/develop/standards/coding-standards
[civilint]: https://github.com/civicrm/civicrm-buildkit/blob/master/bin/civilint
[docker-repo]: https://github.com/mickadoo/civicrm-docker
[readme]: https://github.com/mickadoo/civicrm-docker/blob/master/README.md
[local-build]: http://localhost:8000
