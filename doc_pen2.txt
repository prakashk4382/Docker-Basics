

  
---

# Outline

- What we run

- Where it runs

- What we don't run

- How we build and deploy

- The bright happy future ahead

---


# What we run

---

# Docker Hub (high level)

- [hub.docker.com](http://hub.docker.com/)

- "GitHub for containers"

- Base images (busybox, debian, ubuntu, redis, ...)

- Public images (jpetazzo/dind ...)

- Private images (nsa/prism.red[*] ...)

- Automated builds (from GitHub/BitBucket)

---

# Docker Hub (low level)

- Web frontend: Django app

  - soon with React frontend + Django backend

- Heavy use of Celery workers

- Same image for frontend and workers

  - start gunicorn or celery depending on configuration

  - this ensures code consistency

---

# Registry

- Stores and serves Docker images

- Flask app (v1)

  - https://github.com/docker/docker-registry

- Golang app (v2)

  - https://github.com/docker/distribution

---

# Automated builds (aka Highland)

- Triggered by GitHub / BitBucket webhooks

- Pulls code

- Builds in a temporary Docker-in-Docker
   (Thrown away at the end of the build)

- Pushes resulting images to registry

---



# Where it runs

---

# Bare metal on Rackspace 

- What:

  - Docker Hub

  - Registry (except storage)

- Why:

  - Running containers on EC2 used to be... *complicated*
    (We know. We've done it for 5 years.)

  - Performance/Price

---

# S3 + CloudFront
 
- What:

  - Registry storage (S3)

  - Image pulls

- Why:

  - It's Web Scale ©

---

# Linode

- What:

  - Automated builds

- Why:

  - Cheap and elastic

  - **Choice**

---



# Choice

---

# Choice

- At Docker, each team is free to use the best tools for the job

  - language

  - framework

  - infrastructure

  - external services

- Team consensus (≠ every programmer for themselves)

---



# What wedon't run

---

# Things hosted by third parties

- ElasticSearch: [found.no](http://found.no/)

- Redis: [Redis Labs](http://redislabs.com/)

  - caching

  - formerly queuing

- RabbitMQ

- PostgreSQL: [Heroku Postgres](https://www.heroku.com/postgres)



We .red[♥] you!

---



# Wait!

---

# Doesn't that ring a bell?

--

- In the previous talk, RelateIQ told us:

.quote[We run (pretty much) everything in containers,
except persistent data stores]

--

- Coincidence? I think not!

---



# What do?



---



# What do?

Use Flocker maybe?

---

# Use Flocker maybe?

- Just my personal opinion ☺

- (I.e., not an endorsement by Docker Inc.!)

- Standard disclaimers apply, blah blah blah...

---

# Why don't we run those things ourselves?

- Our value add: Docker

- Not our value add: running those things

- But: we would like to run them in Docker

- And some of those will become mission-critical

- We might internalize some of them!

---



# How we build  and deploy

---

# The pipeline

- Private GitHub repositories

- For simple stuff: automated builds

- For complex stuff: private build server
   (Basically our own internal highland deployment)

- Private Docker Hub images

- Pre-pulls images on our servers

- Manual "go to production" button

---

# The build process

- Scripts

  - Fig

  - Stacker (predates Fig)

  - others... (remember: **Choice**)

- Everybody is converging to Compose

---

# Y U NO use automated builds?!?

- We want tagged releases

- Automated builds doesn't have it yet

  - `git tag -a v1.0 -m "First stable release"`

  - ... doesn't create (yet) `:v1.0` image

.quote[As soon as I have some time to work on it,
that's the first thing I'll implement! — Ken]

---

# The "go to production" button

- Blue/green deployment

- Deploy to blue *or* green

- Both run side by side:

  - frontends receive 50% of traffic

  - workers handle 50% of queued tasks

- Check bugsnag, metrics, etc...

- Then tear down green *or* blue

---

# Networking details

- Load balancing: [HAProxy](http://www.haproxy.org/)

  - Config in a bind-mounted volume

  - Config doesn't change
     (blue and green run on hard-coded ports)

- High availability: [UCarp](https://github.com/jedisct1/UCarp)



Note: they both run in containers!

---

# Things that are not containerized

- Some low-level metrics (collectd, datadog...)

- This might change thanks to Docker 1.5 `--pid=host`

---



# Docker 1.5is out!



---



# Docker 1.5is out!

.small[But I won't talk about it!]

---



# Future 

---

# Future

- Compose (getting there!)

- Swarm: not yet (it's not production ready)

- Machine: not yet (not significant yet)

- Experimentations with EC2

- Continuous deployment

---



# That's all folks!

---

# Questions?

- To get those slides, follow me on twitter: [@jpetazzo](https://twitter.com/jpetazzo)
  .small[Yes, this is a particularly evil scheme to increase my follower count]

- Also .red[WE ARE HIRING!]

  - infrastructure (servers, metal, and stuff)

  - QA (get paid to break things!)

  - Python (Docker Hub and more)

  - Go (Docker Engine and more)

- Send your resume to jobs@docker.com 
  
