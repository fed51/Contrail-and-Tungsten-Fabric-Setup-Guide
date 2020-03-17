
# Disclaimer


***This guide is meant to be a helpful reference to install Kubernetes. It is NOT intended to be an authoritative guide for Kubernetes install. It is merely steps that we use everyday and happens to work for us.***

Please refer to canonical documentation from Kubernetes community  [here](https://kubernetes.io/docs/setup/)

# Installing Kuberntes on Ubuntu hosts

## Installing Kubernetes on Master node

1. Prepare the node by running the following pre-requisites
```
swapoff -a
sudo apt-get update
sudo apt-get install -y curl software-properties-common apt-transport-https
```

2. Install Docker
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce # Install docker
sudo service docker status  # Verify that docker service is running
```

3. Add Kubernetes repo
```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```
4. Install Kubernetes components
```
sudo apt-get install -y kubectl kubelet kubeadm
```
5. Config kubeadm if NodePort service is needed.(OPTIONAL)

   [Kube-proxy config](nodeport-kube-proxy-setup.md)

6. Create K8s cluster

  If kubeadm was configured for nodeport (via step 5):
```
kubeadm init --config config.yaml
```
  else
```
kubeadm init
```
7. Once "kubeadm init" completes, save the "join" command that will be printed on the shell, to a file of your choice. This will be needed to add new nodes to your cluster.

```
example:
"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
```
8. Run the following commands to initially kubernetes command line
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

9. Disable firewalld
```
sysctl -w net.bridge.bridge-nf-call-iptables=1
systemctl stop firewalld; systemctl disable firewalld
```

10. Disable networking on Master if desired. (OPTIONAL)

  Disabling networking on the master will result not workloads not being scheduled on the master node.

   For kubeadm version >= 1.11

  ```
  a. sudo vi /var/lib/kubelet/kubeadm-flags.env

  b. Remove the following 3 arguments from KUBELET_KUBEADM_ARGS variable:
          --cni-bin-dir=/opt/cni/bin
          --cni-conf-dir=/etc/cni/net.d
          --network-plugin=cni

  c. Restart kubelet Service

      systemctl enable kubelet && systemctl start kubelet
  ```

  For kubeadm version < 1.11

  ```
  a. sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  b. Comment out

  "#Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
  sudo systemctl daemon-reload;sudo service kubelet restart

  c. Restart kubelet Service

     systemctl enable kubelet && systemctl start kubelet
  ```

## Installing Kubernetes on Compute (k8s slave/minion) nodes

  1. Prepare the node by running the following pre-requisites
  ```
  swapoff -a
  sudo apt-get update
  sudo apt-get install -y curl software-properties-common apt-transport-https
  ```

  2. Install Docker
  ```
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  sudo apt-get update
  sudo apt-get install -y docker-ce # Install docker
  sudo service docker status  # Verify that docker service is running
  ```

  3. Add Kubernetes repo
  ```
  sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update
  ```
  4. Install Kubernetes components
  ```
  sudo apt-get install -y kubectl kubelet kubeadm
  ```
  5. Disable firewalld
  ```
  sysctl -w net.bridge.bridge-nf-call-iptables=1
  systemctl stop firewalld; systemctl disable firewalld
  ```

  6. Join the Master node

  Copy paste the "join" command you saved from Step-7 of instructions for installation of K8s on master.
  ```
  "kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
  ```

# Installing Kuberntes on Centos hosts

## Installing Kubernetes on Master node

**(Centos 7.4 - 3.10.0-862.3.2 kernel)**

1. Prepare the node by running the following pre-requisites
  ```
sudo setenforce 0
swapoff -a
```
2. Install Docker

  ```
sudo yum install -y docker
systemctl enable docker.service;service docker start
```

3. Add Kubernetes repo
  ```
cat << EOF >> /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
4. Install Kubernetes components
  ```
yum update -y; yum install -y kubelet kubeadm
```

5. Config kubeadm if NodePort service is needed.(OPTIONAL)

   [Kube-proxy config](nodeport-kube-proxy-setup.md)

6. Create K8s cluster

  If kubeadm was configured for nodeport (via step 5):
  ```
kubeadm init --config config.yaml
```
  else
```
kubeadm init
```
7. Once "kubeadm init" completes, save the "join" command that will be printed on the shell, to a file of your choice. This will be needed to add new nodes to your cluster.

  ```
example:
"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
```
8. Run the following commands to initially kubernetes command line
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
9. Disable firewalld
```
sysctl -w net.bridge.bridge-nf-call-iptables=1
systemctl stop firewalld; systemctl disable firewalld
```
10. Disable networking on Master if desired. (OPTIONAL)

  Disabling networking on the master will result not workloads not being scheduled on the master node.

   For kubeadm version >= 1.11

  ```
  a. sudo vi /var/lib/kubelet/kubeadm-flags.env

  b. Remove the following 3 arguments from KUBELET_KUBEADM_ARGS variable:
          --cni-bin-dir=/opt/cni/bin
          --cni-conf-dir=/etc/cni/net.d
          --network-plugin=cni

  c. Restart kubelet Service

      systemctl enable kubelet && systemctl start kubelet
  ```

  For kubeadm version < 1.11

  ```
  a. sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  b. Comment out

  "#Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
  sudo systemctl daemon-reload;sudo service kubelet restart

  c. Restart kubelet Service

     systemctl enable kubelet && systemctl start kubelet
  ```

## Installing Kubernetes on Compute (k8s slave/minion) nodes
**(Centos 7.4 - 3.10.0-862.3.2 kernel)**

1. Prepare the node by running the following pre-requisites
  ```
sudo setenforce 0
swapoff -a
```

2. Install Docker
  ```
sudo yum install -y docker
systemctl enable docker.service;service docker start
```
3. Add Kubernetes repo
  ```
cat << EOF >> /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
4. Install Kubernetes components
  ```
yum update -y; yum install -y kubelet kubeadm
```
5. Disable firewalld
  ```
sysctl -w net.bridge.bridge-nf-call-iptables=1
systemctl stop firewalld; systemctl disable firewalld
```
6. Join the Master node
Copy paste the "join" command you saved from Step-9 of instructions for installation of K8s on master.
```
"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
```

Contrail CNI can be installed on a Kubernetes cluster through multiple provisioning schemes.

![Contrail Standalone Solution](images/standalone-kubernetes.png)

This wiki will describe the most simplest of all: **A single yaml based install**

# Pre-requisites
1. **A running Kubernetes cluster**

See above.

2. Add Contrail repository as insecure repository in docker.
   *** NOTE: This will need to be done on all nodes of your Kubernetes cluster ***
```
Step a:
cat <<EOF >>/etc/docker/daemon.json
{
"insecure-registries": ["ci-repo.englab.juniper.net:5010"]
}
EOF

Step b:
service docker restart
```

3. Make sure that kubernetes master node ip has an entry in /etc/hosts file that resolves to
the hostname.
```
File: /etc/hosts

x.x.x.x hostname hostname.fqname
```

4. Docker version on all nodes should be >= 1.24

# Installation
  Installation is a **1**-step process.

  Note: Replace x.x.x.x with the IP of your Kubernetes Master node.

## Contrail on CentOS

```
K8S_MASTER_IP=x.x.x.x; \
    CONTRAIL_REPO="ci-repo.englab.juniper.net:5010"; \
    CONTRAIL_RELEASE="latest"; \
    mkdir -pm 777 /var/lib/contrail/kafka-logs; \
    TMPLT="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-centos.yaml"; \
    PATCH="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-centos.patch"; \
    FILE=$(basename -- "$TMPLT"); \
    curl $TMPLT -o /tmp/${TMPLT##*/}; \
    curl $PATCH -o /tmp/${PATCH##*/}; \
    patch /tmp/$FILE /tmp/${PATCH##*/}; \
    sed -i "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; \
        s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; \
        s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" \
        /tmp/$FILE; \
    kubectl apply -f /tmp/$FILE; \
    rm /tmp/${FILE%.*}*
```

## Contrail on Ubuntu

```
K8S_MASTER_IP=x.x.x.x; \
    CONTRAIL_REPO="ci-repo.englab.juniper.net:5010"; \
    CONTRAIL_RELEASE="latest"; \
    mkdir -pm 777 /var/lib/contrail/kafka-logs; \
    TMPLT="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.yaml"; \
    PATCH="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.patch"; \
    FILE=$(basename -- "$URL"); \
    curl $TMPLT -o /tmp/${TMPLT##*/}; \
    curl $PATCH -o /tmp/${PATCH##*/}; \
    patch /tmp/$FILE /tmp/${PATCH##*/}; \
    sed -i "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; \
        s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; \
        s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" \
        /var/$FILE; \
    kubectl apply -f /var/$FILE; \
    rm /var/${FILE%.*}*
```

## Tungsten Fabric on CentOS

```
K8S_MASTER_IP=x.x.x.x; \
    CONTRAIL_REPO="docker.io\/opencontrailnightly"; \
    CONTRAIL_RELEASE="latest"; \
    mkdir -pm 777 /var/lib/contrail/kafka-logs; \
    TMPLT="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-centos.yaml"; \
    PATCH="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-centos.patch"; \
    FILE=$(basename -- "$TMPLT"); \
    curl $TMPLT -o /tmp/${TMPLT##*/}; \
    curl $PATCH -o /tmp/${PATCH##*/}; \
    patch /tmp/$FILE /tmp/${PATCH##*/}; \
    sed -i "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; \
        s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; \
        s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" \
        /tmp/$FILE; \
    kubectl apply -f /tmp/$FILE; \
    rm /tmp/${FILE%.*}*
```

## Tungsten Fabric on Ubuntu

```
K8S_MASTER_IP=x.x.x.x; \
    CONTRAIL_REPO="docker.io\/opencontrailnightly"; \
    CONTRAIL_RELEASE="latest"; \
    mkdir -pm 777 /var/lib/contrail/kafka-logs; \
    TMPLT="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.yaml"; \
    PATCH="https://raw.githubusercontent.com/fed51/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.patch"; \
    FILE=$(basename -- "$TMPLT"); \
    curl $TMPLT -o /tmp/${TMPLT##*/}; \
    curl $PATCH -o /tmp/${PATCH##*/}; \
    patch /tmp/$FILE /tmp/${PATCH##*/}; \
    sed -i "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; \
        s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; \
        s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" \
        /tmp/$FILE; \
    kubectl apply -f /tmp/$FILE; \
    rm /tmp/${FILE%.*}*
```

# What just happened ?

**Hurray! Welcome to Contrail.**

1. You installed Contrail CNI in your Kubernetes node. If new compute nodes are added to your Kubernetes cluster, Contrail CNI will be propogated to them auto-magically as it is backed by a Kubernetes DaemaonSet.

2. You installed entire Contrail Networking suite with rich Networking, Analytics, Security, Visualization functions, to name a few.

3. Contrail UI is available on port 8143 of your node.  Feel free to play around. [About Contrail](https://www.juniper.net/documentation/en_US/release-independent/contrail/information-products/pathway-pages/index.html)
```
https://x.x.x.x:8143
Default credentials: admin/contrail123
```
# Check Contrail Status

You can get the status of Contrail components, by running "contrail-status" command line tool in your Kubernetes master node. This will list all Contrail components running in your system.
```
[root@foo ~]# contrail-status
Pod         Service         Original Name                          State    Status         
            zookeeper       contrail-external-zookeeper            running  Up 35 minutes  
analytics   alarm-gen       contrail-analytics-alarm-gen           running  Up 35 minutes  
analytics   api             contrail-analytics-api                 running  Up 35 minutes  
analytics   collector       contrail-analytics-collector           running  Up 35 minutes  
analytics   nodemgr         contrail-nodemgr                       running  Up 33 minutes  
analytics   query-engine    contrail-analytics-query-engine        running  Up 35 minutes  
analytics   snmp-collector  contrail-analytics-snmp-collector      running  Up 35 minutes  
analytics   topology        contrail-analytics-topology            running  Up 34 minutes  
config      api             contrail-controller-config-api         running  Up 35 minutes  
config      cassandra       contrail-external-cassandra            running  Up 35 minutes  
config      device-manager  contrail-controller-config-devicemgr   running  Up 35 minutes  
config      nodemgr         contrail-nodemgr                       running  Up 33 minutes  
config      rabbitmq        contrail-external-rabbitmq             running  Up 35 minutes  
config      schema          contrail-controller-config-schema      running  Up 35 minutes  
config      svc-monitor     contrail-controller-config-svcmonitor  running  Up 35 minutes  
control     control         contrail-controller-control-control    running  Up 35 minutes  
control     dns             contrail-controller-control-dns        running  Up 35 minutes  
control     named           contrail-controller-control-named      running  Up 35 minutes  
control     nodemgr         contrail-nodemgr                       running  Up 33 minutes  
database    cassandra       contrail-external-cassandra            running  Up 35 minutes  
database    kafka           contrail-external-kafka                running  Up 35 minutes  
database    nodemgr         contrail-nodemgr                       running  Up 34 minutes  
kubernetes  kube-manager    contrail-kubernetes-kube-manager       running  Up 35 minutes  
vrouter     agent           contrail-vrouter-agent                 running  Up 34 minutes  
vrouter     nodemgr         contrail-nodemgr                       running  Up 33 minutes  
webui       job             contrail-controller-webui-job          running  Up 35 minutes  
webui       web             contrail-controller-webui-web          running  Up 35 minutes  

WARNING: container with original name 'contrail-external-zookeeper' have Pod os Service empty. Pod: '' / Service: 'zookeeper'. Please pass NODE_TYPE with pod name to container's env

vrouter kernel module is PRESENT
== Contrail control ==
control: active
nodemgr: initializing (NTP state unsynchronized. ) . <-- Safe to ignore
named: active
dns: active

== Contrail kubernetes ==
kube-manager: active

== Contrail database ==
kafka: active
nodemgr: initializing (NTP state unsynchronized. ) . <-- Safe to ignore
zookeeper: inactive                                  <-- Safe to ignore
cassandra: active

== Contrail analytics ==
nodemgr: initializing (NTP state unsynchronized. ) . <-- Safe to ignore
api: active
collector: active
query-engine: active
alarm-gen: active

== Contrail webui ==
web: active
job: active

== Contrail vrouter ==
nodemgr: initializing (NTP state unsynchronized. )    <-- Safe to ignore
agent: active

== Contrail config ==
api: active
zookeeper: inactive                                   <-- Safe to ignore
svc-monitor: active
nodemgr: initializing (NTP state unsynchronized. ) .  <-- Safe to ignore
device-manager: active
cassandra: active
rabbitmq: active
schema: active

```

# Manually Applying Patch

### Basic

Download the yaml and patch files to apply the patch locally.

Run the following command too apply the patch:

CentOS:
```
$ patch -b < contrail-single-step-cni-install-centos.patch
```

Ubuntu:
```
$ patch -b < contrail-single-step-cni-install-ubuntu.patch
```

# Get to know Contrail more

[All about Contrail](https://www.juniper.net/documentation/en_US/release-independent/contrail/information-products/pathway-pages/index.html)

[Contrail and Kubernetes Intro](https://github.com/Juniper/contrail-controller/wiki/Kubernetes)

[Install Kubernetes using Kubeadm](https://github.com/Juniper/contrail-controller/wiki/Install-K8s-using-Kubeadm)

[Provision Contrail Kubernetes Cluster in Non-nested mode](https://github.com/Juniper/contrail-ansible-deployer/wiki/Provision-Contrail-Kubernetes-Cluster-in-Non-nested-Mode)
