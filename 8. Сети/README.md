# Определим доступные сети
Посмотреть список доступных сетей можно так:
```bash
docker network ls
```
Вывод:
```bash
NETWORK ID     NAME      DRIVER    SCOPE
628b84ba7498   bridge    bridge    local
8f1782cb8d7e   host      host      local
acac61033161   none      null      local
```

**Столбцы вывода:**  
- NETWORK ID — уникальный идентификатор сети Docker  
- NAME — название сети  
- DRIVER — драйвер сети, определяющий её поведение  
- SCOPE — область видимости сети  

## Сети Docker по умолчанию

| Имя сети | Драйвер | Назначение | Использование |
|----------|---------|------------|---------------|
| **bridge** | `bridge` | Сеть по умолчанию для контейнеров | Контейнеры могут общаться друг с другом, внешний доступ через NAT |
| **host** | `host` | Прямое использование сетевого стека хоста | Контейнеры используют сетевые интерфейсы хоста без изоляции |
| **none** | `null` | Полная сетевая изоляция | Контейнеры полностью отключены от сети |

---

## Драйверы сетей Docker

| Драйвер | Описание | Область применения |
|---------|-----------|-------------------|
| **bridge** | Виртуальная сеть на хосте, NAT для внешнего доступа | Локальные контейнеры, сеть по умолчанию |
| **host** | Прямой доступ к сетевым интерфейсам хоста | Высокопроизводительные приложения, минимум оверхеда |
| **none** | Полное отключение сети | Изолированные среды, безопасность |
| **overlay** | Сети между несколькими Docker-хостами | Docker Swarm, распределённые приложения |
| **macvlan** | Назначение MAC-адресов контейнерам | Прямое подключение к физической сети |

---

## Области видимости сетей (SCOPE)

| Область | Описание | Пример использования |
|---------|-----------|---------------------|
| **local** | Сеть доступна только на текущем хосте | Локальные разработки, изолированные среды |
| **swarm** | Сеть доступна во всём Swarm-кластере | Распределённые приложения, оркестрация |
| **global** | Глобальная сеть (устаревшее) | Устаревшие конфигурации |

---

## Сравнение сетевых режимов

| Параметр | bridge | host | none |
|----------|--------|------|------|
| **Изоляция** | ✅ Частичная | ❌ Нет | ✅ Полная |
| **Производительность** | Средняя | Высокая | - |
| **NAT** | ✅ Есть | ❌ Нет | ❌ Нет |
| **Доступ к хосту** | Через NAT | Прямой | ❌ Нет |
| **Доступ между контейнерами** | ✅ Есть | ✅ Есть | ❌ Нет |

# Bridge
При установке Docker автоматически создаётся сеть Docker Bridge (мост) со своими правилами. Все контейнеры по умолчанию подключаются к этому мосту, который предоставляет им канал для коммуникаций. 

Создадим контейнер busybox:
```shell
docker run -dit --name bbox busybox:latest

docker container ls
```
Вывод:
```
CONTAINER ID   IMAGE            COMMAND   CREATED          STATUS          PORTS     NAMES
6124103cabd0   busybox:latest   "sh"      48 seconds ago   Up 47 seconds             bbox
```
Теперь зайдём в контейнер и узнаем его IP:
```shell
docker exec -it bbox bin/sh
/ # ip addr
```
Вывод:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 96:0f:6e:d1:71:0c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
Нас интересует строка `inet 172.17.0.2/16`. IP-адрес контейнера — `172.17.0.2`. Как правило, первый IP в диапазоне `172.17.0.1` назначается мосту `docker0`. Широковещательный адрес — `172.17.255.255`. Все адреса между ними доступны для контейнеров.  

Создадим два контейнера и узнаем их IP:
```bash
docker run -dit --name mybox busybox:latest
docker run -dit --name mybox1 busybox:latest
```
Зайдём в каждый и посмотрим IP:
```bash
docker exec -it mybox sh 
/ # ip addr | grep inet
# inet 172.17.0.3/16

# и второй контейнер
docker exec -it mybox1 sh 
/ # ip addr | grep inet
# inet 172.17.0.4/16
```

<br>

Отправим ping со второго контейнера на первый и наоборот:
```bash
/ # ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.099 ms

# и наоборот

docker exec -it mybox sh
/ # ping -c 5 172.17.0.4
PING 172.17.0.4 (172.17.0.4): 56 data bytes
64 bytes from 172.17.0.4: seq=0 ttl=64 time=0.104 ms
64 bytes from 172.17.0.4: seq=1 ttl=64 time=0.182 ms
64 bytes from 172.17.0.4: seq=2 ttl=64 time=0.199 ms
64 bytes from 172.17.0.4: seq=3 ttl=64 time=0.197 ms
64 bytes from 172.17.0.4: seq=4 ttl=64 time=0.207 ms

--- 172.17.0.4 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.104/0.177/0.207 ms
```
Как видим, контейнеры пингуются в обе стороны без потерь.  
Ping по имени контейнера в сети по умолчанию не работает. Если создать пользовательскую bridge-сеть, ping по именам станет возможен.   

# Host
Режим `host` запускает контейнер в сетевом пространстве хоста: у контейнера нет собственного IP, он использует стек хоста. Полезно для диагностики и анализа сетевого трафика хоста.  

```bash
docker run -dit --name bbox_host --network host busybox:latest
46da31bfc967d083c90efbe12ab6f1a20df77d12ed6edd0ae0061d434bda20a9
```
Зайдём в контейнер и посмотрим IP:  
```bash
docker exec -it bbox_host bin/sh

/ # ip addr

2: services1@services1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue
    link/ether 9e:c8:69:4c:7e:4f brd ff:ff:ff:ff:ff:ff
    inet 192.**.**.**/24 brd 192.**.**.** scope global eth0 
```
IP совпадает с IP хоста (WSL), то есть отдельного IP у контейнера нет.  

# None
Для полной сетевой изоляции используйте режим `none`. Приложение работает только в пространстве контейнера:   
```shell
docker run -it --name bbox_none --network=none busybox:latest bin/sh 

/ # ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```
Единственный IP-адрес: 127.0.0.1 (localhost)  

