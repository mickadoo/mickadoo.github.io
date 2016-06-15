---
layout: post
title:  "Elasticsearch in Symfony3 using the Symfony serializer"
date:   2016-06-15 15:00:00 +0100
categories: symfony
---

As of writing this the [fos-elastic-bundle] is not supporting Symfony 3.1 so 
I'm using dev-master (hopefully FOS will tag new version soon).

I'm still waiting for the FOS crew to merge my [pull request][serializer-pr] 
allowing you to use the Symfony serializer. I could suggest using my fork but 
who wants to use a fork that 100% won't be maintained amirite?? If you really 
can't wait you can get on github and ask them to work on closing it.

That means until that PR is merged no using serializer groups as described here 
with the Symfony Serializer (JMS only).

I think the JMS Serializer is very powerful, but for my projects I wanted to 
make things nice and simple (and fast). Elasticsearch is blazingly fast and 
I was hoping that I could use the Symfony serializer with it.

Configuring all the mapping for each entity is a lot of bother and hard to 
maintain. When I saw support for serializer groups I thought it would be much 
easier.

When you include the simplification in configuration from my 
[last pull request][config-pr] configuration changes from: 

```yaml
# config.yml
fos_elastica:
    clients:
        default: { host: localhost, port: 9200 }
    indexes:
        app:
            types:
                story:
                    mappings:
                        id: ~
                        title: ~
                        createdAt: ~
                        sentenceCount: ~
                        blah: ~
                        blah: ~
                        anotherProperty: ~
                    persistence:
                        driver: orm
                        model: YarnyardBundle\Entity\Story
                        provider: ~
                        listener:
                            immediate: ~
                        finder: ~
```

to:

```yaml
# config.yml
fos_elastica:
    clients:
        default: { host: localhost, port: 9200 }
    serializer: ~
    indexes:
        app:
            types:
                story:
                    serializer:
                        groups: [elastica]
                    persistence:
                        model: YarnyardBundle\Entity\Story
```

And inside your entity class just add the serializer groups to the properties / methods:

```php

use Symfony\Component\Serializer\Annotation\Groups;

class Story
{
    /**
     * @Groups({"elastica"})
     *
     * @var string
     */
    private $title;
```    

Nice and simple, right?
 
[fos-elastic-bundle]: https://github.com/FriendsOfSymfony/FOSElasticaBundle
[serializer-pr]: https://github.com/FriendsOfSymfony/FOSElasticaBundle/pull/1086
[config-pr]: https://github.com/FriendsOfSymfony/FOSElasticaBundle/pull/1084
