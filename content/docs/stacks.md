---
title: "Stack Files"
weight: 4
---

## Stack Files

Stacks in Rio can be imported, exported and dynamically edited.  The syntax of the stack files
is an extension of the docker-compose format.  We wish to be backwards compatible with
docker-compose where feasible.  This means Rio should be able to run a docker-compose file, but
a Rio stack file will not run in docker-compose as we are only backwards compatible.  Below is an
example of more complex stack file that is used to deploy istio

```yaml
configs:
  mesh:
    content: |-
      disablePolicyChecks: true
      ingressControllerMode: "OFF"
      authPolicy: NONE
      rdsRefreshDelay: 10s
      outboundTrafficPolicy:
        mode: ALLOW_ANY
      defaultConfig:
        discoveryRefreshDelay: 10s
        connectTimeout: 30s
        configPath: "/etc/istio/proxy"
        binaryPath: "/usr/local/bin/envoy"
        serviceCluster: istio-proxy
        drainDuration: 45s
        parentShutdownDuration: 1m0s
        interceptionMode: REDIRECT
        proxyAdminPort: 15000
        controlPlaneAuthPolicy: NONE
        discoveryAddress: istio-pilot.${NAMESPACE}:15007

services:
  istio-pilot:
    command: discovery
    configs:
    - mesh:/etc/istio/config/mesh
    environment:
    - POD_NAME=$(self/name)
    - POD_NAMESPACE=$(self/namespace)
    - PILOT_THROTTLE=500
    - PILOT_CACHE_SQUASH=5
    global_permissions:
    - '* config.istio.io/*'
    - '* networking.istio.io/*'
    - '* authentication.istio.io/*'
    - '* apiextensions.k8s.io/customresourcedefinitions'
    - '* extensions/thirdpartyresources'
    - '* extensions/thirdpartyresources.extensions'
    - '* extensions/ingresses'
    - '* extensions/ingresses/status'
    - create,get,list,watch,update configmaps
    - endpoints
    - pods
    - services
    - namespaces
    - nodes
    - secrets
    image: istio/pilot:0.8.0
    secrets: identity:/etc/certs
    sidekicks:
      istio-proxy:
        expose:
        - 15007/http
        - 15010/grpc
        image: istio/proxyv2:0.8.0
        command:
        - proxy
        - --serviceCluster
        - istio-pilot
        - --templateFile
        - /etc/istio/proxy/envoy_pilot.yaml.tmpl
        - --controlPlaneAuthPolicy
        - NONE
        environment:
        - POD_NAME=$(self/name)
        - POD_NAMESPACE=$(self/namespace)
        - INSTANCE_IP=$(self/ip)
        secrets: identity:/etc/certs

  istio-citadel:
    image: "istio/citadel:0.8.0"
    command:
    - --append-dns-names=true
    - --grpc-port=8060
    - --grpc-hostname=citadel
    - --self-signed-ca=true
    - --citadel-storage-namespace=istio-system
    global_permissions:
    - write secrets
    - serviceaccounts
    - services
    permissions:
    - get,delete deployments
    - get,delete serviceaccounts
    - get,delete services
    - get,delete deployments
    - get,list,update,delete extensions/deployments
    - get,list,update,delete extensions/replicasets
    secrets: identity:/etc/certs

  istio-gateway:
    labels:
      "gateway": "external"
    image: "istio/proxyv2:0.8.0"
    net: host
    dns: cluster
    command:
    - proxy
    - router
    - -v
    - "2"
    - --discoveryRefreshDelay
    - '1s' #discoveryRefreshDelay
    - --drainDuration
    - '45s' #drainDuration
    - --parentShutdownDuration
    - '1m0s' #parentShutdownDuration
    - --connectTimeout
    - '10s' #connectTimeout
    - --serviceCluster
    - istio-proxy
    - --zipkinAddress
    - ""
    - --statsdUdpAddress
    - ""
    - --proxyAdminPort
    - "15000"
    - --controlPlaneAuthPolicy
    - NONE
    - --discoveryAddress
    - istio-pilot:15007
    env:
    - POD_NAME=$(self/name)
    - POD_NAMESPACE=$(self/namespace)
    - INSTANCE_IP=$(self/ip)
    - ISTIO_META_POD_NAME=$(self/name)
    secrets: identity:/etc/certs
    global_permissions:
    - "get,watch,list,update extensions/thirdpartyresources"
    - "get,watch,list,update */virtualservices"
    - "get,watch,list,update */destinationrules"
    - "get,watch,list,update */gateways"

```

