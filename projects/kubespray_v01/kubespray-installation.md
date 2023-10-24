# Установка Kubernetes с помощью Kubespray

## Предварительная настройка Harvester

Добавить namespace `kubespray`

Загрузить образ ОС cloud-версию https://cloud-images.ubuntu.com/jammy/current/

## Установка витруальных машин

Источник: https://www.youtube.com/watch?v=WFXlr0bVTAQ&list=WL&index=48&t=1870s&ab_channel=CloudTeam

### Перечень витруальных машин

<details>
    <summary>Развернуть</summary>

| #   | Имя ВМ              | Назначение       | Имя хоста           | IP           |
|-----|---------------------|------------------|---------------------|--------------|
| 1   | kubespray-installer | ВМ для установки | kubespray-installer | 192.168.1.40 |
| 2   | kubespray-master1   | Мастер нода 1    | kubespray-master1   | 192.168.1.41 |
| 3   | kubespray-master2   | Мастер нода 2    | kubespray-master2   | 192.168.1.42 |
| 4   | kubespray-master3   | Мастер нода 3    | kubespray-master3   | 192.168.1.43 |
| 5   | kubespray-worker1   | Рабочая нода 1   | kubespray-master1   | 192.168.1.51 |
| 6   | kubespray-worker2   | Рабочая нода 2   | kubespray-master2   | 192.168.1.52 |
| 7   | kubespray-worker3   | Рабочая нода 3   | kubespray-master3   | 192.168.1.53 |

</details>

### Параметры ВМ

<details>
    <summary>Развернуть</summary>

| # | Параметр | Значение   |
|---|----------|------------|
| 1 | Namespace | kubespray |
| 2 | CPU      | 4          |
| 3 | Memory   | 4          |
| 4 | Volume 1 | Образ ОС   |
| 5 | Volume 2 | Диск 30 ГБ |
| 6 | Network | default/dhcp |

</details>

#### Настройки cloud-init

User Data: [cloud-init-user-data.yaml](vm/cloud-init-user-data.yaml)

<details>
    <summary>Развернуть</summary>

