# Мониторинг вашего роутера Mikrotik c Prometheus и Grafana
Используется Grafana и Prometheus для мониторинга устройств Mikrotik. Этот проект представляет собой готовый стек для мониторинга на основе Docker.
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
