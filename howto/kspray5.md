

Copied from ezcmbp64/iac/kspray7

```


cat >$(brew --prefix)/etc/dnsmasq.d/kspray5 <<EOF
address=/m0.kspray5.mbp/192.168.56.50
address=/w1.kspray5.mbp/192.168.56.51
address=/w2.kspray5.mbp/192.168.56.52
address=/w3.kspray5.mbp/192.168.56.53
address=/.ingress.kspray5.mbp/192.168.56.54
address=/padl.kspray5.mbp/192.168.56.55
address=/ldap.kspray5.mbp/192.168.56.56
address=/.tcp3.kspray5.mbp/192.168.56.57
address=/first.pool.kspray5.mbp/192.168.56.54
address=/last.pool.kspray5.mbp/192.168.56.57
EOF

sudo brew services restart dnsmasq

sudo killall -HUP mDNSResponder

```


Ensure:

```
$ kubectx
kubernetes-admin@kspray5.mbp
```

If not:

```
export KUBECONFIG=/Users/sa/dev/d3/git/ezcmbp64/iac/kspray5/build/config
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
--path=clusters/kubespray/mbp64/kspray5/flux


```


```
kubectl sk init https://skas.ingress.kspray5.mbp --force
```
