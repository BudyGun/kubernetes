# kubernetes

### Пример установки одного сервера с ролью Master и одного сервера с ролью Worker.  

10.0.5.140 - kubernetes-master-1.heyvaldemar.net  
10.0.6.19 - kubernetes-worker-1.heyvaldemar.net  

### Настройка Kubernetes Master
Подключаюсь к серверу, на который планируется установить роль Kubernetes Master.

Присвою имя серверу с помощью команды:
```
sudo hostnamectl set-hostname kubernetes-master-1.heyvaldemar.net
```  

Далее в файл “/etc/hosts” добавляю IP-адрес и имя сервера с ролью Master с помощью команды:  
```
echo "10.0.5.140 kubernetes-master-1.heyvaldemar.net kubernetes-master-1" | sudo tee -a /etc/hosts
```

В файл “/etc/hosts” добавляю IP-адрес и имя сервера с ролью Worker с помощью команды: 
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

Загружаю модуль “overlay” с помощью команды:
```
sudo modprobe overlay
```

Загружаю модуль “br_netfilter” с помощью команды:
```
sudo modprobe br_netfilter
```

Устанавливаю параметры ядра для Kubernetes с помощью команды:
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

Применяю внесенные изменения с помощью команды:
```
sudo sysctl --system
```
# Установка и настройка Docker  
```
sudo apt update
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable"| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce -y
```
Добавить пользователя в группу докер:
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

Проверка статуса docker:
```
sudo systemctl status docker
```

Проверяю работу докера:
```
docker run hello-world
```
Вижу приветствие докера, докер работает корректно.  

Теперь устанавливаю пакеты, необходимые для работы Kubernetes, с помощью команды:
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates containerd.io
```

Настраиваю containerd. Выполняем команду:
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
```

Выполняю команду:
```
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Перезапускаю containerd, чтобы применить внесенные изменения, с помощью команды:
```
sudo systemctl restart containerd
```

Включаю автозапуск сервиса containerd при запуске операционной системы с помощью команды:
```
sudo systemctl enable containerd
```

Добавляю официальный ключ Kubernetes с помощью команды:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Подключаю репозиторий Kubernetes с помощью команды:
```
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

На момент написания этого руководства Xenial является актуальным репозиторием Kubernetes, но когда репозиторий будет доступен для Ubuntu 22.04 (Jammy Jellyfish), то нужно будет заменить слово в команде выше с “xenial” на “jammy”.

Обновляю локальный индекс пакетов до последних изменений в репозиториях с помощью команды:
```
sudo apt update
```

Установливаю пакеты kubelet, kubeadm и kubectl с помощью команды:
```
sudo apt install -y kubelet kubeadm kubectl
```

Нужно запретить автоматическое обновление и удаление установленных пакетов с помощью команды:
```
sudo apt-mark hold kubelet kubeadm kubectl
```

Запускаю инициализацию кластера Kubernetes с помощью команды:
```
sudo kubeadm init --control-plane-endpoint=kubernetes-master-1.heyvaldemar.net
```


Для добавления еще одного сервера в кластер потребуется проделать такую же работу по установке и настройке сервера, а затем выполнить команду kubeadm join с соответствующим токеном для сервера с ролью Master или Worker.


Далее необходимо выполнить несколько команд, чтобы начать взаимодействие с кластером.  
Выполняем команду:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Посмотреть адреса мастера и сервисов можно с помощью команды:
```
kubectl cluster-info
```

Посмотреть список всех узлов в кластере и статус каждого узла с помощью команды:
```
kubectl get nodes
```


### Настройка Kubernetes Worker

Присвою имя worker'у с помощью команды:
```
sudo hostnamectl set-hostname kubernetes-worker-1.heyvaldemar.net
```

Далее в файл “/etc/hosts” добавляю IP-адрес и имя сервера с ролью Master с помощью команды:  
```
echo "10.0.5.140 kubernetes-master-1.heyvaldemar.net kubernetes-master-1" | sudo tee -a /etc/hosts
```

В файл “/etc/hosts” добавляю IP-адрес и имя сервера с ролью Worker с помощью команды: 
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

Загружаю модуль “overlay” с помощью команды:
```
sudo modprobe overlay
```

Загружаю модуль “br_netfilter” с помощью команды:
```
sudo modprobe br_netfilter
```

Устанавливаю параметры ядра для Kubernetes с помощью команды:
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

Применяю внесенные изменения с помощью команды:
```
sudo sysctl --system
```
# Установка и настройка Docker  
```
sudo apt update
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable"| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce -y
```
Добавить пользователя в группу докер:
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

