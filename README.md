# kubernetes sandbox

cross platform(freebsd,lin,win,mac..etc)

~~~~
>vagrant global-status
id       name            provider   state    directory
-----------------------------------------------------------------------------------------------------------
c34c93c  k8s-master01    virtualbox running  C:/multimachine/kubernetes-sandbox-remote
adb4ffe  worker01        virtualbox running  C:/multimachine/kubernetes-sandbox-remote
2e21187  worker02        virtualbox running  C:/multimachine/kubernetes-sandbox-remote
b39b49d  remotecontrol01 virtualbox running  C:/multimachine/kubernetes-sandbox-remote

>vagrant ssh remotecontrol01

vagrant@remotecontrol01:~$ history
    1  sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/1_initial.yml
    2  sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/2_kube-dependencies.yml
    3  sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/3_masters.yml
    4  sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/4_workers.yml
    5  sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/5_clients.yml (remote-control)

vagrant@remotecontrol01:~$ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
k8s-master01   Ready    master   2m29s   v1.17.0
worker01       Ready    <none>   98s     v1.17.0
worker02       Ready    <none>   99s     v1.17.0
vagrant@remotecontrol01:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-vlqhq               1/1     Running   0          2m28s
kube-system   coredns-6955765f44-xsk2r               1/1     Running   0          2m28s
kube-system   etcd-k8s-master01                      1/1     Running   0          2m44s
kube-system   kube-apiserver-k8s-master01            1/1     Running   0          2m44s
kube-system   kube-controller-manager-k8s-master01   1/1     Running   0          2m43s
kube-system   kube-flannel-ds-amd64-bldpt            1/1     Running   2          2m
kube-system   kube-flannel-ds-amd64-s7bd6            1/1     Running   0          2m28s
kube-system   kube-flannel-ds-amd64-tgkjm            1/1     Running   1          2m1s
kube-system   kube-proxy-b7c42                       1/1     Running   0          2m28s
kube-system   kube-proxy-cdl4n                       1/1     Running   0          2m
kube-system   kube-proxy-ws67k                       1/1     Running   0          2m1s
kube-system   kube-scheduler-k8s-master01            1/1     Running   0          2m44s

~~~~
~~~~
>vagrant ssh remotecontrol02
    5  sudo ansible-playbook -i /vagrant/kube-cluster/hosts /vagrant/kube-cluster/5_clients.yml (remote-control)
    [vagrant@remotecontrol02 ~]$ kubectl get nodes
    NAME           STATUS   ROLES    AGE    VERSION
    k8s-master01   Ready    master   166m   v1.17.0
    worker01       Ready    <none>   165m   v1.17.0
    worker02       Ready    <none>   165m   v1.17.0

~~~~

Controlling your cluster from machines other than the control-plane node
~~~~
vagrant@k8s-master01:~$ hostnamectl | grep "Operating System"
  Operating System: Ubuntu 19.04
vagrant@worker01:~$ hostnamectl | grep "Operating System"
    Operating System: Ubuntu 16.04.5 LTS
vagrant@worker02:~$ hostnamectl | grep "Operating System"
      Operating System: Ubuntu 18.10


[vagrant@remotecontrol01 ~]$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)


@k8s-master01:/etc/kubernetes/admin.conf .
copy the administrator kubeconfig file from your control-plane node to your workstation
scp vagrant@k8s-master01:/~admin.conf .

Install kubectl
[vagrant@remotecontrol01 ~]$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
[vagrant@remotecontrol01 ~]$ sudo yum install -y kubectl
[vagrant@remotecontrol01 ~]$ kubectl --kubeconfig ./admin.conf get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master01   Ready    master   38m   v1.15.2
worker01       Ready    <none>   31m   v1.15.2
worker02       Ready    <none>   30m   v1.15.2

~~~~
upgrade kubernetes
~~~~
vagrant ssh k8s-master01
$ kubectl get nodes


$ apt-cache madison kubelet | more
 kubelet |  1.16.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

 vagrant@k8s-master01:~$ apt-cache madison kubelet | more
    kubelet |  1.17.2-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.17.1-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.17.0-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.16.6-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
    kubelet |  1.16.5-00 | http://apt.kubernetes.io kubernetes-xenial/main amd64 Packages


kubernetes_version : "=1.17.0-00"
validated_dockerv: "=5:19.03.4~3-0~ubuntu-xenial"

https://kubernetes.io/docs/setup/release/notes/

~~~~
upgrade flannel
~~~~
Installing a pod network add-on
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

Set /proc/sys/net/bridge/bridge-nf-call-iptables to 1 by running sysctl net.bridge.bridge-nf-call-iptables=1 to pass bridged IPv4 traffic to iptables’ chains. This is a requirement for some CNI plugins to work

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network

