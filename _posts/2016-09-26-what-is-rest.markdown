---
layout: post
title:  "What is REST"
date:   2016-09-26 23:00:00 +0100
categories: rest
---

There comes a time for all developers trying to build a RESTful API where they ask themselves "What the hell is REST anyway?". For most of them it is probably in the early stages,
before actually building anything. For me it came a few months after finished a job where I spent 18 months trying to build a RESTful API. 

I don't think I'm alone in my confusion, the second highest question on stack overflow about REST is ["What exactly is RESTful programming?"][so-what-is-rest]. After going through his full dissertation I 
was surprised by its content. Here I was expecting to have someone to finally resolve the questions about POST VS PUT ([top SO question with REST tag][so-put-vs-post]) and when to use a PATCH request.

Instead I found a paper mainly focused on software architecture. Indeed, if you run a word frequency count on it, the most used word (excluding 'and', 'is' etc.) is "architectural", followed 
 closely by "architecture". 

#### Word frequency in Roy Fielding's dissertation: 
 1. 282 architectural
 2. 263 architecture
 3. 187 style
 4. 177 data
 5. 175 software

So I thought I'd go through the paper in full. I made a set of notes as I went along, what follows is a mish-mash of actual quotes from it, a lot of paraphrasing, 
some explanation and a minimal amount of opinion.

## Software Architecture Terms

