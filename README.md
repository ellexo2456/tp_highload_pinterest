# Проектирование высоконагруженной социальнной сети фото-хостинга

Курсовая работа в рамках 3-го семестра программы по Веб-разработке ОЦ VK x МГТУ им. Н.Э. Баумана (ex. "Технопарк") по дисциплине "Проектирование высоконагруженных сервисов"

#### Автор - [Чинаев Алексей](https://park.vk.company/profile/a.chinaev/ "Страница на портале VK x МГТУ")
#### Задание - [Методические указания](https://github.com/init/highload/blob/main/homework_architecture.md)

#### Содержание:
1. [Тема, функционал и аудитория](#1)
2. [Расчёт нагрузки](#2)
2. [Глобальная балансировка нагрузки](#3)


## Часть 1. Тема, функционал и аудитория <a name="1"></a>

### Тема курсовой работы - **"Проектирование высоконагруженной социальной сети фото-хостинга"**
В качестве примера и аналога выбрана социальная сеть [Pinterest](https://ru.pinterest.com/)

### Ключевой функционал сервиса
- Регистрация и авторизация пользователей
- Создание пинов
- Создание коллекций (досок) и сохранение в них пинов
- Бесконечная лента
- Комментарии к пинам
- Лайки к пинам

> пин - единица ленты, созданная пользователем и представляющая собой фото или видео

### Ключевые продуктовые решения
- Сочетание фото и видео в одной ленте для увеличения разнообразия контента
- Поиск по ключевым словам
- Составление ленты и тематических подборок на основе анализа создания досок, добавления и просмотра пинов, подписок и поисковых запросов пользователя
- Рекламные пины

### Целевая аудитория
- 498 млн активных пользователей по всему миру [[1]](https://business.pinterest.com/audience/)
- 1,5 миллиарда пинов сохраняется каждую неделю [[2]](https://newsroom.pinterest.com/company/)


## Часть 2. Расчёт нагрузки <a name="2"></a>

### Продуктовые метрики

|Метрика|Значение|Источник|
| ------------- | --- |-------------|
|MAU (чел.)|498 млн|[[3]](https://business.pinterest.com/audience/)|
|DAU (чел.)|80,5 млн|[[4]](https://www.businessdit.com/social-media-statistics/)|
|Прирост аудитории за 2023 год|49 млн|[[5]](https://www.statista.com/statistics/463353/pinterest-global-mau/)|
|Авторизаций одного и того же человека в год|6|Предположительно|
|Создаваемых одним человеком пинов в день|4|Предположительно|
|Сохраняемых в неделю пинов|1,5 млрд|[[6]](https://newsroom.pinterest.com/company/)|
|Количество созданных досок в месяц одним и тем же пользователем|1|Предположительно|
|Просмотров пинов в день одним и тем же пользователем|50|Предположительно|
|Комментариев в день от одного и того же пользователя|50|Предположительно|
|Комментариев в день от одного и того же пользователя|5|Предположительно|
|Реакций в день от одного и того же пользователя|10|Предположительно|
|Периодичность просматривания пинов в ленте|Каждый седьмой|Предположительно|


#### Средний размер харанилища пользователя по типам:

| Хранимые данные   | Оценочный размер на пользователя         |
|-------------------|------------------------------------------|
| Аватары              | 1 Мб                            |
| Данные профиля              | 1 Кб                     |

**Таким образом всего:** `1 Мб 1Кб` на пользователя при регистрации и вводе персональных данных.

#### Среднее количество действий пользователя по типам в день

|Действие пользователя |Количество / день|
| ------------- |-------------|
|Регистрация|134,25 тыс|
|Авторизация|1,341 млн|
|Создание пинов|322 млн|
|Сохранение пинов|214,3 млн|
|Создание досок|2,87 млн|
|Просмотр конкретных пинов|4,025 млрд|
|Комментирование| 402,5 млн|
|Реакции|805 млн|
|Просмотров пинов в ленте|28,175 млрд|

- За 2023 год аудитория Pinterest возросла на 49 млн [[7]](https://www.statista.com/statistics/463353/pinterest-global-mau/), т.е. в день в среднем регистрировались 134,25 тыс раз.  
- Пусть авторизацию в среднем проходят 6 раза в год => 1,341 млн раз в день.
- Ввиду отсутвия информации о среднем кол-ве создаваемых пинов возьмём среднее от рекомендуемого количества (5-10 [[8]](https://community.pinterest.biz/t5/connect-and-share-with-other-creators/how-many-pins-per-day/td-p/30848)),
  т.е. 7 пинов в день. Однако обычный пользователь очень редко выгружает пины, этим занимаются юр лица, поэтому будем считать, что одним пользователем выгружается 4 пинов/день.
  Тогда всего в день выгружается 322 млн пинов
- В неделю сохраняется 1,5 млрд пинов [[9]](https://newsroom.pinterest.com/company/) => 214,3 млн/день
- Пусть пользователь создаёт 1 доску в месяц => всего пользователи создают 2,87 млн/день
- Пусть пользователь просматривает 50 пинов в день => всего 4,025 млрд/день
- Пользователь оставляет 5 комментариев в день => всего 402,5 млн/день
- И 10 реакций в день => всего 805 млн/день
- Если пользователь просматривает каждый седьмой пин, то всего в ленте у него будет 350 пинов в день => всего 28,175 млрд

### Технические метрики

#### Размер хранения в разбивке по типам данных:
|Хранилище |Стартовый размер (Пб)|Увеличение (Пб/год)|
| ------------- |-------------|-------------|
|Пины |271,39 (Фото - 216,67; Видео - 54,72)|271,39 (Фото - 216,67; Видео - 54,72)|
|Данные пользователя |0,033|0,022|
|Комментарии |0,039|0,039|

- Учитывая скорость создания пинов 322 млн/день из пункта выше, в год создаётся 117,5 млрд пинов.  
Макс размер пина 20 мб [[10]](https://ru.pinterest.com/pin-creation-tool/), средний 2 Мб (т.к. обычно загружаются небольшие изображения).  
Для видео 200 мб [[11]](https://ru.pinterest.com/pin-creation-tool/), средний 50 Мб (т.к. обычно загружаются небольшие видео). .  
99% пинов - изображения [[12]](https://buffer.com/resources/pinterest-marketing-study/).  
Тогда:
  Фото:
  
    ```azure
    117,5 * 10^9 * 2 Мб * 0,99 ≈ 216,67 Пб
    ```
  Видео:
  
    ```azure
    117,5 * 10^9 * 0,01 * 50 Мб ≈ 54,72 Пб
    ```
- Из пункта выше: необходимо `1 Мб 1Кб` на пользователя.  
Средний прирост пользователей за год 350,75 млн [[13]](https://www.bankmycell.com/blog/number-of-pinterest-users/).  
Тогда:

```azure
350,75 * 10^6 * 1025 Кб ≈ 0,033 Пб
```
Пусть прирост будет составлять 2/3 от начального значаения, т.к. многие пользователи не устанавливают аватарки.

- Максимальная длинна комментария 500 символов Unicode. Пусть средняя длина будет 150, т.к. в основном пишутся короткие комментарии.
Тогда 1 комментарий займёт 300 байт. Учитывая скорость комментирования из пункта выше - 402,5 млн/день. За год будет 147 млрд.
Тогда:

```azure
147 * 10^9 * 300 б ≈ 0,039 Пб
```
### Сетевой трафик и rps

#### RPS по типам запросов:
| Тип запроса | RPD   | RPS |Пиковый RPS|
|-------------|-------------------------|-------------------------|-------------|
|Регистрация|134,25 тыс|1,55|2,79|
|Авторизация|1,341 млн|15,52|27,936|
|Создание пинов|322 млн|3727|6708,65|
|Сохранение пинов|214,3 млн|2480,3|4464,54|
|Создание досок|2,87 млн|33,22|59,796|
|Просмотр пинов конкретных пинов|4,025 млрд|46585,65|83854,17|
|Комментирование|402,5 млн|4685,6|8434,08|
|Реакции|805 млн|9317,13|16 770,834|
|Просмотров пинов в ленте|28,175 млрд|326100|586980|

#### Сетевой трафик

**Потребление по типам трафика:**
| Тип трафика        | Пиковое потребление, Тбит/с       | Суммарный суточный трафик, Пб/сутки  |
|--------------------|-----------------------------------|----------------------------------------|
| Cтатические файлы  | `13,81`                            | `80,946`                                |
| API                | `0,013`                            | `0,0743`                                |

- Страница ленты и пина
  - html, js, css - 2 Мб
  - Если пользователь просматривает каждый 7й пин,
    то, учитывая, что пин весит 2 Мб или 50 Мб (фото, видео) и 99% процентов пинов - фото, а также, что пользователь просматривает 50 пинов в день:
  
    Для страницы ленты:  
      Фото:  
      ```azure
      80,5 * 10^6 * (350 * 2 Мб * 0,99 + 3 * 2 Мб) ≈ 52,41 Пб
      ```  
      Видео:  
      ```azure
      80,5 * 10^6 * (350 * 0,01 * 50 Мб + 3 * 2 Мб) ≈ 13,57 Пб
      ```  
    Для страницы пина:  
      Фото:  
      ```azure
      80,5 * 10^6 * 50 * 2 Мб * 0,99 * 2 Мб ≈ 14,84 Пб
      ```  
      Видео:  
      ```azure
      80,5 * 10^6 * 0,01 * 50 Мб * 2 Мб ≈ 0,8 Пб
      ```
- Страницы регистарции, авторизации, создания пина, создания доски
  - html, js, css - 1,5 Мб
    Тогда итоговое суточное потребление:
    
    ```azure
    1,5 Мб * (134,25*10^3 + 1,341*10^5 + 322*10^5 + 2,87*10^5) ≈ 0,046 Пб
    ```
- Создание пинов

  Фото:
    ```azure
    2 Мб * 0,99 * 322 * 10^5 ≈ 0,059 Пб
    ```
  Видео:
    ```azure
    0,01 * 50 Мб * 322 * 10^5 ≈ 0,015 Пб
    ```
- Комментирование
    ```azure
    300 б * 402,5 * 10^5 ≈ 0,000011 ПБ
    ```
- Регистрация
     ```azure
    2 Мб * 134,25 * 10^3 ≈ 0,00026 ПБ
    ```
     
- Допустим, что пиковое дневное потребление в 1.8 раза больше среднего [29] (слайд 14). Получим 145,7028 Пб/день и 0,13374 Пб/день => 13,81 Тбит/сек и 0,013 Тбит/сек


## Часть 3. Глобальная балансировка нагрузки <a name="3"></a>
### Расположение ДЦ
Основная аудитория pinterest находится в Северной и Южной Америке и Азии (по большей части в Японии)[[30]](https://www.demandsage.com/pinterest-statistics/)[[31]](https://www.statista.com/statistics/328106/pinterest-penetration-markets/).  
Расположим ДЦ в соответствии с географичиским положением.  

Тогда:  
- Расположим ДЦ в Нью-Йорке и Сан-Франциско (крупнейшие центры трафика с восточного и западного побережья[[32]](https://global-internet-map-2022.telegeography.com/))
- Сан-Паулу (крупнейший город и центр трафика Браизлии)
- Франкфурт (одна из крупнейших развязок в Европе и мире)
- Токио (крупнейший узел Японии)

### Глобальная балансировка
Для глобальной балансировки запросов и нагрузки будем использовать:

- Для определения региона - GeoDNS, т.к. ДЦ сильно разнесены по миру, что позволит достаточно точно определять в ЦЦ какого региона стоит роутить запрос.
- Для роутинга запросов в пределах региона будем использовать BGP Anycast
