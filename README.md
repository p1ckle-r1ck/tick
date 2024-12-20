## Обязательные задания

1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя
платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой
осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы
выведите в мониторинг и почему?

### Ответ

Я бы предложил отслеживать следующие метрики:

- CPU LA, т.к было отмечено, что вычесления загружают ЦПУ.
- RAM/Swap, вычисления могут потреблять значительные объемы памяти, что способно привести к сбоям из-за нехватки ресурсов.
- IOPS, т.к отчеты сохраняются на диск необходимо мониторить производительность дисковой подсистемы.
- FS, т.к отчеты сохраняются на диск, необходимо отслеживать свободное место на диске.
- inodes, данную метрику необходимо отслеживать из-за того, что если индексные дескрипторов будут переполнены, то дальнейшая запись отчетов может быть не возможна
- IOwait, количество I/O-запросов в очереди, если показатели будут высокими, то запись отчетов на дисковую подсистему будет проходить слишком медленно.
- NetTraffic т.к платформа взаимодействует по HTTP. Задержки или ошибки сети могут повлиять на доступность.
- HTTP запрсы, т.к это основной способ взаимодействия сплатформой, необходимо отслеживать коллисчетсво запросов HTTP, распределение статусов ответов.

#

2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал,
что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы
можете ему предложить?

### Ответ

Предложу заключить SLA c клиентом,внутри опираться на SLO и SLI.

#

3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою
очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации,
чтобы разработчики получали ошибки приложения?

### Ответ

Я решил развернуть Graylog, я думаю, что данная платформа может решить поставленные задачи.

#

4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов.
Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше
70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?

### Ответ

Формула не правильная, так как коректный ответы http сервера может быть не только 200-ые, но и 300-ые и 100-ые.

#

5. Опишите основные плюсы и минусы pull и push систем мониторинга.

### Ответ

#### Push модель

Плюсы:

- Масшатбируемость
- Совместимость с агентами
- Гибкость

Минусы:

- Нет централизованного управления
- Риск потери данных
- Сложность настройки

Pull модель:

Плюсы:

- Централизованное управление
- Простая отладка

Минусы:

- Ограничения масштабируемости
- Задержка данных
- Ограниченная гибкость

#

6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

    - Prometheus
    - TICK
    - Zabbix
    - VictoriaMetrics
    - Nagios

### Ответ

Pull модель: Nagios, Prometheus
Push модель: TICK
Pull и Push модель: Zabbix, VictoriaMetrics

#


7. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк,
используя технологии docker и docker-compose.

В виде решения на это упражнение приведите скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`).

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`

### Ответ

![Image](screenshots/tick_1.png)

8. Перейдите в веб-интерфейс Chronograf (<http://localhost:8888>) и откройте вкладку Data explorer.

    - Нажмите на кнопку Add a query
    - Изучите вывод интерфейса и выберите БД telegraf.autogen
    - В `measurments` выберите cpu->host->telegraf-getting-started, а в `fields` выберите usage_system. Внизу появится график утилизации cpu.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации cpu из веб-интерфейса.

### Ответ

![Image](screenshots/tick_2.png)


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

### Ответ

![Image](screenshots/tick_3.png)
