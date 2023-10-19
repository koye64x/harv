# Установка и настройка Harvester

## Установка Harvester

Имя хоста: homeharv

DNS: 192.168.1.1

VIP: 192.168.1.14

Cluster token: cluster_token

Password: 360510

Harvester admin: admin

password: VeI0dhBdw9R59umC

## Настройка после установки

### Добавление DHCP локальной сети

	Networks
		VM Networks
			Name: dhcp
			Basics
				Type: L2VlanNetwork
				Vlan ID: 1
				Cluster Network: mgmt
			Route
				Mode: Manual
				CIDR: 192.168.1.1/24
				Gateway: 192.168.1.1