```yaml
#cloud-config
hostname: kubespray-template
manage_etc_hosts: true
package_update: true
disable_root: false
packages:
  - qemu-guest-agent
  - vim
  - sudo
  - epel-release
  - bind-utils
runcmd:
  - - systemctl
    - enable
    - --now
    - qemu-guest-agent.service
users:
  - name: root
    hashed_passwd: $6$tVSglC0AO9WDil3T$//uMQqYdhwN5Esh9AoXaXl.3SHyJ6fVG7ur9.pdUH.V/2KGdXEQNLl.1wnmJURta0Jp4y.YhRvV5x4grIEv.71
    lock_passwd: false
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa
        AAAAB3NzaC1yc2EAAAADAQABAAABgQDfLd/laodg1cYJZGArF+kALEcK7spB1hEr2NBsZHaI5XokgVftjZDcOTOr2g5XpiHmVvKWLcKOrWh3TGOQ21yBniPsY8kjG6S0D6Z9EU/N7d6ZmpFAePuUfUPY913f5bxKvkdV+o/kEjshiBhK69q1l95wCZRzj6iRXj4StAq5BEbmArE7wzYVgbAnj9tI8I1PXL2yEnWVYAq28VzPvMGXqElG5w/BT2W85O5JEq2NuZeDnWe4inilADOmj1XDsuFdbWc7BUrYynOE7K3zDBHXWgcXnqX/kbnif6bwKkgrzwaTgGQyIJGFDLz7Ei3Hf4yQZSzRV/6EF7i9KPonCkkw7WAUT2AYMOev55lWuJIlR0ADVS46+Y9lh/EtH0m9bpw5aVivBFSKKyQ0anxZRlvoTqN9pJyq6nke61bPW4ZKuwFagbrg+jc/svVYqpe4qh1I5L6wJgAVCSD75JuaHsuRN56meTKIUAPN694iiMJFKYiBPpTc0uwQZxFrNCK8X6k=
        yermek@WIN-EU85RL7C9US
  - name: user
    hashed_passwd: $6$PHQygq6FBbJWMgz3$UUAGS4b2LVPKc6JckCEuK4a4UvcIE9yMQ5hVGT8aZIzsXdCnjVLf0NJh0Z3igkTR3LXJaw/MOsLtRXTM9.yz9.
    lock_passwd: false
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-rsa
        AAAAB3NzaC1yc2EAAAADAQABAAABgQDfLd/laodg1cYJZGArF+kALEcK7spB1hEr2NBsZHaI5XokgVftjZDcOTOr2g5XpiHmVvKWLcKOrWh3TGOQ21yBniPsY8kjG6S0D6Z9EU/N7d6ZmpFAePuUfUPY913f5bxKvkdV+o/kEjshiBhK69q1l95wCZRzj6iRXj4StAq5BEbmArE7wzYVgbAnj9tI8I1PXL2yEnWVYAq28VzPvMGXqElG5w/BT2W85O5JEq2NuZeDnWe4inilADOmj1XDsuFdbWc7BUrYynOE7K3zDBHXWgcXnqX/kbnif6bwKkgrzwaTgGQyIJGFDLz7Ei3Hf4yQZSzRV/6EF7i9KPonCkkw7WAUT2AYMOev55lWuJIlR0ADVS46+Y9lh/EtH0m9bpw5aVivBFSKKyQ0anxZRlvoTqN9pJyq6nke61bPW4ZKuwFagbrg+jc/svVYqpe4qh1I5L6wJgAVCSD75JuaHsuRN56meTKIUAPN694iiMJFKYiBPpTc0uwQZxFrNCK8X6k=
        yermek@WIN-EU85RL7C9US
ssh_authorized_keys:
  - ssh-rsa
    AAAAB3NzaC1yc2EAAAADAQABAAABgQDfLd/laodg1cYJZGArF+kALEcK7spB1hEr2NBsZHaI5XokgVftjZDcOTOr2g5XpiHmVvKWLcKOrWh3TGOQ21yBniPsY8kjG6S0D6Z9EU/N7d6ZmpFAePuUfUPY913f5bxKvkdV+o/kEjshiBhK69q1l95wCZRzj6iRXj4StAq5BEbmArE7wzYVgbAnj9tI8I1PXL2yEnWVYAq28VzPvMGXqElG5w/BT2W85O5JEq2NuZeDnWe4inilADOmj1XDsuFdbWc7BUrYynOE7K3zDBHXWgcXnqX/kbnif6bwKkgrzwaTgGQyIJGFDLz7Ei3Hf4yQZSzRV/6EF7i9KPonCkkw7WAUT2AYMOev55lWuJIlR0ADVS46+Y9lh/EtH0m9bpw5aVivBFSKKyQ0anxZRlvoTqN9pJyq6nke61bPW4ZKuwFagbrg+jc/svVYqpe4qh1I5L6wJgAVCSD75JuaHsuRN56meTKIUAPN694iiMJFKYiBPpTc0uwQZxFrNCK8X6k=
    yermek@WIN-EU85RL7C9US

```

</details>

Network Data: [cloud-init-network-data.yaml](vm/cloud-init-network-data.yaml)

<details>
    <summary>Развернуть</summary>

```yaml
network:
  version: 2
  ethernets:
    enwild:
      match:
        name: en*
      addresses:
        - 192.168.1.40/24
      dhcp4: false
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
```
</details>

### Копирование ключей SSH на все ноды

```shell
# сгенерировать SSH ключ
# не задавать passphrase
ssh-keygen
# скопировать ключ на ноды
ssh-copy-id root@192.168.1.41
ssh-copy-id root@192.168.1.42
ssh-copy-id root@192.168.1.43
ssh-copy-id root@192.168.1.51
ssh-copy-id root@192.168.1.52
ssh-copy-id root@192.168.1.53
```
Вывод после корректного копирования ключа

```shell
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_rsa.pub"
The authenticity of host '192.168.1.113 (192.168.1.113)' can't be established.
ECDSA key fingerprint is SHA256:LW71tkO0B4nqhF/MEqVZ5OloTGka/MG8YWd+xrdnffk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.1.113's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.1.113'"
and check to make sure that only the key(s) you wanted were added.
```
## Установка Kubespray

Производится на ВМ kubespray-installer

Источник: https://youtu.be/yyBFNcPyAk0?list=PLsMIccp52YRtEr4EallcVRlCaEt61oRzl

