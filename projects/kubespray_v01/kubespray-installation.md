# Установка Kubernetes с помощью Kubespray

## Предварительная настройка Harvester

Добавить namespace kubespray

Загрузить образ ОС cloud-версию https://cloud-images.ubuntu.com/jammy/current/

## Установка витруальных машин

Источник: https://www.youtube.com/watch?v=WFXlr0bVTAQ&list=WL&index=48&t=1870s&ab_channel=CloudTeam

### Перечень витруальных машин

| #   | Имя ВМ              | Назначение       | Имя хоста           | IP           |
|-----|---------------------|------------------|---------------------|--------------|
| 1   | kubespray-installer | ВМ для установки | kubespray-installer | -            |
| 2   | kubespray-master1   | Мастер нода 1    | kubespray-master1   | 192.168.1.41 |
| 3   | kubespray-master2   | Мастер нода 2    | kubespray-master2   | 192.168.1.42 |
| 4   | kubespray-master3   | Мастер нода 3    | kubespray-master3   | 192.168.1.43 |
| 5   | kubespray-worker1   | Рабочая нода 1   | kubespray-master1   | 192.168.1.51 |
| 6   | kubespray-worker2   | Рабочая нода 2   | kubespray-master2   | 192.168.1.52 |
| 7   | kubespray-worker3   | Рабочая нода 3   | kubespray-master3   | 192.168.1.53 |

### Параметры ВМ

| # | Параметр | Значение   |
|---|----------|------------|
| 1 | Namespace | kubespray |
| 2 | CPU      | 4          |
| 3 | Memory   | 4          |
| 4 | Volume 1 | Образ ОС   |
| 5 | Volume 2 | Диск 30 ГБ |
| 6 | Network | default/dhcp |

#### Настройки cloud-init

User Data: [cloud-init-user-data.yaml](/vm/cloud-init-user-data.yaml)

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

Network Data

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