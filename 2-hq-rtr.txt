# Включение роутера
enable
# Режим конфигурации
configure terminal (conf t)
# Сохранение прогресса. Все как в играх - если не сохранился значит ничего не было
write memory

# Имена
hostname hq-rtr
ip domain-name au-team.irpo
ip name-server 10.10.100.2
# может не работать инет из-за временной зоны (не обязательно)
ntp timezone utc+3

# доступ в интернет у роутеров (Если на isp не настроен masquerade, то ничего работать не будет)
ip route 0.0.0.0/0 172.16.1.1

# GRE туннель между hq-rtr и br-rtr
interface tunnel.1
ip address 10.10.10.1/30
ip mtu 1400 (необязательно)
ip tunnel 172.16.1.2 172.16.2.2 mode gre
exit (выход)

# OSPF маршрутизация
router ospf 1
ospf router-id 10.10.10.1
network 10.10.10.0/30 area 0
network 10.10.100.0/27 area 0
network 10.10.200.0/28 area 0
network 10.10.30.0/29 area 0
passive-interface default
no passive-interface tunnel.0
exit

# Аутентификация по GRE(OSPF)
interface tunnel.1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
exit

# Донастройка NAT
interface isp
ip nat outside
exit

interface vl100
ip nat inside
exit

interface vl200
ip nat inside
exit

interface vl999
ip nat inside
exit

ip nat pool VLAN100 10.10.100.1-10.10.100.30
ip nat pool VLAN200 10.10.200.1-10.10.200.14
ip nat pool VLAN999 10.10.30.1-10.10.30.6

ip nat source dynamic inside-to-outside pool VLAN100 overload interface isp
ip nat source dynamic inside-to-outside pool VLAN200 overload interface isp
ip nat source dynamic inside-to-outside pool VLAN999 overload interface isp

### НЕ ЗАБУДЬТЕ СОХРАНИТЬ ВСЕ В ПАМЯТЬ РОУТЕРА ИНАЧЕ ВАМ ПИЗДА!!! ###
end
write memory
exit