Проверка статуса docker:
```
sudo systemctl status docker
```

Проверяю работу докера:
```
docker run hello-world
```
Вижу приветствие докера, докер работает корректно.  


Теперь устанавливаю пакеты, необходимые для работы Kubernetes, с помощью команды:
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates containerd.io
```

Настраиваю containerd. Выполняем команду:
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
```

Выполняю команду:
```
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Перезапускаю containerd, чтобы применить внесенные изменения, с помощью команды:
```
sudo systemctl restart containerd
```

Включаю автозапуск сервиса containerd при запуске операционной системы с помощью команды:
```
sudo systemctl enable containerd
```

Добавляю официальный ключ Kubernetes с помощью команды:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Подключаю репозиторий Kubernetes с помощью команды:
```
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

На момент написания этого руководства Xenial является актуальным репозиторием Kubernetes, но когда репозиторий будет доступен для Ubuntu 22.04 (Jammy Jellyfish), то нужно будет заменить слово в команде выше с “xenial” на “jammy”.

Обновляю локальный индекс пакетов до последних изменений в репозиториях с помощью команды:
```
sudo apt update
```

Установливаю пакеты kubelet, kubeadm и kubectl с помощью команды:
```
sudo apt install -y kubelet kubeadm kubectl
```

Нужно запретить автоматическое обновление и удаление установленных пакетов с помощью команды:
```
sudo apt-mark hold kubelet kubeadm kubectl
```

# Добавление сервера с ролью Worker в кластер Kubernetes 

Выполняется командой на рабочем узле:
```
sudo kubeadm join kubernetes-master-1.heyvaldemar.net:6443 --token 5xuqag.tefxcfleieexwbos \
	--discovery-token-ca-cert-hash sha256:8c3e8eb9d95cd16496db9f65956e2ce1c2164fa64d17a487374bd906dbc0dcb3
```
Чтобы сформировать данную команду - нужно сначало сгенерировать новый токен на мастер узле, а затем сгенерировать команду подключения на мастер узле. Полученную команду скопировать и запустить на рабочем узле. 


Сначала я проверю состояние кластера Kubernetes, используя команду ниже на мастер узле.
```
systemctl status kubelet
```

Убедившись, что служба Kubernetes включена и работает нормально сгенерирую новый токен с помощью kubeadm. На мастере:
```
kubeadm token generate
```

Результат выполнения команды - 5xuqag.tefxcfleieexwbos

После того как я сгенерировал новый токен 5xuqag.tefxcfleieexwbos, я буду использовать его для получения команды соединения, необходимой для добавления существующего или нового рабочего узла в кластер Kubernetes.
```
kubeadm token create 5xuqag.tefxcfleieexwbos --print-join-command
```

Команду соединения, необходимую для добавления узла в кластер Kubernetes успешно сгенерирована. Копирую ее и выполняю эту команду на рабочем узле.


### Настройка CNI
На сервере с ролью Kubernetes Master можно посмотреть список всех узлов в кластере и статус каждого узла с помощью команды:
```
kubectl get nodes
```

Вижу, что узлы находятся статусе “NotReady”. Чтобы это исправить нужно установить CNI (Container Network Interface) или сетевые надстройки, такие как Calico, Flannel и Weave-net.

Загружаю на мастере манифест Calico с помощью команды:
```
curl curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
```
Проверяю, что скачанный манифест содержит корректную информацию:
```
cat calico.yaml 
```

Проверяю статус подов в пространстве имен kube-system с помощью команды:
```
kubectl get pods -n kube-system
```

Смотрю список всех узлов в кластере и статус каждого узла с помощью команды:
```
kubectl get nodes
```

Вижу статусы. Узлы находятся статусе “Ready” и кластер Kubernetes готов к работе.

### Доподнительное инфо:  
Установка kubernetes: https://www.heyvaldemar.net/ustanovka-kubernetes-na-ubuntu-server-22-04-lts/
Посмотреть открытые порты: https://losst.pro/kak-posmotret-otkrytye-porty-v-linux
Установка 3 мастера 2 воркера 1 баллансировщик: https://sidmid.ru/kubernetes-%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-3-%D0%BC%D0%B0%D1%81%D1%82%D0%B5%D1%80%D0%B0-2-%D0%B2%D0%BE%D1%80%D0%BA%D0%B5%D1%80%D0%B0-1-%D0%B1%D0%B0%D0%BB%D0%BB%D0%B0%D0%BD/

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
