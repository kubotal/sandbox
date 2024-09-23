# KUBO3

## Network setting

Create a 'kind' network to allow fixed IP
> This is not required for our single node cluster. But will be compatible with multi-nodes

```
docker network rm kind # If kind was already used.
docker network create -d=bridge -o com.docker.network.bridge.enable_ip_masquerade=true -o com.docker.network.driver.mtu=65535  --subnet 172.18.0.0/16 kind

docker network inspect kind
```


## kubo3 cluster



```
cat >$(brew --prefix)/etc/dnsmasq.d/kubo3 <<EOF
address=/first.pool.kubo3.mbp/172.18.120.1 
address=/.ingress.kubo3.mbp/172.18.120.1 
address=/padl.kubo3.mbp/172.18.120.2 
address=/ldap.kubo3.mbp/172.18.120.3 
address=/last.pool.kubo3.mbp/172.18.120.4 
EOF


sudo brew services restart dnsmasq

sudo killall -HUP mDNSResponder

ping ldap.kubo3.mbp
ping padl.kubo3.mbp
```


```

cat >/tmp/kind-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kubo3
networking:
  apiServerAddress: "127.0.0.1"
  apiServerPort: 5444
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
--path=clusters/kind/kubo3/flux
```


```

kubectl sk init https://skas.ingress.kubo3.mbp
kubectl sk init https://skas.ingress.kubo3.mbp --force

```


## minio1

```
mc alias set minio1 https://minio1-ext.ingress.kubo3.mbp minio minio123

mc alias ls

mc ls minio1
mc mb minio1/yyy
mc ls minio1
mc rb minio1/yyy
```

## minio2

```
mc alias set minio2 https://minio2-ext.ingress.kubo3.mbp minio minio123

mc alias ls

mc ls minio2
mc mb minio2/xxx
mc ls minio2
mc rb minio2/xxx
```

```
mc idp ldap policy attach minio2 consoleAdmin --group='cn=ops,ou=Groups,dc=odp,dc=com'
mc idp ldap policy attach minio2 consoleAdmin --user='uid=idiri,ou=Users,dc=odp,dc=com'
```

```
LDAPTLS_REQCERT=never ldapsearch -LLL -H ldaps://ldap.kubo3.mbp -D "cn=admin,dc=odp,dc=com" -w admin123 -x -bou=Users,dc=odp,dc=com uid=sergea
LDAPTLS_CACERT=/Users/sa/dev/d3/git/odp_certificates/CA/ca.crt ldapsearch -LLL -H ldaps://ldap.kubo3.mbp -D "cn=admin,dc=odp,dc=com" -w admin123 -x -bou=Users,dc=odp,dc=com uid=sergea
```


cleanup

```
docker stop $(docker ps -a -q)
docker container prune --force
docker volume prune --all --force
docker network prune --force
docker image prune --all --force



```