Much of the dissertation discusses software architecture. Software architecture deals with [what is done, where it's done but never with how][so-software-architecture].
Rest is an architectural style. An architectural style is a set of architectural constraints. 

REST is a hybrid style. It is designed to be efficient for large-grain hypermedia data transfer (the common case of the web).

The following terms are used in to describe software software architecture:

#### Components

_A component is defined by its interface and the service it provides to other components._

- origin server (e.g. apache)
- gateway (cgi, reverse proxy)
- proxy
- user agent (browser)  

#### Connectors

_Connectors enable communication between components by transferring data elements from one interface to another without changing the data._

- Client
- Server
- Cache
- Resolver (DNS lookup)
- Tunnel (SSL)

#### Data

_A datum is an element of information that is transferred from or to a component._

- Resource: conceptual target of hypertext reference
- Resource identifier: url/urn
- Representation: html doc/ jpeg image
- Representation metadata: media type, last modified
- Resource metadata: source link, alternates
- Control data: if-modified-since, cache-control

## Distributed Hypermedia

REST was developer to support the "distributed hypermedia" nature of the web. Distributed hypermedia provides a means of accessing services by 
embedding actions in the information retrieved from sites (e.g. clicking on a link). The interactions of these systems consist of [large-grain][wiki-granularity] 
data transfers rather than computation intensive tasks.

## What is REST?

Representational state transfer (REST) is supposed to evoke how a web-app works - the app being a network of web-pages (state machine), 
which has a user click through them (**state** transitions) to show the next page (**representation** of next page of application state).

REST is a coordinated set of architectural constraints that attempts to minimize latency and network communication 
while at the same time maximizing the independence and scalability of component implementations, enforce security and encapsulate legacy systems.

As one of the authors of the HTTP 1.1 spec, Roy Fielding used REST to guide the design and deployment of the web architecture.

REST components communicate by transferring a representation of a resource in a format matching one of the standard data types. The 
representation can be the same format as the raw source or derived from the source. The format of a representation can be in a 
variety of media types, some are for automated processing (e.g. JSON), some for human reading (e.g. text).

REST does not restrict communication to a particular protocol but it does constrain the interface between components.

REST style is usually pulling representations, as push model (pushing from server to client) is infeasible on the web.

Roy defines the in-parameters and out-parameters, so if REST was a function it would have this form:

```
/**
 * @param $controlData     What do do with the resource
 * @param $identifier      Which resource is it
 * @param $representation  Optional representation of the desired state
 *
 * @return array
 */
function ($controlData, $identifier, $representation = null) {

    // do something

    return [
        'controlData' => $controlData,
        'metadata' => $metadata, // about the resource
        'representation' => $representation // (optional) updated representation 
    ];
}
```

Control data defines the purpose of a message, such as the action or overriding the default behavior (e.g. caching).
Depending on control data, the representation may indicate the current state of the requested resource, the desired state or 
the value of another resource.

These are the interface constraints of REST:

- Identification of resources
- Manipulation of resources through representations
- Self descriptive messages
- Hypermedia as the Engine of Application State ([HATEOAS][hateoas])

These are the architectural properties of REST:

- Performance 
- Scalability 
- Simplicity
- Modifiability
- Visibility
- Portability
- Reliability

Modifiability includes evolvability, extensibility, customizability, configurability and reusability.

Visibility refers to the ability of a component to monitor or mediate the interaction between two other components.  

Scalability can be improved by simplifying components, distributing services across many components and by controlling interactions. The 
web has to allow for anarchic scalability, clients can't know about servers, servers can't keep state, and clients are not trustworthy.

These are the architectural constraints of REST:

- Client-server
- Stateless
- Caching
- Uniform interface
- Layered system
- Allows code-on-demand (optional)

## HTTP

Since he was involved in authoring the HTTP spec, it's not surprising that the dissertation includes some overlap with the HTTP spec. 

Some of the rules mentioned include:

- GET and HEAD are the only methods for which conditional header field mean cache refresh, for all others they are a precondition.
- Requests must include the host field (as per HTTP 1.1 spec).
- There is no limit on the length of the uri, header fields, representation or any field value.
- HTTP method extension is allowed whenever a standarizable set of semantics can be shared between client and server and any intermediaries between them.

I've heard some talk recently that RPC might be a good alternative to REST. I think Mr. Fielding didn't regard RPC as viable for use on the web because RPC 
doesn't use a generic interface with standard semantics, so it's harder to be interpreted by intermediaries.

## Caching

Caching is a mentioned quite often and it goes back to improving performance, where the best application performance for 
network apps is obtained by not using the network.

Caching improves efficiency, scalability, user-perceived performance, but reduces reliability of data (because you can get stale data).

- Cache constraints require that the data be implicitly or explicitly labeled as cacheable or non-cacheable.
- A response to a retrieval request (GET) is cacheable and other responses are not.
- If authentication is used or if the response should not be shared then it's only cacheable in a non-shared cache.
- The "Vary" header lists the request header fields under which the response may vary.
- The 'no-cache' directive can be used to get authoritative response from the origin server.

One form of abuse of the URI is to include information that identifies the current user which breaks shared caching.
I've also seen examples of this where the response for the same URI is dependent wholly on the current user (from auth token), which makes caching difficult. 

## State(lessness)

REST really emphasizes statelessness. All REST interactions are stateless - Statelessness is good because connectors 
don't have to care about state, interactions can be done in parallel, it's easy to understand a request in isolation and it's good for caching. It
has the drawback that it increases overload (e.g. repetitive data on each request).

Each request from client to server must contain all the information necessary to understand the request and cannot take advantage of any stored context on the server.
Cookie interaction fails to match REST's model of application state. Frames also fail because they can display one state wedged in the sub-context of another. 

In REST an applications state is defined by pending requests and is only steady when it has no outstanding requests.

## Generality of Interface

The central feature that distinguishes REST from other network styles is its emphasis on uniform interface between components.
By applying the principle of generality to component interface, the overall software architecture is simplified and visibility of interactions is improved.
The trade-off is that uniform interface degrades efficiency - information is transferred in standardized form and not one specific to application's needs.

## Layeredness

Layers can exist between the client and server. He compares the layered system, in combination with the uniform interface, to the uniform pipe 
and filter style often seen in linux, whereby the input is piped across multiple filters to produce an output.

Layers in REST can actively transform the content of messages. They can also use intermediaries to improve system scalability 
by enabling load balancing of services across multiple networks. Layered systems add overhead and latency, reducing 
user-perceived performance.
 
Layered systems can also be used for encapsulating legacy services (hiding them behind a RESTful interface).

## Code on Demand

REST allows client functionality to be extended by downloading and executing code in the form of applets or scripts.
Allowing features to be downloaded after deployment improves system extensibility, but reduces visibility so this constraint is optional.

Code on demand could be used in cases where you could download Java applets or Javascript. He goes on to start a flame war by saying that 
Javascript beat Java because of its low entry barrier, greater visibility (java is a packaged binary), 
better user-perceived latency and the security issues of Java.

## Resources and Identifiers

Any information that can be named can be a resource. A resource is just a conceptual mapping. 

A resource can map to an empty set.

A resource is accessed using an identifier. REST relies on the author choosing an identifier that best fits the nature of the concept. 
Identifiers should change as infrequently as possible.

When a request maps to multiple representations on the server, the server may engage with negotiation with the client or select a representation 
that best fits the client.

## Metadata

Metadata can be included in the form of name-value pairs. Responses can include both representation metadata and resource metadata.

Header field names can be extended, but only when the information they contain is not required to understand the message.

Hypermedia data cannot retain "back pointers" to sources of data that references it, for example a page containing link headers to all other pages that link to it.  

## Conclusion

I thought the REST dissertation would proscribe some good practices for building a RESTful API. Instead what I found was closer to a recipe for building the internet.
There are almost no mentions of HTTP verbs. There is no mention of nouns as resources.

But what does it mean for the design of RESTful APIs, the area I'm most interested in? Well as I've mentioned already I don't think the dissertation was ever intended to address 
a lot of the common issues that I see thrown around with the REST tag. A lot of these questions probably relate more to the HTTP spec, specifically the proper use of HTTP verbs. 

The HTTP part of the dissertation focused on applications that provide content to the user, at one point saying that "write actions using the web are extremely rare". 
I think therefore if you're designing a web app that  processes a lot of user input for the generation and altering of resources the dissertation was 
not written with you in mind.

That's not to say it's not useful for application developers. There are many good practices listed here for creating web apps; proper use of caching, using metadata 
and stateless transactions.
 
It was written quite a long time ago in tech terms. It was published in 2000. For context Google was founded in 1998 and Chrome 
 was released in 2008. Since then an updated HTTP version has been released. I think the dissertation is a great insight into how one of the co-authors 
 of the HTTP spec envisioned web applications would function. But I wouldn't treat it as a bible. The environment has changed since it was written and to be oblivious to that 
would only be a needless limiting factor for your application.

I do wonder why Mr. Fielding, who had such an impact on the early web (with HTTP1.1, libwww-perl and Apache) wasn't more involved in HTTP2. His Wikipedia page says 
 he worked on Waka, a protocol intended to replace HTTP, which was presumably made defunct with the introduction of HTTP2. I've seen some suggestions 
 he made in the HTTP2 mailing list but they seemed to be disregarded. It seems a shame someone with such experience on the protocol that powered the web for so long was not more 
 involved in crafting its successor.

[wiki-granularity]: https://en.wikipedia.org/wiki/Granularity#Computing
[hateoas]: https://en.wikipedia.org/wiki/HATEOAS
[so-what-is-rest]: http://stackoverflow.com/questions/671118/what-exactly-is-restful-programming
[so-put-vs-post]: http://stackoverflow.com/questions/630453/put-vs-post-in-rest
[so-software-architecture]: http://stackoverflow.com/a/704909/1196369
