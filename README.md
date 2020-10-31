# kubeadm-kubernetes

K8S 문서:[External etcd topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#external-etcd-topology)

![kubeadm HA topology - external etcd](https://d33wubrfki0l68.cloudfront.net/ad49fffce42d5a35ae0d0cc1186b97209d86b99c/5a6ae/images/kubeadm/kubeadm-ha-topology-external-etcd.svg)
---

## 최소 사양

- OS
  - Ubuntu 16.04+
  - Debian 9+
  - CentOS 7
  - Red Hat Enterprise Linux (RHEL) 7
  - Fedora 25+
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1800.6.0)
- 2 GB 이상 RAM
- 2 CPU 이상
- 원활한 네트워크 연결
- 유일한 hostname, MAC 주소, product_uuid
- 특정 포트 개방
- **반드시** Swap disabled

---

## 구축 순서

1. Load Balancer
2. Etcd cluster
3. Control Plane Nodes
4. Worker Nodes
5. Deploy Service

## 환경 준비

- Ubuntu 16.04.6 LTS (Xenial Xerus)
- Control Plane x3:
  - RAM: 2GB
  - CPU: 2
- Load Balancer, Etcd x3, Worker x3:
  - RAM: 1GB
  - CPU: 1
  
---

## vagrant 명령어

```bash
# VM 프로비전
vagrant up

# VM 상태
vagrant status

# VM 연결: vagrant ssh <VM 이름>
vagrant ssh controlplane-1
vagrant ssh etcd-1
vagrant ssh worker-1

# VM 초기화
vagrant destroy -f
```

---

## 클러스터 노드 연결

### Kubeadm 실행

다음 명령어를 controlplane 1번 노드에서 실행:

```bash
kubeadm init --config /root/kubeadm-config.yaml --upload-certs
```

다음 join 명령어를 저장:

```bash
Your Kubernetes control-plane has initialized successfully!

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.10.101:6443 --token dcrg34.oa95gk8yxi2a27ll \
    --discovery-token-ca-cert-hash sha256:3a25bd95d5c2648a738cda31be00bfe2f0be28b84f062d3c21226fab14f261d4 \
    --control-plane --certificate-key 769b9c69e408b389d52854dff684a0b56a8bfb988d2b49e1921d88a5259b204a

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.101:6443 --token dcrg34.oa95gk8yxi2a27ll \
    --discovery-token-ca-cert-hash sha256:3a25bd95d5c2648a738cda31be00bfe2f0be28b84f062d3c21226fab14f261d4 
```

### 클러스터 노드 연결

1. controlplane-2, controlplane-3 노드에서 control-plane 노드를 연결하는 명령어를 root 권한으로 실행
1. worker 노드를 연결하는 명령어를 worker 노드들에서 root 권한으로 실행

### kubectl 설정

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 클러스터 확인

controlplane-1번 서버에서 노드의 정보를 확인:

```bash
kubectl get nodes
```

```bash
NAME             STATUS     ROLES    AGE     VERSION
controlplane-1   NotReady   master   8m52s   v1.19.3
controlplane-2   NotReady   master   4m42s   v1.19.3
controlplane-3   NotReady   master   3m59s   v1.19.3
worker-1         NotReady   <none>   3m35s   v1.19.3
worker-2         NotReady   <none>   3m35s   v1.19.3
worker-3         NotReady   <none>   3m35s   v1.19.3
```

### CNI 플러그인 설치

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
