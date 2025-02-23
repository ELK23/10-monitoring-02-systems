# Домашнее задание к занятию "13.Системы мониторинга"

## Обязательные задания

1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя 
платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой 
осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы
выведите в мониторинг и почему?
### Решение
1. Нагрузка на процессор
Причина: Вычисления сильно загружают CPU, что может привести к деградации производительности или падению сервиса.
Метрики:
Общая загрузка CPU
Загрузка CPU по ядрам
Средняя загрузка
2. Использование памяти
Причина: Если платформа использует много памяти, это может привести к сбоям или OOM (Out of Memory).
Метрики:
Общий объем используемой памяти
Доля свободной памяти
Использование swap
3. Доступное дисковое пространство
Причина: Отчеты сохраняются на диск, поэтому нехватка места приведет к сбоям.
Метрики:
Свободное место на диске
Использование inode
4. Время ответа HTTP
Причина: Высокая загрузка CPU может замедлить отклик сервиса, что ухудшит пользовательский опыт.
Метрики:
Время ответа на HTTP-запросы
Количество запросов в секунду
Код ответа (ошибки 5xx/4xx)
#
2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, 
что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы 
можете ему предложить?
### Решение
1. Время отклика сервиса
2. Успешность запросов
3. Доступность сервиса
4. Среднее время выполнения вычислений
5. Количество активных пользователей
6. Очередь на выполнение задач
#
3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою 
очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, 
чтобы разработчики получали ошибки приложения?
### Решение
1. Настроить логирование в системный журнал (journald/syslog)
Если приложения работают на Linux, они могут писать ошибки в journald (если systemd) или syslog.
2. Настроить логирование в файл + отправка через Logrotate
Если приложение пишет логи в файл (/var/log/app.log), можно использовать logrotate для отправки новых ошибок разработчикам.
3. Отправка ошибок в Telegram/Slack через shell-скрипт
Если разработчикам нужен онлайн-доступ, можно настроить простой скрипт, который будет пересылать ERROR-логи в Telegram или Slack.
#
4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. 
Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 
70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?
### Решение
Ошибка в том, что данное выражение не учитывает коды 1xx и 3xx, которые при определенных настройках могут занимать значительный объем requests, для того чтобы получить точное значение требуется пересмотреть выражение, например (summ_2xx_requests+summ_3xx_requests)/summ_all_requests
#
5. Опишите основные плюсы и минусы pull и push систем мониторинга.
### Решение
Плюсы pull-модели:
Простая настройка – сервису не нужно знать, куда отправлять метрики, он просто отдает их по запросу.
Гибкость в сборе данных – можно запрашивать метрики с разной частотой.
Легкая отладка – можно вручную зайти на http://service:port/metrics и увидеть метрики.
Удобно для автоматического обнаружения сервисов – Prometheus находит новые сервисы сам.
Контроль со стороны мониторинга – можно отключить проблемный сервис без влияния на других.
Минусы pull-модели:
Проблемы с динамическими IP-адресами – если сервис меняет IP (например, в Kubernetes), его сложнее найти.
Не подходит для краткоживущих сервисов – если сервис живет 5 секунд, Prometheus может не успеть его опросить.
Сложности при работе через NAT или файрволы – мониторинг-система должна иметь доступ к сервисам.

Плюсы push-модели:
Хорошо работает с динамическими сервисами – сервис неважно, где запущен, он просто отправляет данные.
Подходит для краткоживущих сервисов – контейнеры, серверлесс-функции могут сразу отправить метрики и умереть.
Проще работать через NAT и файрволы – сервису не нужен открытый порт, он просто делает POST запрос.
Может работать в реальном времени – если настроить streaming, можно сразу реагировать на события.
Минусы push-модели:
Сложнее отладка – если метрики не приходят, непонятно, проблема в сервисе или в хранилище.
Нет централизованного контроля – сервисы должны сами знать, куда отправлять данные.
Потеря метрик при сбоях – если хранилище временно недоступно, метрики могут потеряться.
#
6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

    - Prometheus 
    - TICK
    - Zabbix
    - VictoriaMetrics
    - Nagios
### Модели сбора данных в системах мониторинга

| Система          | Pull | Push | Гибрид |
|-----------------|------|------|--------|
| **Prometheus**       |  Да  |  Нет  |  Гибрид (с Pushgateway) |
| **TICK (Telegraf, InfluxDB, Chronograf, Kapacitor)** |  Нет |  Да  |  Нет |
| **Zabbix**          |  Да (Passive mode)  |  Да (Active mode)  |  Гибрид |
| **VictoriaMetrics** |  Да  |  Да  |  Гибрид |
| **Nagios**         |  Да  |  Нет  |  Нет |
#
7. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, 
используя технологии docker и docker-compose.

В виде решения на это упражнение приведите скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`). 

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`
### Скриншот ПО
![image](https://github.com/user-attachments/assets/2fc16f5c-a6ec-492c-a380-ce89d4f0359e)
#
8. Перейдите в веб-интерфейс Chronograf (http://localhost:8888) и откройте вкладку Data explorer.
        
    - Нажмите на кнопку Add a query
    - Изучите вывод интерфейса и выберите БД telegraf.autogen
    - В `measurments` выберите cpu->host->telegraf-getting-started, а в `fields` выберите usage_system. Внизу появится график утилизации cpu.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации cpu из веб-интерфейса.
### Скриншот метрик
![image](https://github.com/user-attachments/assets/08c0e651-ea67-4b4e-b039-8114d3d628b7)
#
9. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). 
Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и 
режима privileged:
```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```

После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в 
веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.
