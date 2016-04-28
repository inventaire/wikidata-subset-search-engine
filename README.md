# Wikidata Subset Search Engine

Tools to setup an ElasticSearch instance fed with subsets of Wikidata, to answer questions like *"give me all the humans with a name starting by xxx"* in a super snappy way, typically for the needs of an autocomplete field.

Powering [data.inventaire.io](https://data.inventaire.io), and tailored for [inventaire](http://github.com/inventaire/inventaire)'s needs, but could probably be adapted to other use cases

## Setup

### Dependencies
see [setup](./SETUP.md) to install dependencies:
* [NodeJs](https://en.wikipedia.org/wiki/Nodejs)
* [ElasticSearch](https://en.wikipedia.org/wiki/Elasticsearch)
* [Nginx](https://en.wikipedia.org/wiki/Nginx)
* [Let's Encrypt](https://en.wikipedia.org/wiki/Let's_Encrypt)
* already installed in any good **nix* system: curl, gzip

### Wikidata filtered-dump import

#### import a filtered Wikidata dump into ElasticSearch
```sh
# the wikidata claim that entities have to match to be in the subset
claim=P31:Q5
# the type that will be passed to ElasticSearch 'wikidata' index
datatype=humans

./bin/dump_wikidata_subset $claim $datatype
# time for a coffee!
```
What happens here:
* we download the latest [Wikidata dump](https://www.wikidata.org/wiki/Wikidata:Database_download#JSON_dumps_.28recommended.29)
* pipe it to [wikidata-filter](https://github.com/maxlath/wikidata-filter) to keep only entities matching the claim `P31:Q5` and keeping only the entities attributes required by a full-text search engine, that is: `id`, `labels`, `aliases`, `descriptions`
* pipe those filtered entities to ElasticSearch `wikidata` index under the datatype `humans`, making those entities searchable from the endpoint `http://localhost:9200/wikidata/humans/_search` (see [ElasticSearch API doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html))

:warning: *you are about to download a whole Wikidata dump that is something like 7GB compressed. Only the filtered output should be written to your disk though.*

#### import multiple Wikidata subsets into ElasticSearch
The same as the above but saving the Wikdiata dump to disk to avoid downloading 7GB multiple times when one time would be enough. This time, you do need the 7GB disk space, plus the space that will take your subsets in ElasticSearch
```sh
alias wdfilter=./node_modules/wikidata-filter/bin/wikidata-filter
alias import_to_elastic=./bin/import_to_elasticsearch

curl -s https://dumps.wikimedia.org/wikidatawiki/entities/latest-all.json.gz > wikidata-dump.json.gz

cat wikidata-dump.json.gz | gzip -d | wdfilter --claim P31:Q5 --omit type,claims,sitelinks | import_to_elastic humans
# => will be available at http://localhost:9200/wikidata/humans

cat wikidata-dump.json.gz | gzip -d | wdfilter --claim P31:Q571 --omit type,claims,sitelinks | import_to_elastic books
# => will be available at http://localhost:9200/wikidata/books
```

## Query ElasticSearch

```sh
curl "http://localhost:9200/wikidata/humans/_search?q=Victor%20Hugo"
```
or try the result on data.inventaire.io
```sh
curl "https://data.inventaire.io/wikidata/humans/_search?q=Victor%20Hugo"
```

## data.inventaire.io
Whitelisted endpoints:
* [`/wikidata/humans/_search`](https://data.inventaire.io/wikidata/humans/_search?q=Victor)

## References
* [ElasticSearch Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)
