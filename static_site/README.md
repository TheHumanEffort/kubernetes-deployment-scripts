== Kubernetes Static Site Deployment

This script makes it easy to get started with a staging/production
static site pushed out to Kubernetes.  It is designed to be copied
into your app so that you can modify it to suit your needs.  It
currently is designed to use a custom load balancer, but can easily be
modified to support Ingress rules.

== Prerequisites

You need to already have a Kubernetes cluster set up and linked to
your kubectl command.  This guide also assumes you have a Makefile and
Dockerfile in the parent directory that know how to build your product
in a sensible way.  The makefile should be simple enough to modify it
to build your product in such a way that you can link it to whatever
other build system you have.

== Usage

Make a directory in your app directory for deployment-related details:

```
cd <your app>
mkdir deploy
cd deploy
```

Download the Makefile, and getting-started kubernetes YAML files.

```
curl --remote-name-all https://raw.githubusercontent.com/TheHumanEffort/kubernetes-deployment-scripts/master/static_site/{Makefile,production.yaml,staging.yaml}
```

Now configure things:

```
make
```

This will step you through the steps required to configure the files
to suit your needs.  Now release again:

```
make release
```

