# Comparing databases for news storage

When testing performance, all tests were conducted using the same pre-generated 1.000.000 documents (319.107.649 words) and indexes as they would be in production. Products that didn't support full-text search were not further analysed.

**Result: [Elasticsearch](#elasticsearch) is the clear winner with a good documentation, amazing speed, a perfect feature set, praise from people in the industry, no setup time and lots of tooling support.**

---

#### [MariaDB / MySQL](https://mariadb.org/)

> One of the most popular database servers. Made by the original developers of MySQL. Guaranteed to stay open source.

- 👍 Already have previous knowledge
- 💥 Already made bad experiences, especially using a large number of documents
- 💥 No support for schema-free documents
- 💥 Very limited full-text search features
- 💥 Very slow insert and search speed with propper full-text search indexes

----

#### [PostgreSQL](http://www.postgresql.org/)

> PostgreSQL is a powerful, open source object-relational database system. It has more than 15 years of active development and a proven architecture that has earned it a strong reputation for reliability, data integrity, and correctness.

- 💥 Unintuitive, complex, not very readable queries with new syntax
- 💥 Not usable enough for the scale this project needs
- 💥 No support for schema-free documents
- 💥 *"Is Postgres Search the silver bullet? Probably not if your core business needs revolve around search. [...] Elasticsearch and SOLR offer advanced solutions."*

---

#### [MongoDB](https://www.mongodb.com/)

> MongoDB 3.2 is a giant leap forward that helps organizations standardize on a single, modern database for their new, mission-critical applications.

- 👍 Already have previous knowledge
- 👍 Schema-Free
- 👍 Medium search speed (only tested across 100k documents)
- 💥 Slow insert speed when text indexes were enabled (5min+ for 10.000 documents)
- 💥 Very, very CPU intensive (🔥🔥🔥)
- 💥 Wonky search results with some keywords
- 💥 *"For more advanced and complex search-centric applications, there are alternative solutions like Elasticsearch or SOLR"*

----

#### [Apache Solr](http://lucene.apache.org/solr/)

> Solr is a popular, blazing-fast, open source enterprise search platform

- 💥 Bad tooling support with Node
- 💥 Extremely bad documentation
- 💥 Very expressive syntax (typical Java)
- 💥 Loads of XML configuration files
- 💥 No support for schema-free documents
- 💥 Queries that go further than the standard have to be programmatically created
- 💥 I did not get any articles imported in the hour I tried, so I can't say anything about the import and search speed. But I don't want to deal with a unsatisfactory documented, XML based tool which is hard to Google when things go wrong. The underlaying technology (Lucene) is the same that Elasticsearch uses anyway.

----

#### [Sphinx](http://sphinxsearch.com/)

> Sphinx is an open source full text search server, designed from the ground up with performance, relevance (aka search quality), and integration simplicity in mind.

- 💥 No tooling support with Node
- 💥 Uses MySQL or PostgresSQL under the hood
- 💥 Requires a secondary database, only returns indexes
- 💥 Hard to build search queries across full text and something else ("For much more advanced filtering options [...] I recommend Elasticsearch.")
- 💥 Extremely bad documentation (and bad googling because of pythons `sphinx-doc`)
- 💥 Started as a batch indexer and only moved to real time over time
- 💥 No standard types, no support for schema-free documents

----

#### [Elasticsearch](https://www.elastic.co/de/products/elasticsearch)

> Distributed, scalable, and highly available. Real-time search and analytics capabilities. Sophisticated RESTful API. Real-Time Data. Real-Time Advanced Analytics. Full-Text Search. Document-Oriented. Schema-Free.

- 👍 Wanting to learn that for a while
- 👍 Good documentation
- 👍 Great tooling support with Node (offical!)
- 👍 Schema-Free
- 👍 Fast insert speed (~7s for 10.000 documents, < 1 ms per document)
- 👍 Extremely fast response times (5ms for count, 20ms for FULL-TEXT query)
- 👍 Very good search results (with "match" scores!)
- 👍 Extensive query syntax
- 👍 Gets referenced a lot by industry professionals
- 👍 Native sharding for distributed queries
- 👍 *"If your own app works/thinks in JSON, then go for ES, it thinks in JSON too."*
- 👍 *"If you need a data store that can handle analytical queries in addition to text searching, Elasticsearch is the best choice for that."*
