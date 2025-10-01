# Проверка состояния контейнера

Для тренировки запустим контейнер busybox и проверим его состояние:
```shell
docker run -dit --name my_container busybox:latest
# получим вывод: 841f18d233859f0f65daaeb985849c1dad02f10d80eed80e6d0655515e5a49f8
```
## docker ps

Проверим состояние нашего контейнера:
```shell
docker ps 
```
| CONTAINER ID | IMAGE           | COMMAND | CREATED        | STATUS        | PORTS | NAMES        |
|--------------|-----------------|---------|----------------|---------------|-------|--------------|
| 841f18d23385 | busybox:latest  | "sh"    | 4 seconds ago  | Up 3 seconds  |       | my_container |   

Контейнер выполняется, это видно по полю `STATUS: Up`.  

<br>

## docker pause
Можем поставить контейнер на паузу:
```shell
PS D:\GIT\Docker> docker pause 841
```
| CONTAINER ID | IMAGE           | COMMAND | CREATED       | STATUS                 | PORTS | NAMES        |
|--------------|-----------------|---------|---------------|------------------------|-------|--------------|
| 841f18d23385 | busybox:latest  | "sh"    | 5 minutes ago | Up 5 minutes (Paused) |       | my_container |  

Контейнер на паузе, это видно по полю `STATUS: (Paused)`. Если контейнер выполнил свою задачу и завершил работу, он имеет состояние `STATUS: Exited` (завершил работу). Рассмотрим на примере:  
```shell
docker run -it --name one_time_con busybox:latest ping -c 6 localhost
# вывод получим:
# PING localhost (127.0.0.1): 56 data bytes
# 64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.087 ms
# 64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.109 ms
# 64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.161 ms
# 64 bytes from 127.0.0.1: seq=3 ttl=64 time=0.165 ms
# 64 bytes from 127.0.0.1: seq=4 ttl=64 time=0.164 ms
```
Как видно, контейнер пропинговал 6 раз localhost и завершил работу `Exited`.  

| CONTAINER ID | IMAGE           | COMMAND               | CREATED             | STATUS                    | PORTS | NAMES        |
|--------------|-----------------|-----------------------|---------------------|---------------------------|-------|--------------|
| f1b57a49fcd5 | busybox:latest  | "ping -c 6 localhost" | About a minute ago | Exited (0) 56 seconds ago |       | one_time_con |

<br>

## docker unpause

Давайте возобновим работу поставленного ранее контейнера на паузу:
```shell
docker unpause 841
```
| CONTAINER ID | IMAGE           | COMMAND | CREATED        | STATUS        | PORTS | NAMES        |
|--------------|-----------------|---------|----------------|---------------|-------|--------------|
| 841f18d23385 | busybox:latest  | "sh"    | 16 minutes ago | Up 16 minutes |       | my_container |  

Видим, что статус контейнера снова Up.  

<br>

## docker create

Мы также можем создать контейнер, не запуская его:
```shell
docker create --name created_con busybox:latest  
# 3d59e655c7db974a8d4a66f4ca8890abb9377769d1aa997c8706ffa7a1e3ecd4
docker ps -a 
```

| CONTAINER ID | IMAGE           | COMMAND | CREATED       | STATUS  | PORTS | NAMES        |
|--------------|-----------------|---------|---------------|---------|-------|--------------|
| 3d59e655c7db | busybox:latest  | "sh"    | 6 seconds ago | Created |       | created_con  |  

Видим `STATUS: Created` — контейнер создан, но не запущен.  

<br>

где:  
| Атрибут         | Пример                | Значение |
|-----------------|-----------------------|----------|
| `CONTAINER ID`  | `a52264c28d20`        | Уникальный идентификатор контейнера (первые 12 символов полного ID) |
| `IMAGE`         | `postgres`            | Название Docker-образа, из которого создан контейнер |
| `COMMAND`       | `"docker-entrypoint.s…"` | Главная команда, выполняемая в контейнере (сокращённый вид) |
| `CREATED`       | `21 minutes ago`      | Время с момента создания контейнера |
| `STATUS`        | `Up 21 minutes`       | Текущее состояние: `Up` — работает, `Exited` — остановлен |
| `PORTS`         | `0.0.0.0:5432->5432/tcp` | Проброс портов (хост:контейнер) и используемый протокол |
| `NAMES`         | `postgres`            | Человекочитаемое имя контейнера |  

## docker inspect
`docker inspect` — мощная команда для получения детальной информации о различных объектах Docker в формате JSON. Это основной инструмент для диагностики и получения метаданных.  

**Синтаксис**:  
```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```
```bash
docker inspect container_name # контейнеры
docker inspect container_id
docker inspect image_name # образы
docker inspect image_id
docker inspect volume_name # тома
docker inspect network_name # сети
```