### rio export STACK_ID_OR_NAME

Export a specific stack.  This will print the stack to standard out.  You can pipe the out
of the export command to a file using the shell, for example `rio export mystack > stack.yml`

### rio up [OPTIONS] [[STACK_NAME] FILE|-]

Import a stack from file or standard in.  The `rio up` can be ran in different forms

```bash
# Create stack foo from standard input
cat stack.yml | rio up foo -

# Create stack foo from file
rio up foo stack.yml

# Run up for all files in the current directory matching *-stack.yml.  The portion before
# -stack.yml will be used as the stack name
rio up
```

### rio edit ID_OR_NAME

Edit a specific stack and run `rio up` with the new contents.

### Questions

When running `up` stack files can prompt the user for questions.  To define a question add questions
to your stack file as follows

```yaml
services:
  foo:
    environment:
      VAR: ${BLAH}
    image: nginx
    
questions:
- variable: BLAH
  description: "You should answer something good"
```

The values of the questions can be references anywhere in the stack file using
${..} syntax.  The following bash style variables are supported (using [github.com/drone/envsubst](http://github.com/drone/envsubst))

```
${var^}
${var^^}
${var,}
${var,,}
${var:position}
${var:position:length}
${var#substring}
${var##substring}
${var%substring}
${var%%substring}
${var/substring/replacement}
${var//substring/replacement}
${var/#substring/replacement}
${var/%substring/replacement}
${#var}
${var=default}
${var:=default}
${var:-default}
```

Questions have the following fields

| Name                | Description |
| ------------------- | ----------- |
| variable            | The variable name to reference using ${...} syntax |
| label               | A friend name for the question |
| description         | A longer description of the question |
| type                | The field type: string, int, bool, enum.  default is string |
| required            | The answer can not be blank |
| default             | Default value of the answer if not specified by the user |
| group               | Group the question with questions in the same group (Most used by UI) |
| min_length          | Minimum length of the answer |
| max_length          | Maximum length of the answer |
| min                 | Minimum value of an int answer |
| max                 | Maximum value of an int answer |
| options             | An array of valid answers for type enum questions |
| valid_chars         | Answer must be composed of only these characters |
| invalid_chars       | Answer must not have any of these characters |
| subquestions        | A list of questions that are considered child questions |
| show_if             | Ask question only if this evaluates to true, more info on syntax below |
| show_subquestion_if | Ask subquestions if this evaluates to true |

For `showIf` and `showSubquestionsIf` the syntax is `VARIABLE=VALUE [&& VARIABLE=VALUE]`.  For example

```
questions:
- variable: STORAGE
  description: Do you want to use persistent storage
  type: bool
- variable: STORAGE_TYPE
  type: enum
  options:
  - aws
  - local
- variable: SIZE
  description: Size of the volume to create
  show_if: STORAGE
- variable: AWS_KEY
  description: Enter AWS API key
  show_if: STORAGE && STORAGE_TYPE=aws
```

### Templating

All stack files are go templates so any go templating can be used.  Please remember that
heavy use of templating makes the stack files hard to read so use conservatively.  Also,
all stack files must render with all empty variable.

`Values` is put into the template context as a map of all variable values, for example `{{ if eq .Values.STORAGE "true" }}`.
Regardless of the type of the question, the `Values` map will always contain strings.

