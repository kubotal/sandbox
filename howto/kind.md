

## Network setting

Create a 'kind' network to allow fixed IP
> This is not required for our single node cluster. But will be compatible with multi-nodes

```
docker network rm kind # If kind was already used.
docker network create -d=bridge -o com.docker.network.bridge.enable_ip_masquerade=true -o com.docker.network.driver.mtu=65535  --subnet 172.18.0.0/16 kind

docker network inspect kind
```


## kind cluster



```
cat >$(brew --prefix)/etc/dnsmasq.d/kind <<EOF
address=/first.pool.kind.local/172.18.200.1 
address=/.ingress.kind.local/172.18.200.1 
address=/padl.kind.local/172.18.200.2 
address=/tcp2.kind.local/172.18.200.2 
address=/tcp3.kind.local/172.18.200.3 
address=/last.pool.kind.local/172.18.200.4 
EOF


sudo brew services restart dnsmasq

sudo killall -HUP mDNSResponder
```


```

cat >/tmp/kind-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind
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
--path=clusters/kind/kind/flux
```


```

kubectl sk init https://skas.ingress.kind.local
kubectl sk init https://skas.ingress.kind.local --force


kubectl sk login admin admin
kubectl sk user create sa --commonName "Serge ALEXANDRE" --email "sa@broadsoftware.com" --password as
kubectl sk user bind sa "system:masters"
kubectl sk user bind sa "skas-admin"
kubectl sk logout

```




cleanup

```
docker stop $(docker ps -a -q)
docker container prune --force
docker volume prune --all --force
docker network prune --force
docker image prune --all --force

```
