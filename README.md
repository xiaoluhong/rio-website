Rio Website
------------
[![Build Status](https://drone-publish.rancher.io/api/badges/rancherlabs/rio-website/status.svg)](https://drone-publish.rancher.io/rancherlabs/rio-website)


## Running for development/editing

The `rancherlabs/rio-website:dev` docker image runs a live-updating server.  To run on your workstation, run:

```bash
  ./scripts/dev
```

and then navigate to http://localhost:9002/.  You can customize the port by passing it as an argument:

```bash
  ./scripts/dev 8080
```
