---
layout: post
title: How a Git Error Made Me Clean Unrelated Oracle Database Trace Files 
comments: false
image: https://res.cloudinary.com/bass3l/image/upload/v1595589177/gitlab-and-docker_qunu9e.png
---

In a shiny day, I was pulling the latest changes from a repository in our own instance of GitLab CE, which is deployed on our docker cloud infrastructure, and was faced with a weird error, I've checked my colleagues and most of them were encountering this error in a random-fashion, this is how it went on.

<!--more-->

![gitlab-docker](https://res.cloudinary.com/bass3l/image/upload/v1595589177/gitlab-and-docker_qunu9e.png)

## TLDR;

When suddenly faced with the following error while **pulling** from a git repository on your own instance of a git solution, check your device storage, one of the errors might be related to insufficient storage:

```
> git pull origin master
error: RPC failed: HTTP 500 curl 22 The requested URL returned error: 500 Internal Server Error
fatal: The remote end hung up unexpectedly
```

When upgrading a GitLab Omnibus instance and you were faced with the following error:

```
Symlink at /var/log/gitlab/redis/config (pointing to /opt/gitlab/sv/redis/log/config) exists but attempting to resolve it leads to a nonexistent file
```

Follow [here](#upgrading-omnibus-gitlab-docker-instance) to fix it.

## Back to the story

Ok, so faced with a strange error on git (or anything in that matter), first thing I do is google it :P...

What I've found was there are multiple causes to such errors on git:

* If faced while pushing a commit, you might be pushing a large file (a large db-mock file, for example) in that case, either exclude it (should not be pushed in the first place) or extend your git client buffer, and that wasn't my case.

* There are permission related configurations which are missed up on your fresh GitLab installed instance, and that was not my case either. Our GitLab instance has been running for 2 years straight happily-ever-after (undiscouraged, of course :) ).

All were unrelated to our problem, so all was left to me was to dig deep into gitlab's sexy logs on its administration panel, two particular streams are interesting, which are `production-logs` and `git-logs`.

I wasn't able to find any HTTP-500 related errors, but both logs were showing errors related to `git-repack`, a module that is responsible of packing and compressing your repositories to be served to the git clients, and it was crashing.

A google search wasn't helpful in `git-repack` issues as well, but I got an idea, let's do a quick `df -h` and check the server mounting points and storages... and there it was:

```
/dev/mapper/vg-Docker    96G   96G    0G     100% /var/lib/docker
```

Our docker's container storage is full, we are using overlay2 as a storage driver and it sets the default size to 100Gb (about 96GB), let's do a quick housekeeping and check if the problem is really about storage:

```
> docker system prune
```

with `system prune`, you remove all stopped containers, unused networks, dangling images (unused images) and caches. After checking all important containers are up, I ran the prune command and was able to save about 5GB, enough to check the solution.

It really was about storage, I was able to pull successfully.

About 100GB of storage is used in our containers, although we use volumes to set them up on a different storage devices with 2TB of storage. We have about 15 containers up in a time spanning between apps and databases (All in staging) but there is a naughty one between them (consuming storage outside its mounted volumes) and I have to catch it, expanding `/var/lib/docker` requires deleting all images and containers in the process, and that is not feasible to me right now.

I `cd`'ed into `/var/lib/docker/volumes` and ran a `du -h` to catch this naughty one, and here what I've found interesting:

```
.....
15.0G ./_data/u01/app/oracle/diag/rdbms/orclcdb/ORCLCDB/trace
.....
```

We have an oracle database container, single instance, for staging purposes, and it has only two schemas with no-production load on it whatsoever. Oracle RDBMS is consuming over 30 GB of storage (excluding it's already mounted volumes), half of them are logs...

When I sat it up, I've followed [Oracle's documentation](https://github.com/oracle/docker-images/blob/master/OracleDatabase/SingleInstance/README.md) of running an Oracle RDBMS in a docker container and setting its data volumes as well. I guess it's not enough, you'll have to tweak the RDBMS configs yourself for storage thresholds, what a nightmare, .ORA files here we go. 

![oracle](https://res.cloudinary.com/bass3l/image/upload/v1595596344/meme-pain-in-the-ass_bpvofq.jpg)

I cleared the trace files and sat up the configs. I thought while I'm at it, let's upgrade the GitLab instance, what could go wrong.

## Upgrading Omnibus GitLab Docker Instance

The upgrade process of GitLab docker instance is straight forward, as long as you are using data volumes. You remove the container, pull the next version of its image, run the container with the old volumes and it should migrate to the new version seamlessly.

The problem is, I was neglecting the upgrades for two years now, where 2 major versions where released... more work should be done.

GitLab follows the famous `MAJOR.MINOR.PATCH` versioning, that means on each increment of MINOR and MAJOR, breaking changes and database changes are presented. So there is a required path of updates you should take one-by-one to reach the target version, the GitLab instance was setting at version `10.1.1`, here is the path I had to take:

```
10.1.1 -> 10.4.5 -> 10.8.7 -> 11.11.8 -> 12.0.12 > 12.10.6 -> 13.0.0 -> 13.2.0
```

At the first versions I faced some errors of missing log configuration files of redis, gitaly and postgres, here is an example:

```
Symlink at /var/log/gitlab/redis/config (pointing to /opt/gitlab/sv/redis/log/config) exists but attempting to resolve it leads to a nonexistent file
```

All the three errors on each upgrade are the same, missing config files, google search wasn't helpful. So I had an idea...

I pulled the latest image of GitLab and run it on fresh volumes as if installing GitLab for the first time, I inspected the files that my upgrade was complaining about from the new image, and they were as the following (the three of them):

```
s209715200
n30
t86400
!gzip
```

I created a file called `config` and pasted those lines in it, removed the fresh container, and was able to use `docker cp` command to copy the file into the volumes of our gitlab instance while it was in stopped state:

```
> docker cp ./config gitlab:/opt/gitlab/sv/redis/log/config
> docker cp ./config gitlab:/opt/gitlab/sv/gitaly/log/config
> docker cp ./config gitlab:/opt/sv/postgresql/log/config
```

And since those were logging-related config files  (fingers crossed), I hoped they won't make any breaking issues while `gitlab reconfigure` is running.

I had to make such adjusts to the gitlab volumes for at least four releases on my path, and it was successful in bypassing them.

Sometimes we face issues where we are not able to reach anything on the internet helpful to solve them, specifically on topics so big like Docker and GitLab, I hope this helps anyone faces such problems, and I hope that this was a good read.