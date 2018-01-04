# Deploy preparation for base images on gcr and heketi
## pull and save image on remote machine
```
for img in kube-proxy-amd64 kube-apiserver-amd64 kube-controller-manager-amd64 kube-scheduler-amd64; do
    docker save  gcr.io/google_containers/$img:v1.9.0  -o $img.tar
done

docker save gcr.io/google_containers/etcd-amd64:3.1.10 -o etcd.tar
docker save gcr.io/google_containers/pause-amd64:3.0 -o pause.tar

for img in kube-dns dnsmasq-nanny sidecar; do
    docker pull gcr.io/google_containers/k8s-dns-$img-amd64:1.14.7
    docker save gcr.io/google_containers/k8s-dns-$img-amd64:1.14.7 -o $img.tar
done
docker save heketi/heketi:dev -o heketi.tar
docker save gluster/gluster-centos -o gluster.tar
```

## load and push to local repo
```
: ${LREPO:=192.168.10.1:5000}
for img in *.tar; do
    docker load -i $img
done
for img in kube-proxy-amd64 kube-apiserver-amd64 kube-controller-manager-amd64 kube-scheduler-amd64; do
    docker tag  gcr.io/google_containers/$img:v1.9.0  $LREPO/$img:v1.9.0
    docker push $LREPO/$img:v1.9.0
done

docker tag gcr.io/google_containers/etcd-amd64:3.1.10 $LREPO/etcd-amd64:3.1.10
docker push $LREPO/etcd-amd64:3.1.10
docker tag gcr.io/google_containers/pause-amd64:3.0 $LREPO/pause-amd64:3.0
docker push $LREPO/pause-amd64:3.0

for img in kube-dns dnsmasq-nanny sidecar; do
    docker tag gcr.io/google_containers/k8s-dns-$img-amd64:1.14.7 $LREPO/google_containers/k8s-dns-$img-amd64:1.14.7
    docker push $LREPO/google_containers/k8s-dns-$img-amd64:1.14.7
done
docker tag heketi/heketi:dev $LREPO/heketi/heketi:dev
docker push $LREPO/heketi/heketi:dev
docker tag gluster/gluster-centos $LREPO/gluster/gluster-centos
docker push $LREPO/gluster/gluster-centos
```
## Fixed join command
```
kubeadm join --ignore-preflight-errors=all --discovery-token-unsafe-skip-ca-verification  --token=abcdef.1234567890abcdef 192.168.10.90:6443
```

## Notes
### command that will run on master for starting proxy
```
docker run -d --name gcr_proxy -p 80:80 --restart always -v  /vagrant/nginx.conf:/etc/nginx/nginx.conf:ro  \
gcr.io/google_containers/nginx-slim:0.8
```
