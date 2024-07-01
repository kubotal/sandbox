


## Network setting

Create a 'kind' network to allow fixed IP

```
docker network rm kind # If kind was already used.
docker network create -d=bridge -o com.docker.network.bridge.enable_ip_masquerade=true -o com.docker.network.driver.mtu=65535  --subnet 172.18.0.0/16 kind

docker network inspect kind

```
# KUBO1 Cluster

```

cat >$(brew --prefix)/etc/dnsmasq.d/kubo1 <<EOF
address=/lb.kubo1.local/172.18.100.10 
address=/m0.kubo1.local/172.18.100.0 
address=/m1.kubo1.local/172.18.100.1 
address=/m2.kubo1.local/172.18.100.2 
address=/w0.kubo1.local/172.18.100.3 
address=/w1.kubo1.local/172.18.100.4 
address=/w2.kubo1.local/172.18.100.5 
address=/first.pool.kubo1.local/172.18.100.6 
address=/.ingress.kubo1.local/172.18.100.6 
address=/padl.kubo1.local/172.18.100.7 
address=/.tcp2.kubo1.local/172.18.100.8 
address=/.tcp3.kubo1.local/172.18.100.9
address=/last.pool.kubo1.local/172.18.100.9 
EOF

sudo brew services restart dnsmasq

sudo killall -HUP mDNSResponder
```

```

cat >/tmp/kubo1-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kubo1
networking:
  apiServerAddress: "127.0.0.1"
  apiServerPort: 7443
loadBalancer:
  dockerIP: lb.kubo1.local
nodes:
  - role: control-plane
    dockerIP: m0.kubo1.local
  - role: control-plane
    dockerIP: m1.kubo1.local
  - role: control-plane
    dockerIP: m2.kubo1.local
  - role: worker
    dockerIP: w0.kubo1.local
  - role: worker
    dockerIP: w1.kubo1.local
  - role: worker
    dockerIP: w2.kubo1.local
EOF

kind create cluster --config /tmp/kubo1-config.yaml

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
--path=clusters/kind/kubo1/flux


```




```
kubectl sk init https://skas.ingress.kubo1.local
kubectl sk init https://skas.ingress.kubo1.local --force


kubectl sk login admin admin
kubectl sk user create sa --commonName "Serge ALEXANDRE" --email "sa@broadsoftware.com" --password as
kubectl sk user bind sa "system:masters"
kubectl sk user bind sa "skas-admin"
kubectl sk user create jt --commonName "Jules TESTEUR" --email "jt@broadsoftware.com" --password tj
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

