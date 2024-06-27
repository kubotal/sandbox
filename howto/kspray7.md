

Copied from ezcmbp64/iac/kspray7

```

cat >$(brew --prefix)/etc/dnsmasq.d/kspray7 <<EOF
address=/n0.kspray7.mbp/192.168.56.70
address=/.ingress.kspray7.mbp/192.168.56.71
address=/padl.kspray7.mbp/192.168.56.72
address=/.tcp2.kspray7.mbp/192.168.56.73
address=/.tcp3.kspray7.mbp/192.168.56.74
address=/first.pool.kspray7.mbp/192.168.56.71
address=/last.pool.kspray7.mbp/192.168.56.74
EOF

sudo brew services restart dnsmasq

```


Ensure:

```
$ kubectx
kubernetes-admin@kspray7.mbp
```

If not:

```
export KUBECONFIG=/Users/sa/dev/d3/git/ezcmbp64/iac/kspray7/build/config
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
--path=clusters/kubespray/mbp64/kspray7/flux


```


```
kubectl sk init https://skas.ingress.kspray7.mbp --force
```
