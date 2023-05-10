# Pool JMS (Java Message Service)

## <p id="contents">Оглавление</p>

<ul>
<li><a href="#01">Описание проекта</a></li>
<li><a href="#02">Стек технологий</a></li>
<li><a href="#03">Требования к окружению</a></li>
<li><a href="#04">Сборка и запуск проекта</a>
    <ol type="1">
        <li><a href="#0401">Сборка проекта</a></li>
        <li><a href="#0402">Запуск проекта</a></li>
    </ol>
</li>
<li><a href="#05">Взаимодействие с приложением</a>
    <ol  type="1">
        <li><a href="#0501">Queue. Запрос на добавление данных (POST)</a></li>
        <li><a href="#0502">Queue. Запрос на получение данных (GET)</a></li>
        <li><a href="#0503">Topic. Запрос на создание топика (POST)</a></li>
        <li><a href="#0504">Topic. Запрос на получение данных (GET)</a></li>
    </ol>
</li> 
<li><a href="#contacts">Контакты</a></li>
</ul>

## <p id="01">Описание проекта</p>

Приложение является аналогом асинхронной очереди, ожидает клиентов 
через запущенный Socket. Работа осуществляется через HTTP протокол. 
Клиенты могут быть двух типов: отправители (publisher) и получатели (subscriber).

Функционал:

- Добавление сообщений в очереди посредством POST-методов (publisher).
- Получение сообщений из очередей посредством GET-методов (subscriber).
- Добавление тем, подписчиков.

Архитектура проекта:

![alt text](images/jms_img_1.jpg)

<p><a href="#contents">К оглавлению</a></p>

## <p id="02">Стек технологий</p>

- Java 14
- JUnit 4
- Maven 3.8
- Lombok
 
Инструменты:

- Javadoc, JaCoCo, Checkstyle

<p><a href="#contents">К оглавлению</a></p>

## <p id="03">Требования к окружению</p>

Java 14, Maven 3.8

<p><a href="#contents">К оглавлению</a></p>

## <p id="04">Сборка и запуск проекта</p>

Для выполнения действий данного раздела необходимо установить
и настроить систему сборки проектов Maven.
По умолчании проект компилируется и собирается в директорию target.

<p><a href="#contents">К оглавлению</a></p>

### <p id="0401">1. Сборка проекта</p>

Команда для сборки в jar
`mvn clean package -DskipTests`

<p><a href="#contents">К оглавлению</a></p>

### <p id="0402">2. Запуск проекта</p>

Команда для запуска
`java -jar target/pool_jms.jar`

<p><a href="#contents">К оглавлению</a></p>

## <p id="05">Взаимодействие с приложением</p>

В качестве клиента можно использовать cURL. `https://curl.se/download.html`
В качестве протокола используется HTTP.
Pool будет иметь два режима: `queue`, `topic`.

<p><a href="#contents">К оглавлению</a></p>

### <p id="0501">1.Queue. Запрос на добавление данных (POST)</p>

Отправитель посылает запрос на добавление данных с указанием очереди `weather`
и значением параметра `temperature=18`. Сообщение помещается в конец очереди.
Если очереди нет в сервисе, то создается новая и в нее помещается сообщение.
POST запрос должен добавить элементы в очередь weather.

`curl -X POST -d "temperature=18" http://localhost:9000/queue/weather`

![alt text](images/jms_img_2.jpg)

<p><a href="#contents">К оглавлению</a></p>

### <p id="0502">2.Queue. Запрос на получение данных (GET)</p>

Получатель посылает запрос на получение данных с указанием очереди. Сообщение
забирается из начала очереди и удаляется. Если в очередь приходят несколько 
получателей, то они поочередно получают сообщения из очереди.
Каждое сообщение в очереди может быть получено только одним получателем.
`queue` указывает на режим «очередь».
`weather` указывает на имя очереди.
GET запрос должен получить элементы из очереди weather:

`curl -X GET http://localhost:9000/queue/weather`

![alt text](images/jms_img_3.jpg)

Повторный запрос показывает отсутствие данных

![alt text](images/jms_img_4.jpg)

<p><a href="#contents">К оглавлению</a></p>

### <p id="0503">3.Topic. Запрос на создание топика (POST)</p>

Отправитель посылает запрос на добавление данных с указанием
топика `weather` и значением параметра `temperature=18`.
Создание топика `weather`:

`curl -X POST -d "temperature=18" http://localhost:9000/topic/weather`

![alt text](images/jms_img_5.jpg)

Сообщение помещается в конец каждой индивидуальной очереди получателей.
Если топика нет в сервисе, то топик создается, а данные игнорируются.
Возвращается ответ: `Bad Request`, статус `400`.

Занесение данных температуры в топик `weather` и все индивидуальные очереди:

`curl -X POST -d "temperature=18" http://localhost:9000/topic/weather`

Возвращается ответ: `Created`, статус `200`.

![alt text](images/jms_img_7.jpg)

<p><a href="#contents">К оглавлению</a></p>

### <p id="0503">4.Topic. Запрос на получение данных (GET)</p>

Получатель посылает запрос на получение данных с указанием топика.
Если топик отсутствует, то создается новый топик с новой индивидуальной очередью. 

Создание индивидуальной очереди `1` в топике `weather`:

`curl -X GET http://localhost:9000/topic/weather/1`

Возвращается ответ: `Topic Not Found`, статус `404`.

![alt text](images/jms_img_6.jpg)

А если топик присутствует, то сообщение забирается из начала индивидуальной очереди 
получателя, удаляется и возвращается клиенту.

![alt text](images/jms_img_8.jpg)

Возвращается ответ: `Данные`, статус `200`.

Все последующие сообщения от отправителей с данными для этого топика
помещаются в эту очередь тоже.
Таким образом в режиме `topic` для каждого потребителя своя будет уникальная очередь
с данными, в отличие от режима "queue", где все потребители получают данные из одной
и той же очереди.

<p><a href="#contents">К оглавлению</a></p>

## <p id="contacts">Контакты</p>

[![alt-text](https://img.shields.io/badge/-telegram-grey?style=flat&logo=telegram&logoColor=white)](https://t.me/T_AlexME)
&nbsp;&nbsp;
[![alt-text](https://img.shields.io/badge/@%20email-005FED?style=flat&logo=mail&logoColor=white)](mailto:amemelyanov@yandex.ru)
&nbsp;&nbsp;

<p><a href="#contents">К оглавлению</a></p>