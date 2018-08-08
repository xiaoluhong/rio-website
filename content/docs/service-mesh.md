---
title: Service Mesh
weight: 7
---

## Service Mesh

### Staging Versions

Service mesh will route traffic to given services.  Services can have multiple versions of
the service deployed at once and then the you can control how much traffic, or which traffic
is routed to each version.

### rio stage [OPTIONS] SERVICE_ID_NAME

The `rio stage` command takes all the same options as `rio create` but instead of updating
the existing service, it will stage a new version of the service.  For example, below is
scenario to do a canary deployment.

```bash

# Create a new service
$ rio run -p 80/http --name test/svc --scale=3 ibuildthecloud/demo:v1

# Ensure service is running and determine public URL
$ rio ps
NAME       IMAGE                    CREATED          SCALE     STATE     ENDPOINT                                  DETAIL
test/svc   ibuildthecloud/demo:v1   17 seconds ago   3         active    http://svc.test.8gr18g.lb.rancher.cloud   

# Stage new version, updating just the docker image and assigning it to "v2" version.
$ rio stage --image=ibuildthecloud/demo:v2 test/svc:v2

# Notice a new URL was created for your staged service
$ rio ps
NAME          IMAGE                    CREATED        SCALE     STATE     ENDPOINT                                     DETAIL
test/svc      ibuildthecloud/demo:v1   10 hours ago   3         active    http://svc.test.8gr18g.lb.rancher.cloud      
test/svc:v2   ibuildthecloud/demo:v2   10 hours ago   3         active    http://svc-v2.test.8gr18g.lb.rancher.cloud   

# Access current service
$ curl -s http://svc.test.8gr18g.lb.rancher.cloud
Hello World

# Access staged service under new URL
$ curl -s http://svc-v2.test.8gr18g.lb.rancher.cloud
Hello World v2

# Export to see stack file format
$ rio export test
services:
  svc:
    image: ibuildthecloud/demo:v1
    ports:
    - 80/http
    revisions:
      v2:
        image: ibuildthecloud/demo:v2
        scale: 3
    scale: 3

# Send some production traffic to new version
$ rio weight test/svc:v2=50%

# See that 50% of traffic goes to new service
$ curl -s http://svc.test.8gr18g.lb.rancher.cloud
Hello World
$ curl -s http://svc.test.8gr18g.lb.rancher.cloud
Hello World v2

# Happy with the new version we promote the stage version to be the primary
$ rio promote test/svc:v2

# All new traffic is v2
$ curl -s http://svc.test.8gr18g.lb.rancher.cloud
Hello World v2
$ curl -s http://svc.test.8gr18g.lb.rancher.cloud
Hello World v2

```
