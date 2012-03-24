## Rucene
Jotting down an idea before I forget. I've used Lucene in the past for
searches (http://www.pcapr.net) and I absolutely <3 it. But some of the blogs
on repurposing Redis for free-text search is too off - storing all prefixes is
such a kludge. So question is, can we do something better?

## Idea
This is in a very raw form. Use a simple tokenizer and a bunch of stop words
to convert the input document into a set of terms. For each term, compute `tf`
which is the term frequency within the document and store that in a sorted
set. The `docid` is a simple opaque document identifier.

    <term1>: zset<docid, tf(term in doc)>
    <term2>: zset<docid, tf(term in doc)>

During query time, compute `idf` which is the number of documents that contain
the term (zcard for the term really). Lucene does a lot more than that with
boosts and what not.

    weight = idf(t)^2
    idf = no of documents that contain term (zcard for the term)

So when we do the searching, simply intersect the terms of the query, do the
per-term weighting as above and then aggregate the sum. This effectively gives
a set of docids, sorted by the relevance. Simplistic, but works.

    zinterstore dst num_terms termi ... termk weights wi ... wk aggregate sum

Code to come soon.
