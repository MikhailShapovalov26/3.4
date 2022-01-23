Домашнее задание к занятию "3.4. Операционные системы, лекция 2"
==================
1. Установка node_exporter и добавление его в автозагрузку. 
1.1 
Скачиваем его с помощью команды wget gz архив, далее распоковываем и скидываем файлы /usr/local/bin. Так как в будущем старт возможен от прав другого пользователя, назначаем права на эту папку другому пользователю.

1.2 Конфиг файл

        nano /etc/systemd/system/node_exporter.service
        [Unit]
        Description=Prometheus Node Exporter
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=node_exporter
        Group=node_exporter
        Type=simple
        ExecStart=/usr/local/bin/node_exporter

        [Install]
        WantedBy=multi-user.target

1.3 У меня возникли сложности со стартом, выдавал ошибку

        level=error ts=2022-01-23T17:43:24.190Z caller=node_exporter.go:194 err="listen tcp :9100: bind: address already in use"
Далее пришлось смотреть какой процесс занял данный порт
        fuser -k 9100/tcp
И убивать его kill -9 номер процесса

1.4 Повторный запуск, и всё прошло отлично.

        systemctl restart node_exporter
        systemctl status node_exporter
            ● node_exporter.service - Prometheus Node Exporter
            Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
            Active: active (running) since Sun 2022-01-23 20:46:51 MSK; 3s ago

1.5
Выполняем команду для автоматического запуска при reboot системе. 
        systemctl enable --now node_exporter
Тестируем, после reboot системы now_exporter поднялся автоматически
Тестируем
        curl http://localhost:9100/metrics

2. 
Что-бы активировать дополнительные иструменты для сбора информации при запуске нужно добавить флаги --collector. имя модуля.
к примеру --collector.processes или --collector.ntp(синхронизация времени NTP) --collecttor.logind
3. 

        lsof -i :19999

        COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
        chrome  1429 mikhail   46u  IPv4  64378      0t0  TCP localhost:55768->localhost:19999 (ESTABLISHED)
        netdata 6234 netdata    4u  IPv4  61663      0t0  TCP localhost:19999 (LISTEN)
        netdata 6234 netdata   76u  IPv4  64379      0t0  TCP localhost:19999->localhost:55768 (ESTABLISHED)

Метрики которые мне попались на глаза на главной странице, после входа.
disk
Total Disk I/O, for all physical disks. You can get detailed information about each disk at the Disks section and per application Disk usage at the Applications Monitoring section. Physical are all the disks that are listed in /sys/block, but do not exist in /sys/devices/virtual/block.
ram
System Random Access Memory (i.e. physical memory) usage.
4. 
        dmesg | grep "Hypervisor detected"
        .... grep virtualiz

5. 
sysctl это утилита предназначенная для управления параметрами ядра во время работы. позволяет читать и изменгять параметры ядра. К примеру, сегменты разделяемой памяти, ограничение на число запущеных процессов, функции маршуртизации. 
        /sbin/sysctl -n fs.nr_open
        1048576
1024*1024

        cat /proc/sys/fs/file-max 
        9223372036854775807

 -H        use the `hard' resource limit  
 -n        the maximum number of open file descriptors

            ulimit -Hn
            1048576
6. 
        ps a | grep sleep
        11219 pts/2    S+     0:00 sleep 1h

        nsenter --target 11219 --mount --pid

7. 
Спасибо, дебиан завис. Только на горячую его смог выключить. Форк бомба. Рабочая. Тест пройден.  
:(){ :|:& };:  
Для того, чтоб понять, нужно : заменить на f

        f(){
         f|f &
        }
        f
Получается, функция которая паралельно пускает два своих экземпляра. И какждый из них пускает ещё по два. Замкнутый круг получается. Если, мы не поставим ограничение на кол-во процессов, то ресурс физической памяти быстро себя изчерпает. Что у меня и произошло. Но было весело. 

        free -m
            total        used        free      shared  buff/cache   available
            Mem:           28095        3373       22482         204        2240       24135
            Swap:            974           0         974
