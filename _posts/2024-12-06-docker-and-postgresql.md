---
layout: post
title:  "postgresql on docker"
date:   2024-12-06
author: Shani
---

I've upgraded `fedora` on my system to the latest stable version. And that fucked up my local `postgresql` installation.

I spent some time trying to fix it.
From what I could figure out, `postgresql` and `fedora` have both their own cycle of releases. It used to be synchronized. But no more.

I've started by playing with configuration files. Then I gave up and just tried to remove and reinstall `postgresql` packages. Still didn't work.

After a while, I've decided I'm spending too much time on this. I thought this could be a nice opportunity to try and add new technologies to my ecommerce project.

My plan was to get a running docker container with `postgresql` running on it, and having the ecommerce project use it for its db needs.

It turned out pretty good. I got to experiment with `docker-compose.yml` and `Dockerfile` - both I haven't used over than a year.
Since it made local deployment more complex, I automated it using `make` which gave me a chance to write my own `Makefile`.

For a long time, I couldn't get the container to run. It kept complaining about not having env variables for db name, username and password.
That drove me crazy, since I set `docker-compose.yml` to use the env variables set in my local env file, `local_settings.env`.

`docker-compose.yml`
```yml
services:
  db:
    build: .
    container_name: ecommerce_postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    env_file:
      - local_settings.env

volumes:
  postgres_data:  # This declares the volume

```

Took me a while to debug it. Turned out, I failed at reading error messages thoroughly:
The docker container was asking for `POSTGRES_*` variables, and my env file supplied it with `DB_*` variables.

After a quick fix, everything clicked together and I could commit it.

