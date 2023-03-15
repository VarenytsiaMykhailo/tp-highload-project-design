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
- [999. Используемые источники](#999)


## <a id="1"></a> 1. Тема и целевая аудитория
### <a id="2"></a> 1.1 Тема
**[Booking](https://booking.com/)** &ndash; система интернет-бронирования отелей, основанная в 1996 году в Амстердаме (Нидерланды). Booking.com предлагает миллионам гостей потрясающие варианты досуга, транспортные услуги и невероятное жилье, начиная от домов и заканчивая отелями и не только. Будучи крупнейшей в мире туристической платформой как для известных брендов, так и для предпринимателей разного уровня, Booking.com помогает владельцам объектов размещения по всему миру привлекать гостей и расширять бизнес.

### <a id="3"></a> 1.2 MVP
- Профили пользователей-арендодателей, предоставляющих жилье
- Просмотр набора объявлений
- Просмотр статусов объявлений
- Возможность осуществлять фильтр по объявлениям (поиск)
- Возможность брони

### <a id="4"></a> 1.3 Целевая аудитория
![Image alt](https://github.com/VarenytsiaMykhailo/tp-highload-project-design/blob/main/images/users_count.jpg)
![Image alt](https://github.com/VarenytsiaMykhailo/tp-highload-project-design/blob/main/images/users_country_stat.jpg)

В данном проекте будем стремиться к нагрузке 1 миллионов пользователей в день, ориентируясь на рынок Европы и Средней Азии.

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

Таким образом, в самом простом случае, пользователь, в среднем, открывает 8 страниц и осуществляет 250 запросов (с округлением вверх) за сеанс. 

Нужно учесть, что дополнительно внутри инфраструктуры сервиса осуществляется гораздо больше запросов во внутренние микросервисы и хранилища данных, но сейчас нам это не нужно учитывать, особенно, пока не сформирована логическая схема хранимых данных.

Итого, имея продуктовую метрику посещений "300 посещений в секунду" и техническую "250 запросов на посещение", мы получаем 75000 запросов в секунду (RPS) в публичный сервис (не считая внутренни системы инфраструктуры).



## <a id="999"></a> Используемые источники
[Booking web-site](https://www.booking.com/content/about.ru.html)
<br/>
[Booking traffic statistics](https://www.similarweb.com/ru/website/booking.com)
<br/>
[Методология расчета нагрузки, количества пользователей информационной системы — web-сайта или сервиса](https://habr.com/ru/post/187386/)