~~~~

upgrade docker
~~~~  

vagrant@k8s-master01:~$ apt-cache madison docker-ce
 docker-ce | 5:19.03.5~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.4~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.3~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.2~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 5:19.03.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages


Update the latest validated version of Docker to 19.03 (#84476, @neolit123)
https://kubernetes.io/docs/setup/release/notes/

On each of your machines, install Docker. Version 19.03.4 is recommended, but 1.13.1, 17.03, 17.06, 17.09, 18.06 and 18.09 are known to work as well. Keep track of the latest verified Docker version in the Kubernetes release notes.
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/

~~~~

Install Docker CE
~~~~

## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common

### Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

### Add Docker apt repository.
add-apt-repository \
 "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) \
 stable"

## Install Docker CE.
apt-get update && apt-get install docker-ce=18.06.2~ce~3-0~ubuntu

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "log-driver": "json-file",
 "log-opts": {
   "max-size": "100m"
 },
 "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker

~~~~
~~~~
Swap disabled. You MUST disable swap in order for the kubelet to work properly
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-control-plane-node
~~~~
~~~~
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-control-plane-node
~~~~
~~~~
install only one pod network per cluster.
For flannel to work correctly, you must pass --pod-network-cidr=10.244.0.0/16 to kubeadm init.
Set /proc/sys/net/bridge/bridge-nf-call-iptables to 1 by running sysctl net.bridge.bridge-nf-call-iptables=1
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
~~~~
~~~~
The Inventory File

Auto-Generated Inventory
The first and simplest option is to not provide one to Vagrant at all. Vagrant will generate an inventory file encompassing all of the virtual machines it manages, and use it for provisioning machines

Static Inventory
The second option is for situations where you would like to have more control over the inventory management.
With the inventory_path option, you can reference a specific inventory resource (e.g. a static inventory file, a dynamic inventory script or even multiple inventories stored in the same directory)
https://www.vagrantup.com/docs/provisioning/ansible_intro.html
~~~~


smoke test helloworld
~~~~

vagrant@remotecontrol01:~$ cat /vagrant/kubernetes-tutorial-4/load-balancer-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  # replicas: 5
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-world
        ports:
        - containerPort: 8080

        vagrant@remotecontrol01:~$ kubectl create -f /vagrant/kubernetes-tutorial-4/load-balancer-deployment
        deployment.apps/hello-world created

        vagrant@remotecontrol01:~$ kubectl get deployments hello-world
        NAME          READY   UP-TO-DATE   AVAILABLE   AGE
        hello-world   0/1     1            0           13s

        vagrant@remotecontrol01:~$ kubectl describe deployments hello-world
        Name:                   hello-world
        Namespace:              default
        CreationTimestamp:      Tue, 27 Aug 2019 17:58:03 +0000
        Labels:                 app.kubernetes.io/name=load-balancer-example
        Annotations:            deployment.kubernetes.io/revision: 1
        Selector:               app.kubernetes.io/name=load-balancer-example
        Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
        StrategyType:           RollingUpdate
        MinReadySeconds:        0
        RollingUpdateStrategy:  25% max unavailable, 25% max surge
        Pod Template:
          Labels:  app.kubernetes.io/name=load-balancer-example
          Containers:
           hello-world:
            Image:        gcr.io/google-samples/node-hello:1.0
            Port:         8080/TCP
            Host Port:    0/TCP
            Environment:  <none>
            Mounts:       <none>
          Volumes:        <none>
        Conditions:
          Type           Status  Reason
          ----           ------  ------
          Available      True    MinimumReplicasAvailable
          Progressing    True    NewReplicaSetAvailable
        OldReplicaSets:  <none>
        NewReplicaSet:   hello-world-bbbb4c85d (1/1 replicas created)
        Events:
          Type    Reason             Age   From                   Message
          ----    ------             ----  ----                   -------
          Normal  ScalingReplicaSet  18s   deployment-controller  Scaled up replica set hello-world-bbbb4c85d to 1

        vagrant@remotecontrol01:~$ kubectl get replicasets
        NAME                    DESIRED   CURRENT   READY   AGE
        hello-world-bbbb4c85d   1         1         1       23s
        vagrant@remotecontrol01:~$ kubectl describe replicasets
        Name:           hello-world-bbbb4c85d
        Namespace:      default
        Selector:       app.kubernetes.io/name=load-balancer-example,pod-template-hash=bbbb4c85d
        Labels:         app.kubernetes.io/name=load-balancer-example
                        pod-template-hash=bbbb4c85d
        Annotations:    deployment.kubernetes.io/desired-replicas: 1
                        deployment.kubernetes.io/max-replicas: 2
                        deployment.kubernetes.io/revision: 1
        Controlled By:  Deployment/hello-world
        Replicas:       1 current / 1 desired
        Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
        Pod Template:
          Labels:  app.kubernetes.io/name=load-balancer-example
                   pod-template-hash=bbbb4c85d
          Containers:
           hello-world:
            Image:        gcr.io/google-samples/node-hello:1.0
            Port:         8080/TCP
            Host Port:    0/TCP
            Environment:  <none>
            Mounts:       <none>
          Volumes:        <none>
        Events:
          Type    Reason            Age   From                   Message
          ----    ------            ----  ----                   -------
          Normal  SuccessfulCreate  28s   replicaset-controller  Created pod: hello-world-bbbb4c85d-fgctl

        vagrant@remotecontrol01:~$ kubectl get pods --selector="app.kubernetes.io/name=load-balancer-example" --output=wide
        NAME                          READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
        hello-world-bbbb4c85d-fgctl   1/1     Running   0          45s   192.168.1.22   worker02   <none>           <none>

        vagrant@remotecontrol01:~$ kubectl expose deployment hello-world --type=NodePort --name=example-service
        service/example-service exposed
        vagrant@remotecontrol01:~$ kubectl describe services example-service
        Name:                     example-service
        Namespace:                default
        Labels:                   app.kubernetes.io/name=load-balancer-example
        Annotations:              <none>
        Selector:                 app.kubernetes.io/name=load-balancer-example
        Type:                     NodePort
        IP:                       10.100.140.243
        Port:                     <unset>  8080/TCP
        TargetPort:               8080/TCP
        NodePort:                 <unset>  31744/TCP
        Endpoints:                192.168.1.22:8080
        Session Affinity:         None
        External Traffic Policy:  Cluster
        Events:                   <none>

        vagrant@remotecontrol01:~$ kubectl describe node | grep InternalIP
          InternalIP:  192.168.50.10
          InternalIP:  192.168.50.11
          InternalIP:  192.168.50.12

        vagrant@remotecontrol01:~$ kubectl get pods -n default -o wide
        NAME                          READY   STATUS    RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
        hello-world-bbbb4c85d-fgctl   1/1     Running   0          2m54s   192.168.1.22   worker02   <none>           <none>

        vagrant@remotecontrol01:~$ kubectl describe pod hello-world-bbbb4c85d-fgctl | grep "Node:"
        Node:           worker02/192.168.50.12

        vagrant@remotecontrol01:~$ curl http://192.168.50.12:31744
        Hello Kubernetes!

        Browse http://192.168.50.12:31744

        kubectl delete -f /vagrant/app/helloworld/load-balancer-deployment

        vagrant@remotecontrol01:~$ kubectl delete services example-service
        service "example-service" deleted

        vagrant@remotecontrol01:~$ kubectl delete deployment hello-world
        deployment.extensions "hello-world" deleted

~~~~
smoke test jenkins (single file)
~~~~
vagrant@remotecontrol01:~$  kubectl create -f /vagrant/app/jenkins/1/jenkins-deployment.yaml
deployment.apps/jenkins-deployment created
service/jenkins-svc created

vagrant@remotecontrol01:~$ kubectl get deployments -o wide --watch
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                SELECTOR
jenkins-deployment   1/1     1            1           38s   jenkins-svc   jenkins/jenkins:lts   app=jenkins
vagrant@remotecontrol01:~$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
vagrant@remotecontrol01:~$ kubectl describe services jenkins-svc
Name:                     jenkins-svc
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=jenkins
Type:                     NodePort
IP:                       10.101.212.26
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30000/TCP
Endpoints:                192.168.1.3:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
vagrant@remotecontrol01:~$ kubectl describe pod jenkins-deployment-f6f666f9-lv8h7 | grep "Node:"
Node:           worker02/192.168.50.12

vagrant@remotecontrol01:~$ curl http://192.168.50.12:30000                                                                                                                                                                                                    
Browse http://192.168.50.12:30000

vagrant@remotecontrol01:~$ kubectl delete -f /vagrant/app/jenkins/1/jenkins-deployment.yaml
deployment.apps "jenkins-deployment" deleted
service "jenkins-svc" deleted

~~~~
smoke test jenkins (seperate files)
~~~~
vagrant@remotecontrol01:~$ kubectl create namespace jenkins
namespace/jenkins created
vagrant@remotecontrol01:~$ kubectl create -f /vagrant/app/jenkins/2/jenkins-deployment.yaml --namespace=jenkins
deployment.apps/jenkins-deployment created
vagrant@remotecontrol01:~$ kubectl create -f /vagrant/app/jenkins/2/jenkins-service.yaml --namespace=jenkins
service/jenkins created
vagrant@remotecontrol01:~$ kubectl get deployments  --all-namespaces -o wide
NAMESPACE     NAME                 READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                     SELECTOR
jenkins       jenkins-deployment   1/1     1            1           24s   jenkins      jenkins/jenkins:lts        app=jenkins
kube-system   coredns              2/2     2            2           23h   coredns      k8s.gcr.io/coredns:1.6.5   k8s-app=kube-dns
vagrant@remotecontrol01:~$ kubectl get deployments  -n jenkins -o wide
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                SELECTOR
jenkins-deployment   1/1     1            1           32s   jenkins      jenkins/jenkins:lts   app=jenkins


vagrant@remotecontrol01:~$ kubectl  describe deployments --namespace=jenkins
Name:                   jenkins-deployment
Namespace:              jenkins
CreationTimestamp:      Tue, 27 Aug 2019 18:21:34 +0000
Labels:                 app=jenkins
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=jenkins
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=jenkins
  Containers:
   jenkins:
    Image:        jenkins:2.60.3
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   jenkins-deployment-868cc579df (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  28s   deployment-controller  Scaled up replica set jenkins-deployment-868cc579df to 1


  vagrant@remotecontrol01:~$ kubectl get pods -n jenkins
  NAME                                  READY   STATUS    RESTARTS   AGE
  jenkins-deployment-868cc579df-mv8k8   1/1     Running   0          9m24s

  vagrant@remotecontrol01:~$ kubectl describe pod jenkins-deployment-868cc579df-mv8k8 -n jenkins | grep "Node:"
Node:           worker01/192.168.50.11

vagrant@remotecontrol01:~$ kubectl get services -n jenkins
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins   NodePort   10.108.24.206   <none>        8080:30000/TCP   10m

vagrant@remotecontrol01:~$ kubectl describe -n jenkins services jenkins
Name:                     jenkins
Namespace:                jenkins
Labels:                   <none>
Annotations:              <none>
Selector:                 app=jenkins
Type:                     NodePort
IP:                       10.108.24.206
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30000/TCP
Endpoints:                192.168.2.25:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

vagrant@remotecontrol01:~$ curl http://192.168.50.11:30000
<html><head><meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script></head><body style='background-color:white; color:white;'>


Authentication required
<!--
You are authenticated as: anonymous
Groups that you are in:

Permission you need to have (but didn't): hudson.model.Hudson.Administer
-->

</body></html>

vagrant@remotecontrol01:~$ kubectl logs -n jenkins jenkins-deployment-868cc579df-mv8k8
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

86f13cf78d6841688d689dc3987230f4

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

vagrant@remotecontrol01:~$ kubectl logs -n jenkins jenkins-deployment-868cc579df-nmh5w | grep "An admin user has been created and a password generated"
Jenkins initial setup is required. An admin user has been created and a password generated.

Browse
http://192.168.50.11:30000

# get the password within the pod.
vagrant@remotecontrol01:~$ kubectl exec -it jenkins-deployment-868cc579df-mv8k8 bash
Error from server (NotFound): pods "jenkins-deployment-868cc579df-mv8k8" not found

vagrant@remotecontrol01:~$ kubectl exec -it jenkins-deployment-868cc579df-mv8k8 -n jenkins bash
jenkins@jenkins-deployment-868cc579df-mv8k8:/$ cat /var/jenkins_home/secrets/initialAdminPassword
86f13cf78d6841688d689dc3987230f4
jenkins@jenkins-deployment-868cc579df-mv8k8:/$ exit
exit
vagrant@remotecontrol01:~$
vagrant@remotecontrol01:~$ cat /var/jenkins_home/secrets/initialAdminPassword
cat: /var/jenkins_home/secrets/initialAdminPassword: No such file or directory


vagrant@remotecontrol01:~$ kubectl get services -n jenkins
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins   NodePort   10.108.24.206   <none>        8080:30000/TCP   17m
vagrant@remotecontrol01:~$ kubectl delete services -n jenkins jenkins
service "jenkins" deleted


vagrant@remotecontrol01:~$ kubectl get deployment -n jenkins
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
jenkins-deployment   1/1     1            1           18m


vagrant@remotecontrol01:~$ kubectl delete -f /vagrant/kubernetes-jenkins-1/jenkins-deployment.yaml --namespace=jenkins
vagrant@remotecontrol01:~$ kubectl delete -f /vagrant/kubernetes-jenkins-1/jenkins-service.yaml --namespace=jenkins

~~~~
