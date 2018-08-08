---
title: "Basics"
weight: 3
---

For each of these commands you can run `rio cmd --help` to get all the available options.

### rio run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a scalable service with given options.  There's a lot of options, run `rio run --help`

### rio create [OPTIONS] IMAGE [COMMAND] [ARG...]

The same as run but create a service with scale=0.  To start the service afterwards
run `rio scale`

### rio ps [OPTIONS] [STACK...]

List the running services or containers.  By default `rio ps` will show services.  To
view the individual containers backing the service run `rio ps -c` or you can run
`rio ps myservice` to list the containers backing a specific service.

### rio scale [SERVICE=NUMBER...]

Scale a service up or down. You can pass as many services as you wish for example

    rio scale myservice=3 otherstack/myservice2=1

### rio rm [ID_OR_NAME...]

Delete a resource.  `rio rm` will delete most any resource by ID or name excepts nodes.
If the name matches multiple resources the CLI will ask you which specific one to delete.
You can use IDs and the `--type` option to narrow down to delete specific things and not use
fuzzy matching.

### rio inspect [ID_OR_NAME...]

Return the raw json API response of the object.  You can use `--format` to change
to yaml or format the output using go formatting.
