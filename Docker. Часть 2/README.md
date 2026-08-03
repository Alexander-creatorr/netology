# Домашнее задание к занятию «Docker. Часть 2»

### Оформление домашнего задания

1. Домашнее задание выполните в вашем git-репозиториий(предпочтительно) или [Google Docs](https://docs.google.com/) и отправьте на проверку ссылку на ваш документ в личном кабинете.  
1. В названии файла укажите номер лекции и фамилию студента. Пример названия: 6.4. Docker. Часть 2 — Александр Александров.
2. Код решения размещайте в отдельном файле на вашем Google-диске, это облегчит проверку вашей работы.
3. Перед отправкой проверьте, что доступ для просмотра открыт всем, у кого есть ссылка. Если нужно прикрепить дополнительные ссылки, добавьте их в свой Google Docs.

Вы можете прислать решение в виде ссылки на ваш репозийторий в GitHub, для этого воспользуйтесь [шаблоном для домашнего задания](https://github.com/netology-code/sys-pattern-homework).

**Правила выполнения заданий к занятию «6.4. Docker. Часть 2»**

- Все задания выполняйте на основе [конфигов](https://github.com/netology-code/sdvps-homeworks/tree/main/lecture_demos/6-04) из лекции. 
- В заданиях описаны те параметры, которые необходимо изменить. 
- Если параметр не упомянут вообще, значит, его нужно оставить таким, какой он был в лекции. 
- Если в каком-то задании, например, в задании 2, нужно изменить параметр, подразумевается, что во всех следующих заданиях будет использоваться уже изменённый параметр.
- Проверяйте правильность отступов. Очень важно их соблюдать, так как это влияет на структуру данных.
- Выполнив все задания без звёздочки, вы должны получить полнофункциональный сервис.

Любые вопросы по решению задач задавайте в разделе "Вопросы по заданию".

### Дополнительные примеры
Примеры различных композ проектов от разработчиков Docker: [https://github.com/docker/awesome-compose/blob/master/wireguard/compose.yaml](https://github.com/docker/awesome-compose/tree/master)

### Дополнительная документация:
  - [блок networks: в compose](https://docs.docker.com/compose/compose-file/06-networks/)
  - [блок volumes: в compose](https://docs.docker.com/compose/compose-file/07-volumes/)


---

### Задание 1

**Напишите ответ в свободной форме, не больше одного абзаца текста.**

Установите Docker Compose и опишите, для чего он нужен и как может улучшить лично вашу жизнь.

---

### Задание 2 

**Выполните действия и приложите текст конфига на этом этапе.** 

Создайте файл docker-compose.yml и внесите туда первичные настройки: 

 * version;
 * services;
 * volumes;
 * networks.

При выполнении задания используйте подсеть 10.5.0.0/16.
Ваша подсеть должна называться: <ваши фамилия и инициалы>-my-netology-hw.
Все приложения из последующих заданий должны находиться в этой конфигурации.

---

### Решение 2 
```
version: '3'
services:

volumes:

networks:
  oparinad-my-netology-hw:
    ipam:
      driver: default
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1
```

---


### Задание 3 

**Выполните действия:** 

1. Создайте конфигурацию docker-compose для Prometheus с именем контейнера <ваши фамилия и инициалы>-netology-prometheus. 
2. Добавьте необходимые тома с данными и конфигурацией (конфигурация лежит в репозитории в директории [6-04/prometheus](https://github.com/netology-code/sdvps-homeworks/tree/main/lecture_demos/6-04/prometheus) ).
3. Обеспечьте внешний доступ к порту 9090 c докер-сервера.

---

### Решение 3

docker-compose.yaml

```
version: '3'

services:
    oparinad-netology-prometheus:
      image: prom/prometheus:v3.13.2
      container_name: prometheus-v3.13.2
      ports: 
        - "9090:9090"
      volumes:
        - ./prometheus:/etc/prometheus #	Монтирует вашу локальную папку ./prometheus внутрь контейнера. Данные хранятся на хосте.
        - prometheus-data:/prometheus  #  Docker управляет томом сам. Данные хранятся в /var/lib/docker/volumes/
      networks:
        - oparinad-my-netology-hw
    
networks:
  oparinad-my-netology-hw:
    ipam:
      driver: default
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1


volumes:
    prometheus-data:
```

prometheus.yml

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  # - job_name: "docker-server"
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
  #   static_configs:
  #     - targets: ["172.17.0.1:9100"]

  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ["pushgateway:9091"]
```
---

### Задание 4 

**Выполните действия:**

1. Создайте конфигурацию docker-compose для Pushgateway с именем контейнера <ваши фамилия и инициалы>-netology-pushgateway. 
2. Обеспечьте внешний доступ к порту 9091 c докер-сервера.

---

### Решение 4

docker-compose.yaml

```
version: '3'

services:
    prometheus:
      image: prom/prometheus:v3.13.2
      container_name: oparinad-netology-prometheus
      ports: 
        - "9090:9090"
      volumes:
        - ./prometheus:/etc/prometheus #	Монтирует вашу локальную папку ./prometheus внутрь контейнера. Данные хранятся на хосте.
        - prometheus-data:/prometheus  #  Docker управляет томом сам. Данные хранятся в /var/lib/docker/volumes/
      networks:
        - oparinad-my-netology-hw
    
    pushgateway:
      container_name: oparinad-netology-pushgateway
      image: prom/pushgateway:v1.11.3
      ports:
        - "9091:9091"
      networks:
      - oparinad-my-netology-hw
      

networks:
  oparinad-my-netology-hw:
    ipam:
      driver: default
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1


volumes:
    prometheus-data:

```

### Задание 5 

**Выполните действия:** 

1. Создайте конфигурацию docker-compose для Grafana с именем контейнера <ваши фамилия и инициалы>-netology-grafana. 
2. Добавьте необходимые тома с данными и конфигурацией (конфигурация лежит в репозитории в директории [6-04/grafana](https://github.com/netology-code/sdvps-homeworks/blob/main/lecture_demos/6-04/grafana/custom.ini).
3. Добавьте переменную окружения с путем до файла с кастомными настройками (должен быть в томе), в самом файле пропишите логин=<ваши фамилия и инициалы> пароль=netology.
4. Обеспечьте внешний доступ к порту 3000 c порта 80 докер-сервера.

---

### Решение 5

docker-compose.yaml

```
version: '3'

services:
    prometheus:
      image: prom/prometheus:v3.13.2
      container_name: oparinad-netology-prometheus
      ports: 
        - "9090:9090"
      volumes:
        - ./prometheus:/etc/prometheus #	Монтирует вашу локальную папку ./prometheus внутрь контейнера. Данные хранятся на хосте.
        - prometheus-data:/prometheus  #  Docker управляет томом сам. Данные хранятся в системной папке Docker - /var/lib/docker/volumes/
      networks:
        - oparinad-my-netology-hw
    
    pushgateway:
      container_name: oparinad-netology-pushgateway
      image: prom/pushgateway:v1.11.3
      ports:
        - "9091:9091"
      networks:
      - oparinad-my-netology-hw

    grafana:
      container_name: oparinad-netology-grafana
      image: grafana/grafana:12.4.6
      ports:
        - "80:3000"
      volumes:
        - ./grafana/grafana.ini:/etc/grafana/grafana.ini 
      networks:
        - oparinad-my-netology-hw


      

networks:
  oparinad-my-netology-hw:
    ipam:
      driver: default
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1


volumes:
    prometheus-data:
    grafana-data:

```
---

### Задание 6 

**Выполните действия.**

1. Настройте поочередность запуска контейнеров.
2. Настройте режимы перезапуска для контейнеров.
3. Настройте использование контейнерами одной сети.
5. Запустите сценарий в detached режиме.

---

### Решение 6

docker-compose.yaml

```
version: '3'

services:
    prometheus:
      image: prom/prometheus:v3.13.2
      container_name: oparinad-netology-prometheus
      ports: 
        - "9090:9090"
      volumes:
        - ./prometheus:/etc/prometheus #	Монтирует вашу локальную папку ./prometheus внутрь контейнера. Данные хранятся на хосте.
        - prometheus-data:/prometheus  #  Docker управляет томом сам. Данные хранятся в системной папке Docker - /var/lib/docker/volumes/
      restart: always
      networks:
        - oparinad-my-netology-hw
    
    pushgateway:
      container_name: oparinad-netology-pushgateway
      image: prom/pushgateway:v1.11.3
      depends_on: 
        - prometheus # Запустить только после prometheus
      restart: unless-stopped # no	Никогда не перезапускать (по умолчанию); always	Всегда перезапускать, даже если остановить вручную (кроме случая docker stop); unless-stopped	✅ Перезапускать всегда, ЕСЛИ только ты сам не остановил его вручную (docker stop); on-failure	Перезапускать только если контейнер упал с ошибкой (код возврата != 0)
      ports:
        - "9091:9091"
      networks:
      - oparinad-my-netology-hw

    grafana:
      container_name: oparinad-netology-grafana
      image: grafana/grafana:12.4.6
      depends_on: 
        - pushgateway
      restart: unless-stopped
      ports:
        - "80:3000"
      volumes:
        - ./grafana/grafana.ini:/etc/grafana/grafana.ini 
      networks:
        - oparinad-my-netology-hw


      

networks:
  oparinad-my-netology-hw:
    ipam:
      driver: default
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1


volumes:
    prometheus-data:
    grafana-data:

```

### Задание 7 

**Выполните действия.**
1. Выполните запрос в Pushgateway для помещения метрики <ваши фамилия и инициалы> со значением 5 в Prometheus: ```echo "<ваши фамилия и инициалы> 5" | curl --data-binary @- http://localhost:9091/metrics/job/netology```.
2. Залогиньтесь в Grafana с помощью логина и пароля из предыдущего задания.
3. Cоздайте Data Source Prometheus (Home -> Connections -> Data sources -> Add data source -> Prometheus -> указать "Prometheus server URL = http://prometheus:9090" -> Save & Test).
4. Создайте график на основе добавленной в пункте 5 метрики (Build a dashboard -> Add visualization -> Prometheus -> Select metric -> Metric explorer -> <ваши фамилия и инициалы -> Apply.

В качестве решения приложите:

* docker-compose.yml **целиком**;
* скриншот команды docker ps после запуске docker-compose.yml;
* скриншот графика, постоенного на основе вашей метрики.

---

### Решение 7

docker-compose.yaml

```
version: '3'

services:
    prometheus:
      image: prom/prometheus:v3.13.2
      container_name: oparinad-netology-prometheus
      ports: 
        - "9090:9090"
      volumes:
        - ./prometheus:/etc/prometheus #	Монтирует вашу локальную папку ./prometheus внутрь контейнера. Данные хранятся на хосте.
        - prometheus-data:/prometheus  #  Docker управляет томом сам. Данные хранятся в системной папке Docker - /var/lib/docker/volumes/
      restart: always
      networks:
        - oparinad-my-netology-hw
    
    pushgateway:
      container_name: oparinad-netology-pushgateway
      image: prom/pushgateway:v1.11.3
      depends_on: 
        - prometheus # Запустить только после prometheus
      restart: unless-stopped # no	Никогда не перезапускать (по умолчанию); always	Всегда перезапускать, даже если остановить вручную (кроме случая docker stop); unless-stopped	✅ Перезапускать всегда, ЕСЛИ только ты сам не остановил его вручную (docker stop); on-failure	Перезапускать только если контейнер упал с ошибкой (код возврата != 0)
      ports:
        - "9091:9091"
      networks:
      - oparinad-my-netology-hw

    grafana:
      container_name: oparinad-netology-grafana
      image: grafana/grafana:12.4.6
      depends_on: 
        - pushgateway
      restart: unless-stopped
      ports:
        - "80:3000"
      volumes:
        - ./grafana/grafana.ini:/etc/grafana/grafana.ini 
      networks:
        - oparinad-my-netology-hw


      

networks:
  oparinad-my-netology-hw:
    ipam:
      driver: default
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1


volumes:
    prometheus-data:
    grafana-data:

```

<img width="1587" height="795" alt="image" src="https://github.com/user-attachments/assets/c7915285-c11c-4f2b-bc15-81436f04829a" />

---

### Задание 8

**Выполните действия:** 

1. Остановите и удалите все контейнеры одной командой.

В качестве решения приложите скриншот консоли с проделанными действиями.

---

### Решение 9

<img width="968" height="215" alt="image" src="https://github.com/user-attachments/assets/61aef79b-7697-43c6-a845-4e813472de05" />

---

## Дополнительные задания* (со звёздочкой)

Их выполнение необязательное и не влияет на получение зачёта по домашнему заданию. Можете их решить, если хотите лучше разобраться в материале.

---

### Задание 9* 

**Выполните действия:** 

1. Создайте конфигурацию docker-compose для Alertmanager с именем контейнера <ваши фамилия и инициалы>-netology-alertmanager. 
2. Добавьте необходимые тома с данными и [конфигурацией](https://github.com/netology-code/sdvps-homeworks/tree/main/6-04/alertmanager), сеть, режим и очередность запуска.
3. Обновите конфигурацию Prometheus (необходимые изменения ищите в презентации или документации) и перезапустите его. 
4. Обеспечьте внешний доступ к порту 9093 c докер-сервера.

В качестве решения приложите скриншот с событием из Alertmanager.

---

### Задание 10* 

Запустите свой сценарий на чистом железе без предзагруженных образов.

**Ответьте на вопросы в свободной форме:**

1. Опишите выполненный вами процесс развертывания сценария.
2. Как вы думаете зачем может понадобиться такой способ развертывания?