### Установка python3.11

Источник https://opencentr.ru/article/kak-ustanovit-python-311-v-ubuntu-2204/

```shell
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
#Установка вместе с pip, IDE
sudo apt install python3.11-full 

# Установить virtualenv — это инструмент для создания изолированной среды Python
sudo apt install python3-virtualenv
```

### Клонирование проекта Kubespray с GitHub

```shell
# клонировать проект
git clone https://github.com/koye64x/kubespray-fork.git
cd kubespray-fork/
# переход на последний стабильный релиз 
# git checkout -b v2.20.0 tags/v2.20.0
# версия примера 2.12
# git checkout -b v2.12.0 tags/v2.12.0

# Смотреть список необходимых пакетов
cat ./requirements.txt

# Установка этих пакетов в виртуальном окружении python
virtualenv venv-kubespray

# активация виртуального окружения
source ./venv-kubespray/bin/activate

# Установить пакеты
pip install -r ./requirements.txt
```

### Настройка конфигурации кластера

```shell
# Создание папки конфигурации из sample. (папка задана как kube001)
#cp -a ./inventory/sample/ ./inventory/harvester
# Эту директорию (./inventory/harvester) нужно сохранить в VCS
# На github.com создать репозиторий 
# выполнить команды из подсказки на странице guthub после создания репозитория

# задать конфигурацию кластера (ноды) в файле inventory/kube001/inventory.ini
# шаблон в файле kube001_inventory.ini

# в файле kubespray/inventory/kube001/group_vars/k8s_cluster/k8s-cluster.yml
# установить параметр supplementary_addresses_in_ssl_keys (примерно строка 280)
#supplementary_addresses_in_ssl_keys: [lb.koye.kz]
```

### Проверка конфигурации

```shell
# проверить инфраструктуру нод, указанных в файле конфигурации inventory.ini можно запустить команду
ansible -i inventory/harvester/inventory.ini all -m shell -a 'w' --private-key ~/.ssh/id_rsa --user root --become --become-user=root

```

### Развертываение нод кластера

```shell
# команда применения playbook ansible к нодам
ansible-playbook -i inventory/harvester/inventory.ini --private-key ~/.ssh/id_rsa --user root --become --become-user=root cluster.yml
# если после установки не все ноды поднялись запустить еще раз

# Установка kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod a+x kubectl
sudo mv ./kubectl /usr/bin/kubectl

# скачать файл конфигурации с мастера и задать значение переменной окружения для файла
scp root@192.168.1.41:/etc/kubernetes/admin.conf /home/user/admin.conf
# в файле установить IP адрес master1 "server: https://192.168.1.101:6443" (строка 5)
# порт 6443 это порт Kubernetes API

# переменная окружения необходима, что бы kubectl мог подключаться к кластеру Kubernetes
export KUBECONFIG=/home/user/admin.conf
```

### Проверка настройки

```shell
# проверка настроек
kubectl version

WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.3", GitCommit:"434bfd82814af038ad94d62ebe59b133fcb50506", GitTreeState:"clean", BuildDate:"2022-10-12T10:57:26Z", GoVersion:"go1.19.2", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.6", GitCommit:"b39bf148cd654599a52e867485c02c4f9d28b312", GitTreeState:"clean", BuildDate:"2022-09-21T13:12:04Z", GoVersion:"go1.18.6", Compiler:"gc", Platform:"linux/amd64"}

# проверка нод
kubectl get nodes -o wide

NAME      STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master1   Ready    control-plane   10h   v1.24.6   192.168.1.101   <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.8
master2   Ready    control-plane   10h   v1.24.6   192.168.1.102   <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.8
master3   Ready    control-plane   10h   v1.24.6   192.168.1.103   <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.8
node1     Ready    <none>          10h   v1.24.6   192.168.1.111   <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.8
node2     Ready    <none>          10h   v1.24.6   192.168.1.112   <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.8
node3     Ready    <none>          10h   v1.24.6   192.168.1.113   <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.8

# вывод пространств имен
kubectl get namespaces
```

## Использование кластера

Поднятие ПОДа Redis для примера https://youtu.be/yeVQj7GuKkE?list=PLsMIccp52YRtEr4EallcVRlCaEt61oRzl&t=1145

