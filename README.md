# Booking 
Семестровый проект по курсу "highload" в технопарке VK

## Содержание
- [1. Тема и целевая аудитория](#1)
  - [1.1 Тема](#2)
  - [1.2 MVP](#3)
  - [1.3 Целевая аудитория](#4)
- [2. Расчет нагрузки](#5)
  - [2.1 Продуктовые метрики](#6)
  - [2.2 Расчет количества запросов в секунду (RPS)](#7)
- [3. Логическая схема базы данных](#8)
  - [3.1 Детали](#9)
  - [3.2 Схема](#10)
- [4. Физическая схема базы данных](#11)
  - [4.1 Детали](#12)
  - [4.2 Схема](#13)
- [999. Используемые источники](#999)


## <a id="1"></a> 1. Тема и целевая аудитория
### <a id="2"></a> 1.1 Тема
**[Booking](https://booking.com/)** &ndash; система интернет-бронирования отелей, основанная в 1996 году в Амстердаме (Нидерланды). Booking.com предлагает миллионам гостей потрясающие варианты досуга, транспортные услуги и невероятное жилье, начиная от домов и заканчивая отелями и не только. Будучи крупнейшей в мире туристической платформой как для известных брендов, так и для предпринимателей разного уровня, Booking.com помогает владельцам объектов размещения по всему миру привлекать гостей и расширять бизнес.

### <a id="3"></a> 1.2 MVP
- Профили пользователей-арендодателей, предоставляющих жилье
- Профили пользователей-арендаторов, арендующих жилье
- Просмотр набора объявлений
- Просмотр статусов объявлений
- Возможность осуществлять фильтр по объявлениям (поиск)
- Возможность брони

### <a id="4"></a> 1.3 Целевая аудитория
![Image alt](https://github.com/VarenytsiaMykhailo/tp-highload-project-design/blob/main/images/users_count.jpg)
![Image alt](https://github.com/VarenytsiaMykhailo/tp-highload-project-design/blob/main/images/users_country_stat.jpg)

В сети не найдено информации сколько пользователей существует в booking.com и сколько объявлений в среднем хранится в базе данных.

Ясно, что 500 миллионов посещений в месяц - это в большинстве не авторизованные пользователи. Предположим, что в booking.com зарегистрировано 500 миллионов пользователей-арендаторов (примерно 6.25% населения земли) и 10 миллионов пользователей-арендодателей.

Таким образом, будем считать, что: 
* В системе зарегистрировано до 10 000 000 пользователей-арендодателей
* В системе зарегистрировано до 500 000 000 пользователей-арендаторов
* В каждый момент времени в сервисе размещается до 100 000 000 объявлений (здесь предположим, что каждый арендодатель, в среднем, за все время разместил не более 10 объявлений, а если аккаунтов в системе 10 000 000, то умножаем на 10)
* В среднем объявление может содержать до 10 картинок
* Количество категорий для объявлений - 20
* Среднее количество запией о попытках бронирования в системе - до 100

## <a id="5"></a> 2. Расчет нагрузки
### <a id="6"></a> 2.1 Продуктовые метрики
Рассмотрим продуктовые метрики, которые могут влиять на производительность сервиса, на основании которых будут составляться технические оценки. 
Далее, средние значения продуктовых метрик будем округлять в большую сторону.

- В месяц, в среднем, booking посещает 600 миллионов пользователей (посещений), а в день - 20 миллионов пользователей (посещений)
- Средняя продолжительность посещения 9 минут
- В среднем пользователи посещают по 9 страниц на сервисе за посещение

Рассчитаем пиковую нагрузку пользователей в час. 

Предположим, что из 20 миллионов посещений в сутки, 80% из них осуществляется в период с 8:00 до 00:00, поскольку в нашем случае система рассматривается для Европы и Средней Азии. Данный временной диапазон является "горячим" и на него отводится 16 миллионов посещений. Будем считать, что в данном временном диапазоне почасовое посещение равномерное (хотя и с высокой натяжкой), таким образом, 16 миллионов нужно поделить на 16 - количество часов в "горячем" временном диапазоне. 
- Таким образом, в час сервис посещает, в среднем, 1 миллион пользователей. 
- В секунду - 300 пользователей (с округлением вверх).

Далее, нужно определить типовые действия пользователей и какие бизнес сущности они наблюдать в процессе посещения сервиса, на основании которых будет производиться расчет нагрузки с технической стороны.


В среднем, пользователь посещает 8 страниц за сеанс. В 95% случаев пользователь не осуществляет бронь и не общается с арендодателями, а всего лишь рассматривает объявления. Таким образом, из действий, согласно нашему MVP, пользователи в основном просматривают объявления и пользуются фильтром, чуть реже посещают профили арендодателей. Всеми остальными действиями пользователя для расчета нагрузки можно пренебречь.

Далее, нужно решить, 8 посещенных страниц - это страницы, между которыми пользователь перемещается в браузере или это количество страниц, которые фактически отдал сервер? 

Если первое, то наверняка у пользователя на стороне клиента в загруженном JS действует механизм кэширования и при повторном переходе на страницу, она не загружается с сервера или же используются другие механизмы оптимизации. Таким образом, фактических отдач страниц с сервера может быть не 8, а меньше. Для простоты будем считать, что 8 посещаемых страниц - это фактически полноценные походы в бэкенд сервиса и полноценная отдача данных без механизмов кэширования (рассматриваем худший для сервиса сценарий).

Положим, что из 8 посещенных страниц:
- 1 страница - стартовая
- 2 страницы - страницы с набором объявлений
- 4 страницы - переходы к конкретным объявлениям
- 1 страница - переход в профиль арендодателя

### <a id="7"></a> 2.2 Расчет количества запросов в секунду (RPS)

Открыв стартовую страницу booking.com или выполнив какой-либо поиск, во вкладке "инструменты разработчика" видно, что на каждую страницу осуществляется, в среднем, 30 запросов следующего вида:


![image](https://user-images.githubusercontent.com/52725148/225346505-f03334f5-8fdc-4708-aa64-4a40110a7474.png)

Таким образом, в самом простом случае, пользователь, в среднем, открывает 8 страниц и осуществляет 250 запросов (с округлением вверх) за сеанс, а на 1 страницу приходится 30 запросов.

Нужно учесть, что дополнительно внутри инфраструктуры сервиса осуществляется гораздо больше запросов во внутренние микросервисы и хранилища данных, но сейчас нам это не нужно учитывать, особенно, пока не сформирована логическая и физическая схема хранимых данных.

Итого, имея продуктовую метрику посещений "300 посещений в секунду" и техническую "30 запросов на страницу", мы получаем 9000 запросов в секунду (RPS) в публичный сервис (не считая внутренни системы инфраструктуры).


## <a id="8"></a> 3. Логическая схема базы данных
### <a id="9"></a> 3.1 Детали
**PK** - Primary Key

**AK** - Alternative Key

**FK** - Foreign Key


В проекте было решено использовать два типа аккаунтов: для арендодателей и для арендаторов. Ниже, на логической схеме, совпало так, что поля таблиц для арендодателей и арендаторов совпадают и кажется, что их можно объеденить в одну. Но очевидно, что в реальном проекте, поля у сущностей "арендодатель" и "арендатор" могут существенно отличаться (да и с точки зрения реального мира - это разные сущности). Например, к арендодателю могут быть подвязаны дополнительная информация, юридические данные и другое. Нужно оставить возможность для добавления новых полей для данных таблиц и в будущем поля могут различаться.

Поэтому было принято решение разделить эти два аккаунта (такой подход часто используется в сервисах типа "продавец-покупатель").

Поскольку для реализации основных пунктов MVP требуется аккаунт арендаторов, то в MVP добавился пункт:
- Профили пользователей-арендаторов, арендующих жилье

Соответствие пунктов MVP с сущностями логической схемы базы данных:
- Профили пользователей-арендодателей, предоставляющих жилье: сущность **LandlordProfile**
- Профили пользователей-арендаторов, арендующих жилье: сущность **Renter**
- Просмотр набора объявлений: сущности **Ad** и **Category**
- Просмотр статусов объявлений: сущность **Status**
- Возможность осуществлять фильтр по объявлениям (поиск): все сущности
- Возможность брони: сущности **Status** и **Renter**

На первый взгляд может показаться, что сущность **Status** реализует связь "многие ко многим", а такие сущности, как правило, выявляются уже в рамках физической схемы и не должны фигурировать в логической схеме, за исключением шаблона проектирования "сопряжение". В данном случае, отношение **Status** реализует данный паттерн проектирования, поэтому его описание в виде "многие ко многим" уместно.

### <a id="10"></a> 3.2 Схема
Ниже приведена логическая схема проекта, выполненная в сервисе draw.io:


<img width="1204" alt="image" src="https://user-images.githubusercontent.com/52725148/226981986-be2c2f88-4a13-448a-8b60-5af8d7691e2f.png">


## <a id="11"></a> 4. Физическая схема базы данных
### <a id="12"></a> 4.1 Детали
Физическая схема составлялась ориентировочно для СУБД PostgreSQL.

В сущности Ad логической схемы присутствует поле images, подразумевающее хранение изображений для объявления. В рамках физической схемы такая функциональность реализуется путем выделения фотографий в отдельную таблицу - images.

Дополнительный контроль состояния, контроль логики "бронирования" будет реализовываться с помощью триггеров.


### <a id="13"></a> 4.2 Схема
Ниже приведена физическая схема проекта, выполненная в сервисе draw.io:


<img width="1490" alt="image" src="https://user-images.githubusercontent.com/52725148/226982457-cd3206fd-5be0-411f-be1d-5b63231bed59.png">


### <a id="14"></a> 4.3 Расчет объема на хранение данных
На основе составленной схемы и выбора типов хранимых данных можно рассчитать какой объем будут занимать хранимые данные. Будем рассчитывать средний объем записи для каждой таблицы.

Считаем, что 1 хранимый символ занимает 2 байта памяти.

Предположим, что средний объем байт для хранения полей не превышает:
* email (из таблиц landlords_profiles и renters): 30 символов = 40 байт
* phone_number (из таблиц landlords_profiles и renters): 12 символов = 24 байт
* password (из таблиц landlords_profiles и renters): 20 символов = 40 байт
* username (из таблиц landlords_profiles и renters): 20 символов = 40 байт
* avatar_img_path (путь к изображению из таблиц landlords_profiles и renters): 2048 символов = 4096 байт
* title (заголовок объявления из таблицы ads): 100 символов = 200 байт
* description (описание объявления из таблицы ads): 500 символов = 1000 байт
* content (контент объявления из таблицы ads): 5000 символов = 10000 байт
* img_path (путь к изображению из таблицы images): 2048 символов = 4096 байт
* category_name (из таблицы categories): 100 символов = 200 байт
* description (из таблицы categories): 500 символов = 1000 байт

Объединяя полученные подсчеты с полями с типами данных фиксированного размера и сформируя итоговые расчеты по таблицам, получим:

Одна запись в таблице в среднем будет занимать:
* landlords_profiles: 8 + 40 + 24 + 40 + 40 + 2 + 4 + 4096 + 8 = 4262 байт
* renters: 8 + 40 + 24 + 40 + 40 + 2 + 4 + 4096 + 8 = 4254 байт = 4262 байт
* ads: 8 + 200 + 1000 + 4 + 10000 + 4 + 4 + 4 = 11224 байт
* images: 8 + 4096 + 4 = 4108 байт
* categories: 8 + 200 + 1000 = 1208 байт
* statuses: 8 + 1 + 8 + 8 + 8 + 4 + 4 = 41 байт

Далее, на основе продуктовых метрик приблизительно рассчитаем, сколько записей содержится в каждой таблице:
* landlords_profiles: 10 000 000 (на основе продуктовых метрик)
* renters: 500 000 000 (на основе продуктовых метрик)
* ads: 100 000 000 (на основе продуктовых метрик)
* images: 1 000 000 000 (на основе продуктовых метрик)
* categories: 20 (на основе продуктовых метрик)
* statuses: 10 000 000 000 (на основе продуктовых метрик)

На основании полученных данных можно приблизительно рассчитать общий объем хранимых данных на каждую таблицу (с округлением до круглого числа в гигабайтах вверх):
* landlords_profiles: 42 620 000 000 байт = 40 гигабайт
* renters: 2 131 000 000 000 байт = 2000 гигабайт
* ads: 1 122 400 000 000 байт = 1050 гигабайт
* images: 4 108 000 000 000 байт = 3830 гигабайт
* categories: 24 160 байт = 0.000023 гигабайт
* statuses: 410 000 000 000 байт = 381 гигабайт


## <a id="14"></a> 5. Выбор технологий
### <a id="15"></a> 5.1 Языки программирования
Предполагаем, что основной состав разработчиков - программисты с опытом программирования на языках JVM и GoLang. Таким образом, будем подбирать технологии опираясь на программирование на JVM языках: 
- Kotlin
- Java
- GoLang

Преимущественно будем опираться на язык программирования Kotlin, поскольку он является мощным аналогом Java, с современным синтаксисом, расширенными возможностями (по сравнению с Java), высокой безопасностью, поддерживающим объектно-ориентированный, функциональный, функционально-декларативный стили программирования, обладающий высокой производительностью сопоставимой с Java и GoLang. Кроме того, Kotlin может сосуществовать с Java в одном проекте и использовать код, написанный на Java.

В местах, где возникают проблемы с использованием Kotlin - допускается использование Java или GoLang. Будем стремиться к тому, чтобы каждый сервис был реализован полноценно либо на Kotlin, либо на Kotlin + Java, либо на GoLang (стремимся не использовать, но для некоторых микросервисов допускается).


### <a id="16"></a> 5.2 Client: frontend-web-приложение, шлюз к ИС в виде Nginx
В качестве основного средства взаимодействия пользователей с сервисом будет использоваться web-сайт, будем называть его frontend.

Frontend будем реализовывать в виде SPA (Single Page Application) приложения, с технологией client-side rendering. 

Основная идея: при обращении пользователя к web-странице сайта, в качестве ответа на запрос к нему в браузер загружаются Java-Script файлы, которые будут рендерить в его браузере все страницы, реагировать на пользовательские события (клики по UI-компонентам), осуществлять запросы к ИС (информационной системе) для получения и отправки данных, событий и т.д.

Взаимодействие JS-скриптов с ИС должно осуществляться через шлюз. В качестве такого шлюза выберем Nginx.

Nginx - веб-сервер и почтовый прокси-сервер, позволяющий адресовать запросы к backend сервисам ИС, присылать клиенту необходимые JS файлы. Nginx позиционируется производителем как простой, быстрый и надёжный сервер, не перегруженный функциями. 

Данный Nginx сервер будет также считаться frontend-сервером, с которого будут в виде JS-файлов отправляться и обновляться страницы.

При построении детальной архитектуры в будущем, возможно использование нескольких Nginx серверов, например, один для реализации frontend, другие - для походов к backend ИС. Возможно использование нескольких инстансов серверов в комбинации с балансировщиком нагрузки.

![image](https://user-images.githubusercontent.com/52725148/231576626-3f08f5dd-2c95-4667-85a8-517a75fb4648.png)

![image](https://user-images.githubusercontent.com/52725148/231577039-50f6b665-a81e-4ff3-80bf-6b848936411b.png)


### <a id="17"></a> 5.3 Client: мобильные android и IOS приложение
Если позволяют ресурсы, в качестве дополнительного средства взаимодействия с сервисом возможна реализация мобильных android и IOS приложений.
Реализуемые приложения для походов к backend ИС могут также обращаться к шлюзам на основе Nginx. Для мобильных приложений могут требоваться свои инстансы шлюзов. 

Допускается архитектура переиспользования frontend web-приложения используя технологию WebView - когда в мобильном приложении работает встроенный браузер и пользователь получает frontend страницы в виде JS, адаптированные под мобильную верстку.

### <a id="18"></a> 5.4 Балансировщики нагрузки
На рынке существует широкий спектр продуктов, реализующих логику балансировки нагрузки и перераспределения запросов на различные сервисы системы. Для разных сервисов могут потребоваться разные решения. Выбор конкретных решений будет произведен позже. Важно использовать балансировщик нагрузки, например, для обращения к инстансам Nginx и внутренним backend-сервисам, особенно там, где это действительно необходимо. Также стоит учесть, что Nginx тоже предоставляет средства балансировки и возможно их использование. А для взаимодействия с PostgreSQL предлагается использовать PgBouncer.

![image](https://user-images.githubusercontent.com/52725148/231581181-846e2ce2-e0dc-40a0-a193-52bacb0503b5.png)


### <a id="18"></a> 5.5 Сервисы хранения аналитической информации и информации о крэшах и ошибках, производимых в системе
Важно в проекте использовать решение, которое будет сохранять аналитическую информацию и информацию об ошибках, крэшах, которые будут возникать в процессе функционирования ИС.
В качестве одного из таких решений предлагается использовать, например, Firebase. Также данное решение позволяет накапливать другую аналитическую информацию.

Firebase - это облачная база данных, которая позволяет пользователям хранить и получать сохраненную информацию, а также имеет удобные средства и методы взаимодействия с ней. Данные сервисы будут запущены в экосистеме google, а настраивать придется немного - все уже реализовано за нас и мы получим доступ к накапливаемой информации через удобную админку. Основной плюс использования firebase для этих целей - быстрое и легкое внедрение технологии в проект, что позволит быстро запуститься на рынке, но данное решение является платным, поэтому в будущем, в процессе развития проекта возможен переход на другие бесплатные решения.

Также предлагается использовать Prometheus и Grafana для сбора статистики и отображения собранных данных в виде дашбордов.

![image](https://user-images.githubusercontent.com/52725148/231581555-a7be3a03-d430-4252-840f-fb6a782a56af.png)

![image](https://user-images.githubusercontent.com/52725148/231582499-57b8a3ef-c992-4e9b-b901-c2a81145067d.png)


### <a id="19"></a> 5.6 Репозитории, CI/CD
В качестве размещения исходных кодов проекта предлагается использовать GitLab. Если сервисов планируется очень много, то рекомендуется под каждый сервис использовать свой репозиторий. Если сервисов планируется мало, то для них можно организовать одно окружение.

Для CI/CD предлагается использовать средства, предоставляемые GitLab - GitLab CI/CD.

![image](https://user-images.githubusercontent.com/52725148/231583138-b6dafc0d-dd64-480c-a184-52b45e2c4a53.png)


### <a id="20"></a> 5.7 Механизмы реализации очереди сообщений
В web-приложениях очереди сообщений часто используются для отложенной обработки событий (например, обработку лайков) или в качестве временного буфера между другими сервисами, тем самым защищая их от всплесков нагрузки.

В проекте может потребоваться использование очереди сообщений. В качестве такого решения предлагается использовать Kafka.

Apache Kafka — распределённый программный брокер сообщений с открытым исходным кодом, разрабатываемый в рамках фонда Apache на языках Java и Scala. Цель проекта — создание горизонтально масштабируемой платформы для обработки потоковых данных в реальном времени с высокой пропускной способностью и низкой задержкой.

![image](https://user-images.githubusercontent.com/52725148/231583574-4f48da20-c64b-44af-b911-0f0716cb1b47.png)


Как правило, приложение пишет и читает из очереди с помощью нескольких инстансов продюсеров и консьюмеров. Это позволяет эффективно распределить нагрузку.

![image](https://user-images.githubusercontent.com/52725148/233114130-16c4644b-eea4-4363-853f-12d2f88746a2.png)


### <a id="21"></a> 5.8 Backend
Для backend сервисов предлагается использовать технологии на Java-Kotlin стеке, например, сервисы предлагается реализовывать на Spring Boot, Ktor или Akka (каждый сервис реализуется с использованием одной из перечисленных технологий, в зависимости от задачи сервиса), а взаимодействие между сервисами организовать по REST, gRPC или с помощью очереди сообщений там, где это необходимо.

Например, легкие микросервисы можно реализовывать с использованием Ktor, более функциональные и тяжелые микросервисы с использованием Spring Boot, а Akka можно применить для сервисов, которые предназначены для обработки большого количества сообщений из очереди сообщений, например, из Kafka. Также REST сервисы можно реализовывать с помощью Akka HTTP. Ktor можно использовать для реализации клиентов к другим сервисам.

Spring Boot - это популярный фреймворк для создания веб-приложений с использованием Java. Это часть фреймворка Spring, которая представляет собой набор инструментов и библиотек для создания приложений корпоративного уровня. Spring Boot упрощает создание автономных приложений производственного уровня, которые можно легко развернуть и запустить с минимальной конфигурацией.

Ktor — это асинхронная платформа для создания микросервисов, веб-приложений, клиентов, написанная на Kotlin, основу которой составляет Kotlin и корутины. С помощью Ktor можно создавать клиентские и серверные приложения, которые будут работать на разных платформах. Фреймворк отлично подходит для приложений, требующих связь по HTTP и/или сокетам. Это могут быть, например, HTTP-бэкенды и RESTful-системы, независимо от того, построены ли они по микросервисному принципу или нет.

Akka — это набор инструментов и среда выполнения с доступным исходным кодом, упрощающие создание параллельных и распределенных приложений на JVM. Akka поддерживает несколько моделей программирования для параллелизма, но делает упор на параллелизм на основе акторов. Akka была создана для решения проблем в высоконагруженных системах.


![image](https://user-images.githubusercontent.com/52725148/231610332-18776592-fd67-443e-9d37-745bebcc2701.png)


Ktor:

![image](https://user-images.githubusercontent.com/52725148/231610954-132c5e14-76fe-4e6c-a435-465d9fc7d4f3.png)


Akka:

![image](https://user-images.githubusercontent.com/52725148/231611251-6791a363-6ab6-4746-81f6-3a6cfec590b1.png)


### <a id="22"></a> 5.9 Базы данных
В качестве технологий для реализации баз данных предлагается использовать следующий стек, идеально подходящий для проектов на JVM: 
- PostgreSQL - для хранения данных в реляционной форме 
- Redis - для хранения данных в формате ключ-значение или в виде популярных структур данных (списки, хэш-таблицы, множества). Например, Redis предлагается использовать для хранения сессий клиентов
- Хранилища семейства S3 - для хранения изображений
- Cassandra - в случае, если потребуется нереляционная база данных
- Hadoop, Spark, Hive и смежные технологии - если возникнет потребность в хранении больших файлов и данных, например, для аналитики


### <a id="23"></a> 5.10 Механизмы кэширования
Для реализации механизмов кэширования предлагается использовать следующие технологии:
- Redis
- Memcached

Данные технологии имеют свои плюсы и минусы. Например, Redis более универсален и его можно использовать для кэширования сложных данных, а у Memcached лучше реализована многопоточность, но он не способен поддерживать сложные структуры данных.


### <a id="24"></a> 5.11 Сервера конфигураций
В качестве технологии, позволяющей конфигурировать и синхронизировать другие сервисы системы предлагается использовать ZooKeeper.

Apache ZooKeeper - это сервис для предоставления информации о конфигурации, именовании, синхронизации в распределенных системах.

ZooKepper будет следить о состоянии сервисов и предоставлять клиентам информацию о них, например, находить в системе нужные сервисы для решения определенных задач. ZooKeeper часто используют в паре с Kafka. 


![image](https://user-images.githubusercontent.com/52725148/231611475-732a622a-b3b8-4a08-8380-9580562aef76.png)

![image](https://user-images.githubusercontent.com/52725148/233137123-3d41da7a-8881-4d4d-a573-2cc53e13e60a.png)

![image](https://user-images.githubusercontent.com/52725148/233136662-4231e82c-7877-44df-b4ca-9ef169c80b67.png)


## <a id="999"></a> Используемые источники
[Booking web-site](https://www.booking.com/content/about.ru.html)
<br/>
[Booking traffic statistics](https://www.similarweb.com/ru/website/booking.com)
<br/>
[Методология расчета нагрузки, количества пользователей информационной системы — web-сайта или сервиса](https://habr.com/ru/post/187386)
<br/>
[Nginx](https://ru.wikipedia.org/wiki/Nginx)
<br/>
[Firebase](https://kolmogorov.pro/what-is-firebase-chto-takoe)
<br/>
[Kafka](https://ru.wikipedia.org/wiki/Apache_Kafka)
