# About

FactQL is a domain specific language for working with Factual's public API. It is similar to SQL... except it's a Lisp.

Here's a FactQL statement that finds restaurants near a lat lon, where each restaurant delivers dinner, sorted by distance:

```clojure
(select restaurants-us  
  (around {:lat 34.06021 :lon -118.4183 :miles 3})
  (where              
    (= :meal_deliver true)
    (= :meal_dinner true))
  (order :$distance)    
  (limit 3))
```

# Installation

FactQL is hosted at [Clojars](http://clojars.org/factql). Add this to your project dependencies:

````clojure
[factql "1.0.0"]
````

Now you can use FactQL on the REPL:

````clojure
(use 'factql.core)
````

Or in your ns declaration:

````clojure
(require [factql.core :as factql])
````

# Authentication

Before your FactQL queries can run, you must provide your Factual API key and secret:

````clojure
(factql! "YOUR-KEY" "YOUR-SECRET")
````

# select

<tt>select</tt> takes a table name and an optional set of clauses, such as a <tt>where</tt> clause. <tt></tt> will run this query against Factual and return a sequence of results hash-maps.

The simplest example:

````clojure
(select places)
````

Another simple example:

````clojure
(select :places (where (like :name "starbucks*")))
````

# where

The <tt>where</tt> clause allows you to specify row filters. Here's a FactQL statement with a tasty <tt>where</tt> clause:

````clojure
; Find restaurants in LA with valid websites and a good rating,
; that won't make me dress too nice:
(select restaurants-us
  (where
    (= :locality "los angeles")
    (not-in :attire ["formal" "smart casual" "business casual"])
    (not-blank :website)
    (>= :rating 2.5)))
````

## supported filter logic

* in
* not-in
* like
* not-like
* search
* blank
* not-blank
* =
* not=
* >
* <
* >=
* <=

# fields

You can specify the fields you want returned using <tt>fields</tt>, like this:

````clojure
(select places
  (fields :name :locality :website))
````

# order

You can order your results, that is sort rows, by using <tt>order</tt>. For example:

````clojure
(select places
  (order :name))
````

Example of ordering by name ascending, locality descending, using explicit :asc and :desc modifiers:

````clojure
(select places
  (order :name:asc :locality:desc))
````

# search (Full Text Search)

Example:

````clojure
(select places
  (search "starbucks"))
````

# limit and offset

You can limit the returned results with <tt>limit</tt>, like:

````clojure
(select restaurants-us (limit 12))
````

You can page through results using <tt>order</tt>, <tt>limit</tt> and <tt>offset</tt>, like:

````clojure
(select places
  (order :name:asc :locality:desc)
  (offset 20)
  (limit 10))
````

# around (Geo Proximity Filter)

You can use <tt>around</tt> to limit your results to be within a geographic radius of a lat lon coordinate. For example:

````clojure
; Find places near Factual:
(select places
  (around {:lat 34.06021 :lon -118.4183 :miles 3}))
````

# Query Composition

FactQL provides support for composing queries. You can define a query without running it, using <tt>select*</tt>. Later you can create new queries based on that query, and run them at anytime.

For example, imagine you want to define a base query that finds U.S. restaurants that have valid owners and telephones:

````clojure
(def base (-> (select* "restaurants-us")
              (where
                (not-blank :owner)
                (not-blank :tel))))
````

Running that query is as easy as using <tt>exec</tt>:

````clojure
(exec base)
````

But you can also define a new query that builds off of <tt>base</tt>. For example, let's define a new query that uses <tt>base</tt> but adds a filter for non-null websites, and also adds a sort based on website:

````clojure
(def websites (-> base
                (where 
                  (not-blank :website))
                  (order :website)))
````

You can run the <tt>websites</tt> query anytime with <tt>exec</tt>:

````clojure
(exec websites)
````

You can go on to create a new query that is is similar to the <tt>websites</tt> query but, say, adds a limit clause:

````clojure
(def a-few-sites (-> websites (limit 3)))
````

And of course, you can run that at anytime with <tt>exec</tt>.

And the previous <tt>base</tt> and <tt>websites</tt> queries are unchanged, so you can continuing building new queries off of them as well.
