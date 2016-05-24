---
layout: post
title:  "Comparing search options in Symfony3"
date:   2016-05-24 02:54:00 +0100
categories: symfony
---

I've heard a lot of hype about Elasticsearch although I've never used it in a project. With a bit of time to spare I 
thought I'd try it out, and why not do a bit of a performance comparison while I'm at it. And sure if I'm doing a 
simple comparison why not also include one of the other options, Solr. Shouldn't take too long to do a simple 
check, right?

Three days later...

I could have just stuck with Doctrine. I don't even have a serious project for this in mind, and ,as I read somewhere 
along the line, for 90% Elasticsearch is overkill. Still, once I'd started I couldn't stop.

#### The Test

My idea was to compare `LIKE` using Doctrine , `MATCH AGAINST` using Doctrine, `Elasticsearch` using the 
[FriendsOfSymfony/FOSElasticaBundle][fos-elastic-bundle] and `Solr` using [floriansemm/SolrBundle][solr-bundle]

I used docker-compose to set up a Symfony app with linked containers for PHP 7.0.2, MySQL 5.7.10, 
Elasticsearch 1.7.5 (currently the elastic search bundle isn't compatible with ^2.0) and Solr v6.0.0
 
I wanted to test the search of a story entity, which has a title and text. I populated the database with random stories. 
The title was 2 words long and the text was 200 random words long. I tested with 500, 5,000 and 50,000 records. 
 
I wanted to be able (if necessary) to do a fuzzy search. I included one story with the words 'text' and 'angel' 
and wanted to check if a search for 'tetx' or 'angkl' would return it.

Another search was checking if selected words from a sentence would match a record containing the full sentence, e.g. 
by searching for 'if words day golden' could return a story with the text 'if i had words to make a day for you, 
I'd sing you a morning golden and true'.
 
I got the average of 3 response times (excluding some trial requests to warm up the cache). To try to measure the 
 time of the search itself more accurately I added a timer around a `doSearch` method and logged the time taken. It was easier
 then to just change the behaviour of the `doSearch` method.
  
Since I didn't want to take entity population into account the search only returns a hit count each time. 
 
#### The Results

*The time is the average total response time (in milliseconds) over three requests. The time in parentheses is the time 
taken to perform the search.*
 
| Scenario        | LIKE | MATCH AGAINST | Elasticsearch | Solr |
| --------------- |:-------------:|:---------------------:|:-------------:|:----:|
| single word search against 500  | 132 (21) | 126 (16) | 118 (4) | 128 (14)
| sentence search against 500  | 129 (19) | 132 (16) | 121 (4) | - |
| single word search against 5000 | 162 (52) | 130 (17) | 118 (5) | 137 (14) |
| sentence search against 5000 | 133 (25) | 129 (16) | 120 (4) | - |
| single word search against 50000 | 399 (289) | 168 (26) | 120 (4) | - |
| sentence search against 50000 | 171 (62) | 185 (26) | 123 (6) | - |

<br/>

#### Matching most likely result from multiple word input

- Doctrine LIKE works here when I tokenized the words but is not sorted by relevancy.
- MATCH AGAINST also worked here and includes a scoring which correctly identified the target story as result #1
- Elasticsearch got the correct record when using the default search, but not when using fuzzy search. Not sure why
- I couldn't get this working with Solr

#### Fuzzy Search

- Doctrine LIKE (as expected) did not match "tetx" with "text" or any other misspelling 
- Same for MATCH AGAINST, didn't work
- Elasticsearch correctly identified 'angkl' as matching 'angel' in a fuzzy search, but did not match 'tetx' with 'text'
I assume this is related to the Levenshtein distance and can be configured.
- I couldn't manage to get fuzzy search working with the Solr bundle and gave up after an hour 

#### Conclusion

I think from the results it's pretty clear that Elasticsearch performs better than any of the rest. There's not much
difference for small result sets, but for the larger ones MySQL starts to lag a bit.

I don't have many results for Solr. This was because I had some trouble finding out how to use the bundle and couldn't
 get it to match any word or get fuzzy search working. I'm sure it's possible, I've even seen it in the Solr 
 documentation but I just couldn't figure out how to do it using the Symfony bundle and didn't want to put too much 
 time into it.
 
I think the Elasticsearch bundle was a lot more usable and I had it set up and running in no time. There also seems
to be a lot more resources available on using it with Symfony.

Another problem with Solr was the indexing time. I could populate the Elasticsearch indices for 500 records in about
1 second. Solr indexing took 41 seconds for 500, so I didn't even try index the 50,000 records.

One nice feature of Solr was support for mapping by annotation. I think the same thing might be possible using 
 the serializer annotation with the Elasticsearch bundle, which would be great - just adding a serializer group instead
 of adding completely new annotations.
 
It might not be so cool to use MySQL but I think that in a lot of cases it is adequate. The simple LIKE search is usable 
 as-is, no new bundles or changes to your code at all. The `MATCH AGAINST` will require altering your table to add a new
 index for each field combination you want to match against. I had to run this directly in the MySQL console which sets
 off some alarm bells. If you're using an DBAL then I guess you shouldn't be required to directly update the database to
 get things working, but there you go ¯\\\_(ツ)_/¯
 
Another good thing about basic MySQL that you don't see in the test results is improved performance on create/update 
actions. When you're adding or editing an entity using Elasticsearch or Solr there are event listeners that will update 
the index, but this will of course add some overhead. I noticed that populating test data took a lot longer after 
 enabling Elasticsearch or Solr.
 
I'd heard it was fast, but still I was pretty surprised by the speed of Elasticsearch. It didn't seem to fluctuate much
 at all for larger result sets. The documentation is good and it seems very very flexible so I think I might play around
 with it a little more.
 
[fos-elastic-bundle]: https://github.com/FriendsOfSymfony/FOSElasticaBundle
[solr-bundle]: https://github.com/floriansemm/SolrBundle