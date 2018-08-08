---
  title: "Service Config Files"
  weight: 6
---

## Service Config Files

In a stack you can define the contents of files that can then be injected into containers. For example

```
configs:
  index:
    content: "<h1>hi</h1>"
    
services:
  nginx:
    image: nginx
    ports:
    - 80/http
    configs:
    - index:/usr/share/nginx/html/index.html

```

Configs can be defined using a string or base64 format, using
either the contents or encoded keys for string or base64
respectively.

```
configs:
  index:
    # String format
    content: "<h1>hi</h1>"
  index2:
    # base64, for binary data
    encoded: PGgxPmhpPC9oMT4K
```

### rio cat [NAME...]

Echo to standard out the contents of the reference config

### rio config ls

List all configs

### rio config create NAME FILE|-

Create a config of the given NAME from FILE or from standard input if `-` is passed.

### rio config update NAME FILE|-

Update a config of the given NAME from FILE or from standard input if `-` is passed.

### rio edit

The standard `rio edit` command can edit configs also
