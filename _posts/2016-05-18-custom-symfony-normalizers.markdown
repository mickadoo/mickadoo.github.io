---
layout: post
title:  "Creating custom tagged normalizers for the Symfony serializer"
date:   2016-05-18 23:40:00 +0100
categories: symfony
---

I used the JMS serializer for any Symfony FOS Rest project I've worked on and it is very flexible. Sometimes you just wanna cut everything back to basics though.

I found that including lots of levels of nested entities in the response body can make things a lot slower and more complicated down the road, so I wanted to keep things minimal, restricting serialized properties to scalars (no related entities!). 

Since Symfony are [putting more effort into developing their serializer component][symfony-blog-post] I thought I'd look into using it. It worked fine until I noticed that \DateTime was showing just as `[]` in the response.

There seems to be a lack of documentation from Symfony about using the serializer as a service. [This blog article][thomas-jarrand-blog] was very informative, but I thought defining a custom serializer service and injecting the normalizers was not what I wanted.

Looking into the service definitions I found that the definition for the default normalizer `serializer.normalizer.object` is tagged with `serializer.normalizer` and a priority of -1000. I made a really simple \DateTime normalizer:

{% highlight php %}
class DateTimeNormalizer implements NormalizerInterface
{
    /**
     * @param \DateTime $object
     * @param null $format
     * @param array $context
     *
     * @return mixed
     */
    public function normalize($object, $format = null, array $context = array())
    {
        return $object->format(\DateTime::ISO8601);
    }

    /**
     * @param mixed $data
     * @param null $format
     *
     * @return bool
     */
    public function supportsNormalization($data, $format = null)
    {
        return $data instanceof \DateTime;
    }
}
{% endhighlight %}

And defined it in `services.yml`:

{% highlight yaml %}
    date_time.normalizer:
        class: MyBundle\Serializer\Normalizer\DateTimeNormalizer
        tags:
            -  { name: serializer.normalizer }
{% endhighlight %}

It worked nicely.

I like the idea of having a global format for \DateTime in the response body. The combination of tagging, priority and `supportsNormalization` method allow a lot of freedom to define normalizers for different types or classes. 

For simple RESTful API projects it might be worth looking into using the Symfony serializer, the JMS option is great but it can be overkill. 

[symfony-blog-post]: http://symfony.com/blog/new-in-symfony-2-7-serialization-groups
[thomas-jarrand-blog]: http://thomas.jarrand.fr/blog/serialization/