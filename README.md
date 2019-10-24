# Launch a rancherOS on openstack
Attach configuration drive and set content to
```yaml
#cloud-config
rancher:
  resize_device: /dev/vda
  dev: /dev/vda
  autoformat: /dev/vda
  console: debian
```

# Install or upgrade rancher 2.x on rancherOS
```bash
docker run -d -p80:80 -p443:443 \
	-v rancher-data:/var/lib/rancher \
	-v rancher-mysql-data:/var/lib/mysql \
	-v rancher-mysql-log:/var/log/mysql \
	--restart=unless-stopped \
	--name=rancher-server-2.2.6 \
	rancher/rancher:v2.2.6 \
	--acme-domain rancher.rebirth.tech
```

# Add a master node
```bash
docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.6 --server https://rancher.rebirth.tech --token $token --address awspublic --internal-address awslocal --etcd --controlplane
```

# Add a worker node
```bash
export token=
docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.6 --server https://rancher.rebirth.tech --token $token --address awspublic --internal-address awslocal --worker
# longhorn
sudo apt update
sudo apt install -y open-iscsi
```

# Install HELM
```bash
kubectl -n kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:tiller

helm init --service-account tiller
```

# Create an openstack anti-affinity server group
`openstack --os-auth-url=https://identity.api.r1.nxs.enix.io/v3 --os-identity-api-version=3 --os-username=abuisine --os-user-domain-name=Default --os-project-name=rebirthtechnologies server group create --policy anti-affinity workers.k8s.rebirth.tech`