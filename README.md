# 1. Установка Grafana Stack

1. Установка Docker'а

        sudo yum install wget

![изображение](https://github.com/user-attachments/assets/daaf8789-ede0-4cdc-a052-7be5a034e6fd)

    sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo

![изображение](https://github.com/user-attachments/assets/47526f24-bdcf-4edc-ae0e-b6967b8eb3b5)

        sudo yum install docker-ce docker-ce-cli containerd.io

![изображение](https://github.com/user-attachments/assets/13b821ca-5e78-4ab3-9b44-b8558dee38fd)

        sudo systemctl enable docker --now

![изображение](https://github.com/user-attachments/assets/fa710212-70c4-43fe-be89-6642ab5dc3db)

2. Установка Compose

        sudo yum install curl

![изображение](https://github.com/user-attachments/assets/2e171846-bda7-4dc5-b030-dadba206493e)

    COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)

![изображение](https://github.com/user-attachments/assets/f615c841-47b6-4095-83ae-39ff400a6155)

    sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose

![изображение](https://github.com/user-attachments/assets/f77ab18d-eee3-417b-b901-47412784f78c)

    sudo chmod +x /usr/bin/docker-compose

![изображение](https://github.com/user-attachments/assets/55fcbd1b-74a8-42e9-bf5f-b81eb244931f)

    sudo docker-compose --version

![изображение](https://github.com/user-attachments/assets/034d1486-0110-4634-a347-a20a6509d67e)

3. Делаем Grafana

        sudo yum install git

![изображение](https://github.com/user-attachments/assets/eb91a500-a852-4eeb-ad79-d4096c848b94)

    sudo git clone https://github.com/skl256/grafana_stack_for_docker.git

![изображение](https://github.com/user-attachments/assets/56a5bb7f-2095-4912-a80b-b866a0ce8842)

Заходим в папку grafana_stack_for_docker

    cd grafana_stack_for_docker

cd .. - возвращает в папку выше

![изображение](https://github.com/user-attachments/assets/685bbaf6-ce53-4d93-9e1a-b153cfa38a44)

(После этого можно вставлять готовый docker-compose)

Cоздаем папки двумя разными способами

     sudo mkdir -p /mnt/common_volume/swarm/grafana/config

     sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}

![изображение](https://github.com/user-attachments/assets/42619b7f-4ed9-43b5-87c8-b516d23eead6)

Выдаем права

     sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}

Создаем файл

     sudo touch /mnt/common_volume/grafana/grafana-config/grafana.ini

Копирование файлов

     sudo cp config/* /mnt/common_volume/swarm/grafana/config/

Переименовывание файла

     sudo mv grafana.yaml docker-compose.yaml

![изображение](https://github.com/user-attachments/assets/82e25fc9-9862-4024-aa4e-2c44be3617ef)

     sudo docker compose up -d

![изображение](https://github.com/user-attachments/assets/825260ae-30aa-4cf6-8189-14217ee71030)

После запуска, нам нужно будет на время остановить docker - sudo docker compose stop

После остановки, продолжаем работать дальше

    sudo vi docker-compose.yaml

Затем в docker-compose нужно вставить node-exporter:

    node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter < запомнить название name
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default

![изображение](https://github.com/user-attachments/assets/3ce7af23-d9ca-49fc-958d-459c47ee01a2)

Заходим в другую папку

    cd /mnt/common_volume/swarm/grafana/config
    
![изображение](https://github.com/user-attachments/assets/82e68b02-a5fc-44f3-96e3-a5b038e22cd7)

     sudo vi prometheus.yaml

Далее нужно исправить targets: на exporter:9100

![изображение](https://github.com/user-attachments/assets/5499a6c0-5f37-4e5d-8b0e-be648543503d)

:w![изображение](https://github.com/user-attachments/assets/0a29244a-c829-486d-adf3-a3d22ad041c5)

После этого запускаем Docker

# 2. Установка Prometheus

переходим на сайт http://localhost:3000

Логин и пароль: Admin

![изображение](https://github.com/user-attachments/assets/65295454-6eb6-43f3-91fd-947f9ef72091)

в меню выбираем вкладку Dashboards, нажимаем на кнопку "+ create dashnoard", затем на "+ add visualization" и после на "configure a new data source".
выбираем Prometheus, в connection нужно будет ввести: "http://prometheus:9090".
в authentication нужно будет поставить "Basic authentication" и ввести логин и пароль: admin, после этого нажимаем на "Save & test" и должно показать зеленую галочку
Authentication

![изображение](https://github.com/user-attachments/assets/63e96b38-d0bb-4caf-9edf-c741b041f71b)

возвращаемся обратно в Dashboards и нажимаем снова Create Dashboard
нажимаем Import dashboard и в поле "Find and import dashboards" нужно будет ввести "1860", потом нужно будет выбрать Prometheus и импортировать dashboard

![изображение](https://github.com/user-attachments/assets/ab0d7adb-3d14-4138-b2dc-82eba6cf24c9)


# 3. Делаем VictoriaMetrics

Для начала зайдем в нужную папку

     cd grafana_stack_for_docker

Открываем docker-compose

     sudo vi docker-compose.yaml

После prometheus вставляем vmagent

      vmagent:
    container_name: vmagent
    image: victoriametrics/vmagent:v1.105.0
    depends_on:
      - "victoriametrics"
    ports:
      - 8429:8429
    volumes:
      - vmagentdata:/vmagentdata
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--promscrape.config=/etc/prometheus/prometheus.yml"
      - "--remoteWrite.url=http://victoriametrics:8428/api/v1/write"
    restart: always
      # VictoriaMetrics instance, a single process responsible for
      # storing metrics and serve read requests.
      victoriametrics:
        container_name: victoriametrics
        image: victoriametrics/victoria-metrics:v1.105.0
        ports:
          - 8428:8428
          - 8089:8089
          - 8089:8089/udp
          - 2003:2003
          - 2003:2003/udp
          - 4242:4242
        volumes:
          - vmdata:/storage
        command:
          - "--storageDataPath=/storage"
          - "--graphiteListenAddr=:2003"
          - "--opentsdbListenAddr=:4242"
          - "--httpListenAddr=:8428"
          - "--influxListenAddr=:8089"
          - "--vmalert.proxyURL=http://vmalert:8880"
        restart: always

Затем возвращаемся в localhost и переходим во вкладку connection, нажимаем на data sources и затем на prometheus. Нам нужно будет поменять в connection с "http://prometheus:9090" на "http://victoriametrics:8428" и поменять название на "VikMetric" 

![изображение](https://github.com/user-attachments/assets/7bd82a0a-2453-44b0-b8a6-7c7b8342d6ec)

![изображение](https://github.com/user-attachments/assets/8ead6550-3e6d-48c7-9108-d26c1c6804f9)

После возвращаемся в Dashboards и нажимаем на Add visualization, на этот раз выбираем VikMetric, в созданной Dashboard нам нужно будет выбрать вкладку "code"

![изображение](https://github.com/user-attachments/assets/d022cfac-6f20-40e6-bbcd-0c7203d6847c)

После переходим в терминал и пишем эти команды:

     echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus

     curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'

![изображение](https://github.com/user-attachments/assets/0cb89ab1-7474-4a59-83f0-e95fe2b6b496)

Копируем переменную OILCOINT_metric1 и вставляем в code и нажимаем run queries

![изображение](https://github.com/user-attachments/assets/ebd50971-20a5-41a6-98ed-6d0c04f55a46)

