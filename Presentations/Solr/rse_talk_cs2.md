class: center, middle, inverse

# Become a Solr Power User: Getting the most out of Solr

## Swithun Crowe

### Research Computing, The Library

### 14 June 2016

---

# What is Solr?

* [Apache Solr](http://lucene.apache.org/solr/) is for indexing and searching documents

* Built on top of the Java [Lucene](http://lucene.apache.org/index.html) search library

* It is a server with clients in many languages, plus a REST-like API

* Output in XML, JSON, CSV

---

# Solr isn't a RDBMS

* Solr documents are flat lists of name/value pairs

* Don't think in SQL

* Don't normalise your data
 * Stuff as much data into each document as you need to retrieve it
 * And even as much as you need to display it

---

# Running queries from the command line

```bash
links -source
"http://localhost:8983/solr/mycore/select?q=text:foo AND -text:bar"
| xmllint --format - | less```

* Use `links` text-only browser

* Format XML output using `xmllint`

* Pipe to `less`

---

# Sample query

```
?fl=root.link,root.link.selected,root.ara,root.trans
&sort=root.ara.sortable+asc
&q=root.trans:W* AND public:public
&group=true&group.field=root.ara.sortable
&start=0&rows=10000```

* `fl` specifies the field list - the fields one wants returned
* `sort` specifies the field/s on which to sort, and in which direction
* `q` is the main query
* `group.field` specifies on which field to group results (requires
`group=true`)
* `start` specifies the offset and `rows` the number of hits to get

---

# Getting data into Solr

* Can pull in data from RDMBSs - each row in the output becomes a Solr document
* Via client in your favourite language
* POST XML documents to Solr using REST-like API

```bash
URL=http://localhost:8983/solr/bioreg/update
for f in $*; do
	curl -s $URL --data-binary @$f -H 'Content-type:application/xml'
done
curl -s "$URL?softCommit=true"
```

```xml
<add>
  <doc>
    <field name="id">1</field>
    <field name="text">Some text</field> ...
  </doc> ...
</add>
```

---

# Schema

## Field types

* Basic ones for integers, floats, dates etc.
* Text fields can have stop words, (language specific) stemming, case folding, ASCII-folding
* Different tokenisers and filters can be applied at index time and at
query time
* Write Java code for your own tokenisers and filters (e.g. to strip vowels from Arabic)

## Fields

* Field name - can have wild cards
* Field type
* Should field be indexed, stored or multi-valued?

---

# Schema

## Indexing

* Fields that you want to search

## Storing

* Fields that you want returned
* Can put HTML in these fields, to help generate a page of results

## Copying fields

* Can copy one field into another one
* Can have a default field (e.g. `text`) which has everything in it,
for simple searches, and more specific fields for advanced searches

---

# Facets

* Facets show you how search results break down by different values for fields
* Can facet on discrete values and on date and number ranges
* Solr gives you counts for how many documents share a field value

```
?facet=true&facet.mincount=1  
&facet.date=work_date_created_start  
&f.work_date_created_start.facet.date.gap=+50YEARS  
&f.work_date_created_start.facet.date.end=2100-01-01T00:00:00Z  
&facet.field=doc_type&facet.field=auth_type```

---

# Joining

* Not like a SQL join
* Specify a query, a `from` field and a `to` field
* Results will be documents where their `to` field value matches the `from` field value in documents matched by the query

```
?work=(text:Rumi)  
&q=(_query_:"{!join from=work_parent_id to=ms_id v=$work}") OR 
   ((text:Rumi))```

---

# Think Solr might be useful?

* Then contact Research Computing

* res-comp@st-andrews.ac.uk

* We can offer advice and host your index
