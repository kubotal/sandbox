

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


## minio1

```
mc alias set minio1 https://minio1-ext.ingress.kubo2.mbp minio minio123

mc alias ls

mc ls minio1
mc mb minio1/yyy
mc ls minio1
mc rb minio1/yyy
```

Add Ldap config:

```
mc idp ldap add minio1/ \
      server_addr=skas-main-padl.skas-system.svc:636 \
      tls_skip_verify=on \
      lookup_bind_dn=cn=readonly,dc=system,dc=skasproject,dc=com \
      lookup_bind_password=mysecret \
      user_dn_search_base_dn=ou=users,dc=skasproject,dc=com \
      user_dn_search_filter="(uid=%s)" \
      group_search_base_dn=ou=groups,dc=skasproject,dc=com \
      group_search_filter="(&(objectclass=groupOfUniqueNames)(member=%d))" \
&& mc admin service restart minio1
```


To bind a policy (consoleAdmin) to a group:

```
k sk user bind sa minio1-admin

mc idp ldap policy attach minio1 consoleAdmin --group='cn=minio1-admin,ou=groups,dc=skasproject,dc=com'
```


## minio2

```
mc alias set minio2 https://minio2-ext.ingress.kubo2.mbp minio minio123

mc alias ls

mc ls minio2
mc mb minio2/xxx
mc ls minio2
mc rb minio2/xxx



```


```

k sk user bind sa minio2-admin

mc idp ldap policy attach minio2 consoleAdmin --group='cn=minio2-admin,ou=groups,dc=skasproject,dc=com'

mc idp ldap policy attach minio2 consoleAdmin --user='uid=sa,ou=users,dc=skasproject,dc=com'

```



cleanup

```
docker stop $(docker ps -a -q)
docker container prune --force
docker volume prune --all --force
docker network prune --force
docker image prune --all --force



```
