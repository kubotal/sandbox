

## Network setting

Create a 'kind' network to allow fixed IP
> This is not required for our single node cluster. But will be compatible with multi-nodes

```
docker network rm kind # If kind was already used.
docker network create -d=bridge -o com.docker.network.bridge.enable_ip_masquerade=true -o com.docker.network.driver.mtu=65535  --subnet 172.18.0.0/16 kind

docker network inspect kind
```


## kubo2 cluster



```
cat >$(brew --prefix)/etc/dnsmasq.d/kubo2 <<EOF
address=/first.pool.kubo2.mbp/172.18.110.1 
address=/.ingress.kubo2.mbp/172.18.110.1 
address=/padl.kubo2.mbp/172.18.110.2 
address=/ldap.kubo2.mbp/172.18.110.3 
address=/last.pool.kubo2.mbp/172.18.110.4 
EOF


sudo brew services restart dnsmasq

sudo killall -HUP mDNSResponder

ping ldap.kubo2.mbp
ping padl.kubo2.mbp
```


```

cat >/tmp/kind-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kubo2
networking:
  apiServerAddress: "127.0.0.1"
  apiServerPort: 5443
EOF

kind create cluster --config /tmp/kind-config.yaml


```

```

export GITHUB_USER=SergeAlexandre
export GITHUB_REPO=sandbox
export GITHUB_TOKEN=

flux bootstrap github \
--owner=${GITHUB_USER} \
--repository=${GITHUB_REPO} \
--branch=kdp2 \
--interval 15s \
--owner kubotal \
--path=clusters/kind/kubo2/flux
```


```

kubectl sk init https://skas.ingress.kubo2.mbp
kubectl sk init https://skas.ingress.kubo2.mbp --force

```




cleanup

```
docker stop $(docker ps -a -q)
docker container prune --force
docker volume prune --all --force
docker network prune --force
docker image prune --all --force

```
