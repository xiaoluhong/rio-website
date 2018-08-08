---
title: "Concepts"
weight: 2
---

### Service

The main unit that is being dealt with in Rio are services.  Services are just a collection of containers that provide a
similar function.  When you run containers in Rio you are really creating a Scalable Service.  `rio run` and `rio create` will
create a service.  You can later scale that service with `rio scale`.  Services are assigned a DNS name so that group
of containers can be accessed from other services.

### Stack

A stack is a group of services and their related resources, such as configuration files, volumes and routes.  A stack
ends up typically representing one application.  All the names of services are unique within a stack, but not globally
unique.  This means a stack creates a scope for service discovery.  Under the hood a stack will use a Kubernetes
namespace.

### Workspace

A workspace is a collection of stacks, and other resources such as secrets. The `rio` command line runs commands within
a single workspace.  Using `rio --workspace WORKSPACE` you can point to a different workspace.  Stack names are unique
within a workspace only.  As the permissions model of Rio matures the workspace will be the primary unit that is used
for collaboration.  Users are invited to and given access to workspaces.

### Service Mesh

Rio has a built in service mesh, powered by Istio and Envoy.  The service mesh provides all of the core communication
abilities for services to talk to each other, inbound traffic and outbound traffic.  All traffic can be encrypted,
validated, and routed dynamically according to the configuration.  Rio specifically does not require the user to
understand much about the underlying service mesh.  Just know that all communication is going through the service mesh.

