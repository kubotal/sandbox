
# Sandbox: Introduction

Aim of this sandbox is two demonstrate two principles

- A fully automated deployment, starting from a bare kubernetes cluster to a fully featured one, including middleware 
and application. This in a fully automated way.
- Full GitOps, where all action on the cluster will be performed by editing files in git, without direct access to the cluster.

It mostly rely on [Flux](https://fluxcd.io/) 

Here is a short resume of the steps to be performed:

- Create a cluster with Kind
- Copy this repository
- Perform some configuration. 
- Bind the cluster to the repo using the `flux` CLI command.
- Wait for all components to be deployed

# Prerequisite

- Docker Desktop
- [Kind](https://kind.sigs.k8s.io/) 
- For Mac Os: [Docker Mac Net Connect](https://github.com/chipmk/docker-mac-net-connect)
- kubectl
- flux, the [FluxCD CLI client](https://fluxcd.io/flux/installation/#install-the-flux-cli)
- Internet access
- A Github account and a GitHub personal access token with repo permissions. See the GitHub documentation on creating a personal access token.


Optionally:

- [k9s](https://k9scli.io/): A terminal based UI to interact with your Kubernetes clusters
- dnsmasq: To ease resolution of local DNS name. Editing /etc/hosts is a viable alternative.

# Deployment

## cluster creation

```
kind create cluster
```

This will create a fully operational kubernetes cluster, with a single node acting both as control plane and worker.

> It is of course possible to create more sophisticated cluster with several workers or/and control plane node. 
But, take care if you intend to build cluster with more than one control plane node. 
See [kind-fip](https://github.com/kubotal/kind-fip/blob/fixeip/README.md)

You can ensure your cluster is up and running:

```
kubectl get --all-namespaces pods
# or
k9s
```

## Setup our cluster repository

Next step is to create your own copy of this repository. For this, click on the `Use this template` button on the upper 
right corner of this repo main page.

> This procedure assume you have copied the repo into your personal GitHub account, under the name `sandbox`

> It is better to copy the repo using this method than performing a Fork, which is more restrictive.

This repo will be the driver of the state of you cluster. In other words, the included tooling (based on FluxCD) will
permanently reconcile the configuration of the cluster with the content of the repo.

## Configuration

One key point of this sandbox is we will access the provided service through VIP (Virtual IP) on the docker network, 
using a load balancer. This will allow us to refer to services through DNS name, and not by using cumbersome local 
port number. And also this will be more realistic, similar to a 'real' cluster in the cloud or on bare metal.

> This why, on  Mac Os, we require [Docker Mac Net Connect](https://github.com/chipmk/docker-mac-net-connect) to be
installed. It will provide access from the host to the Docker network.

The load balancer used here is [metallb](https://metallb.universe.tf/). We need to provide a range of IP for metallb 
to allocate VIPs. This range must be in the IPv4 subnet used by kind, but without conflict with existing containers.

The first step is to figure out what is this IPv4 subnet. For this, you can issue the following command:

```
docker network inspect kind -f '{{ .IPAM.Config}} }}' | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}/[0-9]{1,2}'
```

The result is typically `172.18.0.0/16` or `172.19.0.0/16`. For this README, let's assume it is `172.18.0.0/16`. If not, 
you should adjust accordingly.

As docker allocate containers IP from the beginning of the range, we can assume than using IP above 172.18.200.0 is safe.

So, for our cluster we will allocate a small range from `172.18.200.1` to `172.18.200.4`. And the first one will be 
bound to the ingress entry.

To define this range, enter the following in local `/etc/hosts` file:

```
172.18.200.1 first.pool.kind.local ingress.kind.local podinfo.ingress.kind.local
172.18.200.4 last.pool.kind.local 
```

`podinfo` is a sample application, which will be accessible through the ingress controller. As the `/etc/hosts` file 
does not accept some wildcard (such as *.ingres.kind.local), each ingress entry must be explicitly bound to the VIP 

Alternatively, if `dnsmasq` is configured on your system, you can configure the following:

```
address=/first.pool.kind.local/172.18.200.1 
address=/.ingress.kind.local/172.18.200.1 
address=/last.pool.kind.local/172.18.200.4 
```

`address=/.ingress.kind.local/172.18.200.1` is the 'dnsmasq way' to configure wildcard name. So 
`podinfo.ingress.kind.local` will resolve to `172.18.200.1`

These value must now be configured in the Git repository. Edit the file `/clusters/kind/kind/context.yaml`:

```
# Context specific to a kind cluster named 'kind'

context:

  cluster:
    name: kind

  apiServer:
    portOnLocalhost: 53220    # <== To configure

  metallb:
    ipRanges:
      - first: 172.18.200.1  # <== To configure
        last: 172.18.200.4   # <== To configure

  ingress:
    urlRoot: ingress.kind.local
    vip: 172.18.200.1    # <== To configure
```

to reflect your subnetwork, if different.

Also, you must configure `apiServer.portOnLocalhost` with the value of the port on localhost exposing the k8s API api 
server. It is the external port bound the the internal port `6443`. To find it; just enter `docker ps`:

```
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS       PORTS                       NAMES
7ef081115829   kindest/node:v1.29.2   "/usr/local/bin/entr…"   7 hours ago   Up 7 hours   127.0.0.1:53220->6443/tcp   kind-control-plane
                                                                                                    -----
```

> If you edit these file locally, after cloning your repo, don't forget to commit....

## Boostrap FluxCD

Now, we can bootstrap our deployment process, by using our Github token and the `flux` CLI command

First, you need to setup some environment variables:

```
export GITHUB_USER=<Your username>
export GITHUB_TOKEN=<your token>
export GITHUB_REPO=sandbox
```

Some points to note here:

- It is assumed the repo was copied into your **personal** GitHub account, under the name `sandbox`. 
If not the case, the command above should be slightly modified. Refer to the FluxCD documentation.
- The repository will be updated by the `flux` command. So, the provided token must allow such access.

Then:

```
flux bootstrap github \
--owner=${GITHUB_USER} \
--repository=${GITHUB_REPO} \
--branch=main \
--interval 15s \
--personal \
--path=clusters/kind/kind/flux
```







## What is installed

## Adding a Certificate Authority

## More about configuration

## SKAS (Kubernetes authentication)

# Roadmap

- Ubuntu VM as host
- Windows as host (If possible ?)
- Vagrant/kubespray based cluster.

