# kubernetes

Установка kubernetes: https://www.heyvaldemar.net/ustanovka-kubernetes-na-ubuntu-server-22-04-lts/

Должны быт открыты следующие порты TCP для доступа к серверу:

### Kubernetes Master (Control Plane):

TCP порт 6443 - для работы Kubernetes API.  
TCP порт 2379-2380 - для работы etcd server client API.  
TCP порт 10250 - для работы Kubelet API.  
TCP порт 10259 - для работы kube-scheduler.  
TCP порт 10257 - для работы kube-controller-manager.  

### Kubernetes Worker:  

TCP порт 0250 - для работы Kubelet API.  
TCP порт 30000-32767 - для работы NodePort Services.  

### Пример установки одного сервера с ролью Master и одного сервера с ролью Worker.  

10.0.5.140 - kubernetes-master-1.heyvaldemar.net  
10.0.6.19 - kubernetes-worker-1.heyvaldemar.net  

Подключаемся к серверу, на который планируется установить роль Kubernetes Master.

Присвоим имя серверу с помощью команды:
```
sudo hostnamectl set-hostname kubernetes-master-1.heyvaldemar.net
```  

Далее в файл “/etc/hosts” добавим IP-адрес и имя сервера с ролью Master с помощью команды:  
```
echo "10.0.5.140 kubernetes-master-1.heyvaldemar.net kubernetes-master-1" | sudo tee -a /etc/hosts
```

В файл “/etc/hosts” добавим IP-адрес и имя сервера с ролью Worker с помощью команды: 
```
echo "10.0.6.19 kubernetes-worker-1.heyvaldemar.net kubernetes-worker-1" | sudo tee -a /etc/hosts
```

Перезапускаю службу hostamed, чтобы внесенные изменения для имени сервера вступили в силу, с помощью команды:
```
sudo systemctl restart systemd-hostnamed
```

Проверяю корректность имени сервера с помощью команды:
```
hostname
```

Заменю текущий процесс оболочки на новый с помощью команды:
```
exec bash
```

Оттключаю файл подкачки с помощью команды:
```
sudo swapoff -a
```

Чтобы файл подкачи был отключен и после перезагрузки, выполню команду:
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Загружаю модули ядра:
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```


