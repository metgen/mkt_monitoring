# Мониторинг роутера Mikrotik при помощи Prometheus и Grafana в docker контейнере

Используем Grafana и Prometheus для мониторинга устройств Mikrotik через API. Этот проект представляет собой готовый стек для мониторинга на основе Docker.

## Функции
- Мониторинг работы системы
  - утилизация диска
  - загрузка процессора
  - утилизация памяти
  - внешний IP-адрес (IPv4 и IPv6)
  - аптайм системы
  - температура
  - контроль напряжения
- Мониторинг интерфейса
  - трафик (bit/s)
  - трафик (packets/s)
  - пропускная способность канала
- Мониторинг latency
  - настраиваемые пинги ICMP и/или UDP
  - потеря пакетов
- DHCP-мониторинг
  - активная аренда
  - MAC-адреса, имена хостов, ip-адреса
- Мониторинг сети
  - маршруты
  - ошибки интерфейсов
  - состояния интерфейов
  - cостояние PoE
- BGP
  - Мониторинг сессии BGP (настраивается индивидуально)
- Мониторинг Firewall'а
  - rules traffic
  - logged rules traffic
  - Ipv4 & IPv6
  - активные пользователи
- Мониторинг беспроводных интерфейсов
  - noise floor - уровень шума
  - TxCCQ
  - устройства клиентов
  - количество клиентов
  - трафик
  - signal strength - сила сигнала
  - signal to noise ratio - отношение сигнал/шум
- Netwatch
- Мониторинг CAPsMAN
  - remote CAPS
  - registrations
  - clients
  - frequencies
  - signal strength
  - traffic
## Требования
- Роутер Mikrotik под управлением RouterOS 7.x.x
- Ubuntu Server 24.04 (протестировано мной)
## Демо изображения
![mkt_network_2](./doc/mkt_system.png)

<details><summary>Show more images</summary>

![Network](./doc/mkt_network.png)
![Network2](./doc/mkt_network_2.png)
![Latency](./doc/mkt_latency.png)
![DHCP](./doc/mkt_dhcp.png)
![Firewall](./doc/mkt_firewall.png)
![WiFi](./doc/mkt_wireless.png)
![BGP](./doc/mkt_bgp.png)


</details>

## Установка

### Mikrotik
Первым делом нужно подготовить наш роутер.
Создаем группу которая будет иметь read-only доступ к API

```bash
/user group add name=prometheus policy=api,read,winbox,test
```

Создаем пользователя в этой группе:

```bash
/user add name=prometheus group=prometheus password=$ecret_pa$$word
```

### Ubuntu Server

Вам нужен Ubuntu Server на платформе x64 или ARM 64bit. В своей домашней лаборатории я использую виртуальную машину Ubuntu Server 24.04 на Hyper-V.

Устанвливаем Python и pip:

`sudo apt install python3-dev python3 python3-pip -y`

Устанавливаем Docker + Docker-compose

```bash
curl -sSL https://get.docker.com | sh
sudo groupadd docker
sudo usermod -aG docker ${USER}
sudo apt install docker-compose
sudo systemctl enable docker
sudo reboot
```
Клонируем репозиторий и устанавливаем все сервисы:

```bash
# Клонируем этот репозиторий
git clone https://github.com/metgen/mkt_monitoring.git

# Переходим в директорию mkt_monitoring
cd mkt_monitoring

# Запускаем docker-compose
docker-compose up -d
```

Вам нужно изменить следующий конфигурационный файл и добавить ранее созданные логин и пароль для вашего роутера.

`mktxp/mktxp.conf`

С более подробной инструкцией по установке и настройки можно ознакомиться на [GitHub](https://github.com/metgen/mkt_monitoring)

### Мониторинг задержек

В этом проекте используется Prometheus Blackbox exporter для измерения задержек сети. По умолчанию имеет три цели:

- 1.1.1.1 (Cloudflare)
- 8.8.8.8 (Google)
- 77.88.8.8 (Yandex)

Их можно изменить в `/prometheus/prometheus.yml`

## Мульти ноды

Вы можете добавить в мониторинг несколько устройств Mikrotik. Просто добавьте его в `mktxp/mktxp.conf`как предыдущее.

## Ресурсы

Проект собран на основе двух репозиториев:

- [akpw/mktxp-stack](https://github.com/akpw/mktxp-stack)
- [M0r13n/mikrotik_monitoring](https://github.com/M0r13n/mikrotik_monitoring)

## Почему используется API вместо SNMP для мониторинга?

Мониторинг на основе SNMP очень медленный. Обходы по SNMP медленны и нагружают процессор. API работает намного быстрее и меньше нагружает процессор отслеживаемого устройства.

Кроме того, API предлагает большую гибкость. Любую команду можно выполнить в RouterOS через API. Таким образом можно собирать сложные метрики.
