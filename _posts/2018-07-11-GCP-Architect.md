---
layout: post
title: Architect Note
---

### App Engine

#### location

app engine is regional and cannot be changed after set.

#### standard environment vs flexible environment

standard environment runs in sandbox. scale fast. cheap.
flexible environment runs in docker. support more languages.

#### flexible environment vs. compute engine

vm instances in flexible environment restarts on a weekly basis. 
flexible environment has no root access to vm instance.


#### hwo to scale 

load test. spikey load test. Flush memcache and stop instance to simulate.
set daily spending limit.
api not generally available is not subject to strict api quota limit.
shard task queues if high throughput is needed.
deal with version updating(some instances will be unvailable): traffic migration(enable warmup requests) and traffic splitting. 
caching problems: use cache-control and expires http headers for dynamic resource. change resource url when updating version if resource is static.
