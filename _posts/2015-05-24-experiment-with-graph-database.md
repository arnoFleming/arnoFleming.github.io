---
layout: post
title: Experiment with a graph database
baseline: Just for the fun of it

---

#{{ page.title }}
_{{ page.baseline}}_

I work for a [job agency](https://www.youngcapital.nl). Most of my colleges are 
recruiters. To support their daily work, we have an app that does some extensive
searching through our RDBM. 
We also have an Elastic Search cluster so we can search for specific keywords in our candidates' resumes, and do other types of tropical matching.

## Matching candidates a different way
One of our devs wanted to learn more about graph databases, and last friday we had ourselves a hackathon.

The question we started with was: How can a graph database help us in that process?

## Getting up and running
We chose [Rails] as deliver platform, because we only had the better part of one day to build an MVP, and we write most of out software in it.

We chose [Neo4j] as graph database, because of some peculiar reasoning: 

1. We need a graph database 
2. Neo4j is a graph database 
3. We need Neo4j 

And because it is the most popular graph database according to 
[some website](http://db-engines.com/en/ranking/graph+dbms). 
Running Neo4j requires [Java], so we installed that as well. To get our graph
database running, it is just downloading, extracting, `cd`-ing to the extracted 
directory and running `bin/neo4j start`

We initialized a rails project with `rails new graphdatabase`. After that we 
added `gem 'neo4j'` to our Gemfile. There are a different number of gems that provide ruby support for Neo4j.
The reason we chose the [neo4j gem] is that it mimics the behavior of ActiveRecord, 
and has a lot of magic that speeds up development.

Getting the Rails ORM to work with Neo4j, we have to alter some files

Add to `application.rb`:

```ruby
# config/application.rb
require 'neo4j/railtie'

# ...20 lines omitted...

    config.neo4j.session_options = { basic_auth: { username: 'neo4j', password: 'pzzwrd'} }
    config.neo4j.session_type = :server_db
    config.neo4j.session_path = 'http://localhost:7474'
    
    # only use neo4j as data source
    config.generators { |g| g.orm :neo4j }
```

This should set up our application so it uses neo4j as datastore.

## Scaffolding our objects
We have simplified our domain. There are a lot of factors deciding whether a candidate 
is the best fit for a job. For our hackathon we decided that the 'winning' candidate
is the candidate that best fits the competences that the job opening demands.

* we have candidates,
* candidates can have educations,
* educations provide competences,
* jobs require competences

As a side note, me and my colleges come from an RDBM background with (just some) knowledge
of document stores. And because we were learning something new we chose the path we knew 
from our RDBM experiences.

We decided to give our candidates a name and an age, our educations have a name 
and a level, and our competences just have a name.  
We scaffold those in our framework.

```
rails g scaffold candidate name:string age:integer
rails g scaffold education name:string level:integer
rails g scaffold competence name:string
```

The we found on some documentation that our relations ar objects that can have 
their own attributes. That is when, in retrospect, I started to realize that 
we were too focused on the RDBM way, i.e. we were building what we knew, not
what we could. 
Because we were so focused on getting it working, that penny 
didn't drop until later (well, the weekend following the hackaton for me, actually).

### Defining the graph
We commenced forward, defining our relations (for RDBM thinkers), better known 
as the edges (in graph database terminolgy). 
We started with the relation on our candidate. 
Fortunately for us the Neo4j gem's syntax mimics the conventions used in the ActiveRecord gem.

We edited our candidate model to add the relation to the educations. 
The power of this database type is the ability to define properties on the relation
between nodes. Thankfully we can define a class and `include Neo4j::ActiveRel`.

Our candidate can have outgoing edges to educations, and we define these as having the type 'enrollment'.

```ruby
# app/models/candidate.rb

# ...snip...

# define our relation to educations
has_many :out, :educations, type: 'enrollment'
```

That leaves us to define the `Enrollment` class, with some properties as well.

```ruby
# app/models/enrollment.rb
class Enrollment
  include Neo4j::ActiveRel

  from_class  Candidate
  to_class    Education
  type        'enrollment'

  property :since,    type: Date
  property :finished, type: Boolean
end

```
Our educations define a relation to competences in the same way, but those relations are of the type 'provided_competences'.

## But what can we do with it?
 
Let's start by seeding our database.



[Neo4j]: http://neo4j.com/download-thanks/?edition=community&flavour=unix&release=2.2.2#
[Rails]: http://rubyonrails.org/
[Java]: http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html
[neo4j gem]: https://rubygems.org/gems/neo4j
