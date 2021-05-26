---
title: "Welcome to My Dumb Blog"
date: 2021-05-26T11:33:45-04:00
draft: false
---

# evaluation instance

https://console-openshift-console.apps.srust-openshift.tn.akamai.com/

Login: kubeadmin
Password: EM9a3-cQj5I-pHj9A-kGGpT

# Initial Impressions

Cert appears to be self-signed and issued by the ingress operator
Common Name
ingress-operator@1621277462

Landing page shows the status of these categories

## Cluster
## Control Plane
#  API Servers, Controller Managers, Schedulers, and an item for API Request
#  Success RAte
#
## Operators
#  Shows 8 "Operators" 
### Advanced Cluster Management for Kubernetes" (Redhat)
### Red Hat OpenShift Logging
### Kiali Operator
### OpenShift Virtualization
### Package Server
### Prometheus Operator
### Red Hat OpenShift Pipelines
### Red Hat OpenShift Service Mesh
#   Looks to be branded Istio
#
#  31 "Cluster operators" installed
### authentication
### baremetal
### cloud-credential
### etcd
### image-registry
### kube-apioserver
### a bunch of others

I don't understand the difference between "Operator" and "Cluster Operator"

On the Cluster tab, there's also a "Global configuration" link. This is broken
down by "Configuration resource". The settings for these things show a
friendly view where fields from the literal configuration are shown in a
table, and then a YAML view which can be edited on the fly.

* I've read a little that the actual management of openshift is not presently
  performed via gitops.

It looks like the alerts feature surfaced in the portal is using prometheus
alert-manager.

## Insights

Seems to be similar in function to the Unifi insights feature, surfaces
recommendations about tweaks.

## Virtualization

No details shown

## Projects View

There are a large number of projects in the list which appear to be created by
default, like "hive" and "openshift".

* Are projects just a synonym for namespaces? Looking at the yaml that appears
    to be the case.

# Project Creation

In the Projects view there is a "Create Project" option.

I created one called "bailey-project"

There is an activity view on the Project detail page which shows the events
that occurred as a result of this request. In this instance, there is one
even, "created SCC ranges".

There are a bunch of default RoleBindings already- I guess these are either
added by default or there is some manner of inheritance.

## User Management

Going out of order, but I would like to see what sort of account I would use
in order to set up a deployment as a non-administrative user.

Well that was quick- in order to create users, you must create an Identity
Provider (the system uses Oauth) and we don't have one configured.

Groups _can_ be created with no IDP configured.

ServiceAccounts- some exist already "builder", "default", "deployer" and
"pipeline"

Roles- a huge number of roles exist, and a huge number of RoleBindings


## Administration

This is actually the same view we're taken to if we click "Cluster" on the
landing page.

It would be interesting to compare the Projects list with the Namespaces list.
There doesn't appear to be a way to export the table data in a machine
readable way, but obviously we can use the API to do this.

I noticed that my view of things seems to be narrowed by the project that I
created, and while looking at the top right to see if the login menu provided
some waty of switching that context, I noticed that the system provides you
with API login information where a token has been generated and substituted
in. They provide a curl invocation:

curl -H "Authorization: Bearer sha256~nyXACUuu0JXIKTlD_0vNYKWaiQ0ZyNUTrSFVQQv4mG8" "https://api.srust-openshift.tn.akamai.com:6443/apis/user.openshift.io/v1/users/~"

So that will get me into the API, which is documented at:

https://docs.openshift.com/container-platform/4.7/welcome/index.html

### Administration via API

It's actually a collection of APIs, so for example to look at projects we
would use.

The API is self-documenting- e.g. a request to:

https://api.srust-openshift.tn.akamai.com:6443/apis/project.openshift.io/v1/

will return a structure like:

{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "project.openshift.io/v1",
  "resources": [
    {
      "name": "projectrequests",
      "singularName": "",
      "namespaced": false,
      "kind": "ProjectRequest",
      "verbs": [
        "create",
        "list"
      ]
    },
    {
      "name": "projects",
      "singularName": "",
      "namespaced": false,
      "kind": "Project",
      "verbs": [
        "create",
        "delete",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ]
    }
  ]
}

And so I can then make a request, e.g.:

https://api.srust-openshift.tn.akamai.com:6443/apis/project.openshift.io/v1/projects

To enumerate those. Pipe the output of that, to e.g "jq .items[0]" to get the
first item, or "jq .items[].metadata.name" to get the list. Now we can see
whether projects and namespaces are synonyms.

There is an API "core" which has a bunch of stuff which appears to be sort of
core-k8s stuff. I can't quite figure out how to query that one, but a request
to /apis enumerates them all, so lets see if we can find it that way.

I don't see anything obvious in that output though.

This view is better:

https://docs.openshift.com/container-platform/4.7/rest_api/index.html

Aha, this appears to work:

/api/v1/namespaces | jq .items[].metadata.name

So at the moment, Projects and Namespaces are identical.





## ResourceQuotas

No ResourceQuotas exist presently.

## CustomResourceDefinitions

There are a bunch- it's a mix of Red Hat stuff and other third party

# Compute

Shows the status of the Nodes. Curiously there is also a "Machines" list that
says no Machines are configured. What is a Machine? Machines have their own
API- I'm guessing maybe these are k8s-hosted VMs.

This is pretty interesting because it means that the platform that we used for
managing the k8s clusters would get us some amount of VM hosting capability.
Obviously this is probably possible with Rancher as well- I think the question
would be rather the Rancher APIs expose this ability in a straightforward way.

# Workloads

This was probably the next place I should have gone but I was curious about
the other stuff.

Pods are exposed in this view, obviously ther are a boatload of them.

There is a Virtualization view, which appears to enumerate VMs. I'm guessing
this is sort of redundant with Machines but they could also be differing APIs
for similar purpose (maybe one being legacy)

Deployments are enumerated here and they're a mix of the platform services and
user stuff. For example someone has added an "akamai-static-pulsar-app" item.

DeploymentConfigs is empty. What are these? On the create view it says that
DeploymentConfigs define the template for a pod (presumably a deployment can
reference a DeploymentConfig to use when creating pods?)

I intend to create a deployment, but I'll come back to that after skimming
through the portal more.

# Services

There's one Service created "akamai-static-pulsar-app".


# Builds

There are no BuildConfigs configured at present, it looks like maybe we need
to create one first in order to use Builds.

# Pipelines



# Questions for Steve at this point

OpenShift Projects and k8s Namespaces appear to be used interchangeably. Do
you know of exceptions?

To create the Service entry for akamai-static-pulsar-app, did you just create
static yaml with the deployment and service etc?

It doesn't look like by default we could manage Openshift itself in the gitops
manner.


I see stuff like CASL for provisioning the openshift infrastructure

The openshift gitops stuff seems to imply that the argocd instance can be used
for the actual cluster management itself.
