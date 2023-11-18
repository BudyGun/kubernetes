# kubernetes

Установка kubernetes: https://www.heyvaldemar.net/ustanovka-kubernetes-na-ubuntu-server-22-04-lts/

Обратите внимание, вам потребуется открыть следующие порты TCP для доступа к вашему серверу:

Kubernetes Master (Control Plane):

TCP порт 6443 - для работы Kubernetes API.
TCP порт 2379-2380 - для работы etcd server client API.
TCP порт 10250 - для работы Kubelet API.
TCP порт 10259 - для работы kube-scheduler.
TCP порт 10257 - для работы kube-controller-manager.
Kubernetes Worker:

TCP порт 0250 - для работы Kubelet API.
TCP порт 30000-32767 - для работы NodePort Services.

Мы будем рассматривать установку одного сервера с ролью Master и одного сервера с ролью Worker. В дальнейшем вы сможете самостоятельно добавить необходимое количество серверов для обеспечения высокой доступности.

Подключаемся к серверу, на который планируется установить роль Kubernetes Master.

Присвоим имя серверу с помощью команды:

```
sudo hostnamectl set-hostname kubernetes-master-1.heyvaldemar.net
```
