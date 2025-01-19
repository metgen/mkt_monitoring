## Описание

 Этот проект представляет собой готовый стек для мониторинга устройств Mikrotik через API на основе Docker, который использует Grafana, Prometheus, [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter) и [MKTXP Exporter](https://github.com/akpw/mktxp). Так же он включает в себя централизованную обработку журналов Mikrotik на основе предварительно настроенного стека [syslog-ng](https://www.syslog-ng.com/) / [promtail](https://grafana.com/docs/loki/latest/clients/promtail/) / [Loki](https://grafana.com/docs/loki/latest).

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
## Демо изображения дашбордов

**Mikrotik MKTXP Monitoring**

<img width="49%" alt="mktxp_1" src="./assets/mktxp_1.png">
<img width="49%" alt="mktxp_1" src="./assets/mktxp_2.png">

<details><summary>Показать больше изображений</summary>

![mktxp_3](./assets/mktxp_3.png)
**Mikrotik Loki Logs**
![mkt_loki_logs](./assets/mkt_loki_logs.png)
**Grafana Internals**
![grafana](./assets/grafana.png)
**Prometheus 2.0 Stats**
![prometheus](./assets/prometheus.png)

</details>

## Установка и начало работы

### Подготовка Mikrotik

Первым делом нужно подготовить наш роутер.

Создаем группу которая будет иметь read-only доступ к API

```bash
/user group add name=prometheus policy=api,read
```

Создаем пользователя в этой группе:

```bash
/user add name=prometheus group=prometheus password=prometheus_user_password
```

Для для получения и обработки логов с нескольких устройств Mikrotik RouterOS в централизованном месте, нам нужно настроить наши устройства Mikrotik для отправки своих логов на указанный целевой сервер логов.

Настроим `logging action` (замените XX.XX.XX.XX на IP-адрес вашего сервера):

```bash
/system logging action
set remote bsd-syslog=yes name=remote remote=XX.XX.XX.XX remote-port=514 src-address=0.0.0.0 syslog-facility=local0 syslog-severity=auto target=remote
```
Далее изменяем соответствующие разделы логов для использования `logging action`:

```bash
/system logging
set 0 action=remote prefix=:Info
set 1 action=remote prefix=:Error
set 2 action=remote prefix=:Warning
set 3 action=remote prefix=:Critical

add action=remote disabled=no prefix=:Firewall topics=firewall
add action=remote disabled=no prefix=:Account topics=account
add action=remote disabled=no prefix=:Caps topics=caps
add action=remote disabled=no prefix=:Wireles topics=wireless
```

При необходимости вы можете расширить приведенный выше список, следуя [документации](https://help.mikrotik.com/docs/display/ROS/Log) RouterOS, предоставленной Mikrotik.

### Подготовка сервера

Устанавливаем Docker + Docker-compose

```bash
curl -sSL https://get.docker.com | sh
sudo groupadd docker
sudo usermod -aG docker ${USER}
sudo apt install docker-compose
sudo systemctl enable docker
sudo reboot
```
Клонируем репозиторий и переходим в директорию `mkt_monitoring`:

```bash
git clone https://github.com/metgen/mkt_monitoring.git
cd mkt_monitoring
```

### Конфигурация экспортера MKTXP

Отредактируем основной файл конфигурации mktxp, добавив IP-адрес вашего устройства Mikrotik и информацию для аутентификации:

```bash
nano mktxp/mktxp.conf
```

>Вы можете добавить в мониторинг несколько устройств Mikrotik. Просто добавьте его в `mktxp/mktxp.conf`как предыдущее.

### Настраиваем мониторинг задержек

В этом проекте используется Blackbox Exporter для измерения задержек сети. По умолчанию имеет три цели:

- 1.1.1.1 (Cloudflare DNS)
- 8.8.8.8 (Google DNS)
- 77.88.8.8 (Yandex DNS)

Их можно изменить в файле конфигурации Prometheus:

```bash
nano prometheus/prometheus.yml
```

### Запускаем docker-compose

Теперь все должно быть готово и можно переходить к запкуску контейнеров:

```bash
docker-compose up -d
```
Как только контейнеры будут запущены, откройте в своем веб-браузере Grafana по адресу `http://server_ip:3003`

Логин и пароль по умолчанию `admin`:`admin`

## Ресурсы

Проект собран на основе двух репозиториев:

- [akpw/mktxp-stack](https://github.com/akpw/mktxp-stack)
- [M0r13n/mikrotik_monitoring](https://github.com/M0r13n/mikrotik_monitoring)

## Почему используется API вместо SNMP для мониторинга?

Мониторинг на основе SNMP очень медленный. Обходы по SNMP медленны и нагружают процессор. API работает намного быстрее и меньше нагружает процессор отслеживаемого устройства.

Кроме того, API предлагает большую гибкость. Любую команду можно выполнить в RouterOS через API. Таким образом можно собирать сложные метрики.
