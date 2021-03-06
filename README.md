# ElasticQuery v0.2

A _simple_ query builder for Elasticsearch. Outputs JSON ready to be sent to Elasticsearch via your favourite client.

+ [Example](#example)
+ [API](#api-elasticquery)
	- [ElasticQuery](#api-elasticquery)
	- [Filter](#filter)
	- [Query](#query)
	- [Aggregate](#aggregate)
+ [Internals](#internals)


## Example

	from elasticquery import ElasticQuery, Filter, Query, Aggregate
	
	query = ElasticQuery()
	
	# Filter
	query.must(Filter.range('price', 0, 500))
	
	# Query
	query.should(Query.nested('nested_key', musts=[
		Query.terms(nested_key_key=['value']),
		Query.prefix(another_key='prefixed_with_')
	]))
	
	# Aggregate
	query.aggregate('key_terms', Aggregate.terms('key'))
	
	json = query.compile()


## API: ElasticQuery

`query = ElasticQuery()`

### query.compile()

Return a json string ready to send to Elasticsearch.

### query.fields(fields)

**fields**: list of fields to return

### query.sort(field, order=False)

Sort the result set by a field, order optional.

### query.must(Query/Filter)

Search where the `Query`/`Filter` object matches.

### query.should(Query/Filter)

Search where the `Query`/`Filter` object might matches.

### query.must_not(Query/Filter)

Search where the `Query`/`Filter` object does not match.

### query.aggregate(name, Aggregate)

Add an `Aggregate` to our search query.


## Filter

### Filter.nested(path, musts=[], shoulds=[], must_nots=[])

Adds a nested query.

**musts, shoulds & must_nots**: all lists containing `Query` or `Filter` objects.

### Filter.range(field, gt=None, gte=None, lt=None, lte=None)

### Filter.prefix(**kwargs)

Prefix multiple keys.

### Filter.term(**kwargs)

Search multiple key=>value terms.

### Filter.terms(**kwargs)

Search multiple key=>[values] terms.

### Filter.missing(field)

Filter for missing/null fields.

### Filter.raw_string(string, default_operator='AND')

Adds a raw query string to match against.

### Filter.string(default_operator='AND', **kwargs)

This builds a query string based on `kwargs`. Values can be simple (ints/strings) or lists, in which case they are `OR`'d together.

### Filter.or_filter(*args)

Or the arg filters/queries together.


## Query

The `Query` class inherits from `Filter`, see above for API details.

### Query.mlt(field, match, min_term_frequency=1, max_query_terms=False)

Searches objects where field is similar to the match.


## Aggregate

### Aggregate.sub(aggregate, **aggregates)

Puts **aggregates as sub-aggregats under aggregate.

### Aggregate.sum(field)

Get sum of a field.

### Aggregate.avg(field)

Get the average value across a field.

### Aggregate.min(field)

Get the lowest value of a field.

### Aggregate.max(field)

Get the highest value of a field.

### Aggregate.stats(field)

Get stats on a field.

### Aggregate.extended_stats(field)

Get extended stats on a field.

### Aggregate.missing(field)

Count how many documents are missing a given field.

### Aggregate.value_count(field)

Count how many documents contain a given field.

### Aggregate.histogram(field, interval)

Generate a histogram.

### Aggregate.date_histogram(field, interval='day')

Generate a date histogram.

### Aggregate.terms(field)

Count number of terms on a field.

### Aggregate.nested(path)

Create a nested aggregation (for use with sub aggregates, see Aggregate.sub).

### Aggregate.filter(musts=[], shoulds=[], must_nots=[])

Creates a filtered aggregate (for use with sub aggregates, see Aggregate.sub).


## Internals

`ElasticQuery` contains `.structure` which represents the final output. `Filter.*`, `Query.*` and `Aggregate.*` functions are essentially structs, each builds from its input and returns a dict representing the relevant json structure Elasticsearch wants.

Although there's multiple ways of doing most query/filter types in ES (ie `query['range'] = { 'to': 500 '}`), `ElasticQuery` stores everything inside a bool query. I haven't seen any performance degredation as a result makes everything a lot simpler. This way queries, filters and nested queries/filters all have a combination none or more of `must`, `should` and `must_not` matches.
