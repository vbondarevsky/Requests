Публикация на [Инфостарт](https://infostart.ru/public/709325/)


[![Quality Gate Status](https://sonar.openbsl.ru/api/project_badges/measure?project=connector&metric=alert_status)](https://sonar.openbsl.ru/dashboard?id=connector)
[![Stars](https://img.shields.io/github/stars/vbondarevsky/Connector.svg?label=Github%20%E2%98%85&a)](https://github.com/vbondarevsky/Connector/stargazers)
[![Release](https://img.shields.io/github/tag/vbondarevsky/Connector.svg?label=Last%20release&a)](https://github.com/vbondarevsky/Connector/releases)

# Коннектор: удобный HTTP-клиент для 1С:Предприятие 8
В мире python очень популярна библиотека для работы с HTTP запросами - [Requests](http://docs.python-requests.org/en/master) (автор: Kenneth Reitz).
Библиотека берет на себя всю рутину работы с HTTP запросами. 
Буквально в одну строку можно получать данные, отправлять, не заботясь о необходимости конструирования URL, кодирования данных и т.п. 
В общем библиотека очень мощная и проста в использовании.

**Коннектор** - это "Requests" для мира 1С.

## Возможности
Основные возможности библиотеки:
- Передача параметров в строку запроса (в URL)
- Удобная работа с запросами и ответами в формате `JSON`
- Отправка данных формы (полей формы), `application/x-www-form-urlencoded`
- Отправка данных формы (полей формы и файлов), `multipart/form-data`
- Прозрачная поддержка ответов, закодированных `GZip`
- Сжатие тела запроса `GZip`
- `Basic`, `Digest` и `AWS4-HMAC-SHA256` аутентификация
- Автоматическое разрешение редиректов
- Установка и чтение Cookies
- Работа в рамках сессии с сохранением состояния (cookies, аутентификация и пр.)
- Переиспользование `HTTPСоединение` в рамках сессии
- Настраиваемые повторные попытки соединения/отправки запроса с экспоненциальной задержкой
- Работает в т.ч. и на мобильной платформе
- И многое другое

## Требования
- Платформа **8.3.10** и выше.
- Мобильная платформа (проверено только на **8.3.15**)

## Использование
Скопируйте общий модуль к себе в конфигурацию.

## Пример мощи библиотеки
*Чем же хороша библиотека? Давай уже покажи пример.*

Получим данные `JSON` с помощью `GET`-запроса:

Вот так это делается стандартными средствами 1С
```bsl
ЗащищенноеСоединение = Новый ЗащищенноеСоединениеOpenSSL(Неопределено, Новый СертификатыУдостоверяющихЦентровОС);
Соединение = Новый HTTPСоединение("api.github.com", 443,,,, 30, ЗащищенноеСоединение);	
Запрос = Новый HTTPЗапрос("/events");
Ответ = Соединение.Получить(Запрос);
Поток = Ответ.ПолучитьТелоКакПоток();
Кодировка = "utf-8"; // ну допустим мы знаем что там такая кодировка

Ридер = Новый ЧтениеJSON;
Ридер.ОткрытьПоток(Поток, Кодировка); // Кодировка в заголовке ответа
Результат = ПрочитатьJSON(Ридер);
Ридер.Закрыть();
```

А вот так с помощью **Коннектора**
```bsl
Результат = КоннекторHTTP.GetJson("https://api.github.com/events");
```

Все! В `Результат` будет десериализованный из `JSON` ответ сервера. 
При этом:
- Библиотека сама разбила URL на составляющие
- Установила защищенное соединение 
- Определила кодировку ответа из заголовков
- Десериализовала `JSON`
 
И это достаточно простой пример. Всю мощь библиотеки рассмотрим далее.

## Передача параметров в строку запроса (в URL)
Работать с параметрами запроса очень просто:
```bsl
ПараметрыЗапроса = Новый Структура;
ПараметрыЗапроса.Вставить("name", СтрРазделить("Иванов,Петров", ","));
ПараметрыЗапроса.Вставить("salary", Формат(100000, "ЧГ="));

Ответ = КоннекторHTTP.GetJson("https://httpbin.org/anything/params", ПараметрыЗапроса);	
```

Поддерживается передача нескольких значений для одного параметра, достаточно указать в качестве значения `Массив` (см. `name`).

Параметры можно задать:
- Явно в URL
- Передать в параметре `ПараметрыЗапроса`
- Скомбинировать оба варианта

Результат будет один и тот же:
- Коннектор подставит параметры в URL в виде пар ключ=значение
- Закодирует строку URL, используя `URLEncoding`
- Выполнит запрос

Итоговое значение URL можно получить из свойства ответа `URL`
```bsl
Ответ = КоннекторHTTP.Get("https://httpbin.org/anything/params", ПараметрыЗапроса);
```
`Ответ.URL` - https://httpbin.org/anything/params?name=%D0%98%D0%B2%D0%B0%D0%BD%D0%BE%D0%B2&name=%D0%9F%D0%B5%D1%82%D1%80%D0%BE%D0%B2&salary=100000

## Произвольные HTTP заголовки
В основных сценариях использования библиотеки заголовки формируются автоматически.
При необходимости произвольные заголовки можно задать через параметр `ДополнительныеПараметры`, свойство `Заголовки`.
```bsl
Заголовки = Новый Соответствие;
Заголовки.Вставить("X-My-Header", "Hello!!!");
Результат = КоннекторHTTP.GetJson("http://httpbin.org/headers", Неопределено, Новый Структура("Заголовки", Заголовки));
```

## Работа с JSON
Для облегчения работы с JSON есть методы: 
`GetJson`, `PostJson`, `PutJson`, `DeleteJson`.
Запросы отправляются в формате JSON, ответы - JSON прочитанный в `Соответствие`/`Структура`. 
```bsl
Результат = КоннекторHTTP.GetJson("http://httpbin.org/get");
Результат = КоннекторHTTP.PostJson("http://httpbin.org/post", Новый Структура("Название", "КоннекторHTTP"));
Результат = КоннекторHTTP.PutJson("http://httpbin.org/put", Новый Структура("Название", "КоннекторHTTP"));
Результат = КоннекторHTTP.DeleteJson("http://httpbin.org/delete", Новый Структура("Название", "КоннекторHTTP"));
```

Сериализация в JSON и десериализация из JSON настраиваются с помощью параметров в `ДополнительныеПараметры.ПараметрыПреобразованияJSON`.

## Отправка данных формы
Отправить данные формы очень просто.
Передаем данные (`Структура` или `Соответствие`) в метод `POST` и все. 
```bsl
Данные = Новый Структура;
Данные.Вставить("comments", "Постучать в дверь");
Данные.Вставить("custemail", "vasya@mail.ru");
Данные.Вставить("custname", "Вася");
Данные.Вставить("custtel", "112");
Данные.Вставить("delivery", "20:20");
Данные.Вставить("size", "medium");
Данные.Вставить("topping", СтрРазделить("bacon,mushroom", ","));

Ответ = КоннекторHTTP.Post("http://httpbin.org/post", Данные);
```

Данные будут закодированы, заголовку `Content-Type` автоматически будет установлено значение `application/x-www-form-urlencoded`.

## Отправка файла
Для отправки файла нужно сформировать описание файла и передать его в параметр `ДополнительныеПараметры.Файлы`.
```bsl
Файлы = Новый Структура;
Файлы.Вставить("Имя", "f1");
Файлы.Вставить("ИмяФайла", "file1.txt");
Файлы.Вставить("Данные", Base64Значение("0J/RgNC40LLQtdGCINCc0LjRgCE="));
Файлы.Вставить("Тип", "text/plain");

Результат = КоннекторHTTP.Post("https://httpbin.org/post", Неопределено, Новый Структура("Файлы", Файлы));
```

Файл будет закодирован в теле запроса, заголовку `Content-Type` автоматически установлено значение `multipart/form-data`.

## Отправка файлов и данных формы
Для отправки данных формы и файлов в одном запросе нужно сформировать описание файлов и данных формы и передать их в параметрах `ДополнительныеПараметры.Файлы`, `ДополнительныеПараметры.Данные`.
```bsl
Файлы = Новый Массив;
Файлы.Добавить(Новый Структура("Имя,Данные,ИмяФайла", "f1", Base64Значение("ZmlsZTE="), "file1.txt"));
Файлы.Добавить(Новый Структура("Имя,Данные,ИмяФайла", "f2", Base64Значение("ZmlsZTI="), "file2.txt"));

Данные = Новый Структура("field1,field2", "value1", "Значение2");

Результат = КоннекторHTTP.Post("https://httpbin.org/post", Неопределено, Новый Структура("Файлы,Данные", Файлы, Данные));
```

Файлы и данные формы будут закодированы в теле запроса, заголовку `Content-Type` автоматически установлено значение `multipart/form-data`.

## Отправка произвольных данных
Чтобы отправить произвольные данные (`Строка`, `ДвоичныеДанные`) их нужно передать в параметре `Данные`.
```bsl
XML = 
"<?xml version=""1.0"" encoding=""utf-8""?>
|<soap:Envelope xmlns:xsi=""http://www.w3.org/2001/XMLSchema-instance"" xmlns:xsd=""http://www.w3.org/2001/XMLSchema"" xmlns:soap=""http://schemas.xmlsoap.org/soap/envelope/"">
|  <soap:Body>
|    <GetCursOnDate xmlns=""http://web.cbr.ru/"">
|      <On_date>2019-07-05</On_date>
|    </GetCursOnDate>
|  </soap:Body>
|</soap:Envelope>";
    
Заголовки = Новый Соответствие;
Заголовки.Вставить("Content-Type", "text/xml; charset=utf-8");
Заголовки.Вставить("SOAPAction", "http://web.cbr.ru/GetCursOnDate");
Ответ = КоннекторHTTP.Post(
    "https://www.cbr.ru/DailyInfoWebServ/DailyInfo.asmx",
    XML, 
    Новый Структура("Заголовки", Заголовки));
```

## Содержимое ответа
Методы, которые не заканчиваются на Json, созвращают ответ в виде `Структура`:
- `ВремяВыполнения` - Число - время выполнения запроса в миллисекундах
- `Cookies` - cookies полученные с сервера
- `Заголовки` - HTTP заголовки ответа
- `ЭтоПостоянныйРедирект` - признак постоянного редиректа
- `ЭтоРедирект` - признак редиректа
- `Кодировка` - кодировка текста ответа
- `Тело` - тело ответа
- `КодСостояния` - код состояния ответа
- `URL` - итоговый URL, по которому был выполнен запрос

Получить данные из ответа в виде JSON, теста или двоичных данных можно с помощью соответствующих методов, описанных ниже.

### Чтение ответа как JSON
Получить данные из ответа в виде десериализованного JSON можно с помощью метода `КакJson`.
```bsl
Результат = КоннекторHTTP.КакJson(КоннекторHTTP.Get("http://httpbin.org/get"));
```

### Чтение ответа как Текст
Получить данные из ответа в виде текста можно с помощью метода `КакТекст`.
```bsl
Результат = КоннекторHTTP.КакТекст(КоннекторHTTP.Get("http://httpbin.org/encoding/utf8"));
```
При этом можно указать кодировку в соответствующем параметре. 
Если параметр не указан, то **Коннектор** возьмет значение кодировки из заголовков (если она там есть).
 
### Чтение ответа как ДвоичныеДанные
Метод `КакДвоичныеДанные` преобразует ответ в `ДвоичныеДанные`.
```bsl
Результат = КоннекторHTTP.КакДвоичныеДанные(КоннекторHTTP.Get("http://httpbin.org/image/png"));
```

### Чтение ответа XML как XDTO 
Метод `КакXDTO` преобразует ответ XML в `ОбъектXDTO`.
```bsl
Результат = КоннекторHTTP.КакXDTO(Ответ);
```

## GZip-кодирование тела запроса
**Коннектор** может автоматически сжимать тело запроса `GZip`.
Для этого нужно добавить заголовок `Content-Encoding` = `gzip`.

**Примечание**: сервер должен быть настроен соответствующим образом, чтобы распаковывать входящие запросы.

```bsl
Json = Новый Структура;
Json.Вставить("field", "value");
Json.Вставить("field2", "value2");
Заголовки = Новый Соответствие;
Заголовки.Вставить("Content-Encoding", "gzip");
Результат = КоннекторHTTP.PostJson("http://httpbin.org/anything", Json, Новый Структура("Заголовки", Заголовки));
```

## GZip-декодирование тела ответа
По умолчанию **Коннектор** просит сервер кодировать ответы в формате `GZip`.

**Примечание**: любое кодирование тела ответа можно отключить, задав заголовок
`Accept-Encoding` = `identity`.

Декодирование выполняется прозрачным образом в методах `GetJson`, `PostJson`, `PutJson`, `DeleteJson`, `КакJson`, `КакТекст`, `КакДвоичныеДанные`.
```bsl
Результат = КоннекторHTTP.GetJson("http://httpbin.org/gzip");
```

## Таймаут
Таймаут можно задать в параметре `ДополнительныеПараметры.Таймаут`.
```bsl
Ответ = КоннекторHTTP.Get("https://httpbin.org/delay/10", Неопределено, Новый Структура("Таймаут", 1));
```
Значение по умолчанию - 30 сек.

## Basic-аутентификация
Параметры Basic-аутентификации можно передать в параметре `ДополнительныеПараметры.Аутентификация`
```bsl
Аутентификация = Новый Структура("Пользователь, Пароль", "user", "pass");
Результат = КоннекторHTTP.GetJson(
    "https://httpbin.org/basic-auth/user/pass",
    Неопределено,
    Новый Структура("Аутентификация", Аутентификация));
```
или в `URL`
```bsl
Результат = КоннекторHTTP.GetJson("https://user:pass@httpbin.org/basic-auth/user/pass");
```

## Digest-аутентификация
Параметры Digest-аутентификации можно передать в параметре `ДополнительныеПараметры.Аутентификация`.
При этом `Тип` нужно установить в значение `Digest`.
```bsl
Аутентификация = Новый Структура("Пользователь, Пароль, Тип", "user", "pass", "Digest");
Результат = КоннекторHTTP.GetJson(
    "https://httpbin.org/digest-auth/auth/user/pass",
    Неопределено,
    Новый Структура("Аутентификация", Аутентификация));
```

## AWS4-HMAC-SHA256-аутентификация
Параметры AWS4-HMAC-SHA256-аутентификации можно передать в параметре `ДополнительныеПараметры.Аутентификация`.
При этом `Тип` нужно установить в значение `AWS4-HMAC-SHA256` и задать свойства: `ИдентификаторКлючаДоступа`, `СекретныйКлюч`, `Сервис`, `Регион`.

```bsl
Аутентификация = Новый Структура;
Аутентификация.Вставить("Тип", "AWS4-HMAC-SHA256");
Аутентификация.Вставить("ИдентификаторКлючаДоступа", "AKIAU00002SQ4MT");
Аутентификация.Вставить("СекретныйКлюч", "МойСекретныйКлюч");
Аутентификация.Вставить("Регион", "ru-central1");
Аутентификация.Вставить("Сервис", "s3");

Файл = Новый ДвоичныеДанные("my_file.txt");

Заголовки = Новый Соответствие;
Заголовки.Вставить("Content-Type", "text/plain");
Заголовки.Вставить("x-amz-meta-author", "Vladimir Bondarevskiy");
Заголовки.Вставить("Expect", "100-continue");

ДополнительныеПараметры = Новый Структура;
ДополнительныеПараметры.Вставить("Заголовки", Заголовки);
ДополнительныеПараметры.Вставить("Аутентификация", Аутентификация);
ДополнительныеПараметры.Вставить("Таймаут", 300);
Ответ = КоннекторHTTP.Put("https://test.storage.yandexcloud.net/my_file.txt", Файл, ДополнительныеПараметры);
```

## Доступ через прокси-сервер
Настройки прокси можно передать в параметре `ДополнительныеПараметры.Прокси`.
```bsl
Прокси = Новый ИнтернетПрокси;
Прокси.Установить("http", "192.168.1.51", 8192);
Результат = КоннекторHTTP.GetJson("http://httpbin.org/headers", Неопределено, Новый Структура("Прокси", Прокси));
```
Если в конфигурации используется `БСП`, то настройки прокси по умолчанию берутся из `БСП`.

## Поддерживаемые HTTP методы
Для `GET`, `OPTIONS`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE` есть соответствующие методы.
Для любого из HTTP-методов можно отправить запрос через вызов метода `ВызватьМетод`.

## Редиректы (Перенаправления)
**Коннектор** по умолчанию автоматически разрешает редиректы.
Например, попробуем получить результат поиска в Яндексе (http://ya.ru).
```bsl
Результат = КоннекторHTTP.Get("http://ya.ru/", Новый Структура("q", "удаление кеша метаданных инфостарт"));
```
Что по факту произойдет при выполнении этого запроса:
- **Коннектор** выполнит запрос к URL http://ya.ru/
- Сервер попросит выполнить запрос используя `https`, т.е. вернет код статуса `302` и значение заголовка `Location`=`https://ya.ru/?q=...`
- **Коннектор** выполнит перезапрос, используя схему `https`
- Cервер попросит выполнить запрос, используя другой URL, т.е. вернет код статуса `302` и значение заголовка `Location`=`https://yandex.ru/search/?text=...` 
- **Коннектор** выполнит перезапрос, используя URL `https://yandex.ru/search/?text=...`
- Cервер наконец-то вернет результат в виде `html`

Отключить автоматический редирект можно с помощью параметра `ДополнительныеПараметры.РазрешитьПеренаправление`.

## Проверка серверного сертификата SSL
Нужно ли проверять сертификат сервера и какие корневые сертификаты для этого использовать можно задать через параметр `ДополнительныеПараметры.ПроверятьSSL`.
```bsl
Результат = КоннекторHTTP.Get("https://my_super_secret_server.ru/", Новый Структура("ПроверятьSSL", Ложь));
```

## Клиентские сертификаты
Клиентский сертификат можно задать через параметр `ДополнительныеПараметры.КлиентскийСертификатSSL`.
```bsl
КлиентскийСертификатSSL = Новый СертификатКлиентаФайл("my_cert.p12", "123");
Результат = КоннекторHTTP.Get("https://my_super_secret_server.ru/", Новый Структура("КлиентскийСертификатSSL", КлиентскийСертификатSSL));
```

## Работа с Cookies
**Коннектор** извлекает cookies из заголовков `Set-Cookie` ответа сервера для дальнейшего использования. 
Полученные cookies можно посмотреть в свойстве ответа `Cookies`.

Передать произвольные cookies на сервер можно с помощью параметра `ДополнительныеПараметры.Cookies`.
```bsl
Cookies = Новый Массив;
Cookies.Добавить(Новый Структура("Наименование,Значение", "k1", Строка(Новый УникальныйИдентификатор)));
Cookies.Добавить(Новый Структура("Наименование,Значение", "k2", Строка(Новый УникальныйИдентификатор)));
Ответ = КоннекторHTTP.Get("http://httpbin.org/cookies", Неопределено, Новый Структура("Cookies", Cookies));
```

## Работа в рамках сессии
**Коннектор** позволяет работать с сервером в рамках сессии, т.е. сохраняет состояние между вызовами.
Для этого нужно:
- Создать объект `Сессия` методом `СоздатьСессию`
- При каждом вызове передавать созданный объект в параметр `Сессия`

Например, попробуем получить с сайта releases.1c.ru список обновлений.
```bsl
Сессия = КоннекторHTTP.СоздатьСессию();
Ответ = КоннекторHTTP.Get("https://releases.1c.ru/total", Неопределено, Неопределено, Сессия);

Данные = Новый Структура;
Данные.Вставить("execution", ИзвлечьExecution(Ответ));
Данные.Вставить("username", Логин);
Данные.Вставить("password", Пароль);
Данные.Вставить("_eventId", "submit");
Данные.Вставить("geolocation", "");
Данные.Вставить("submit", "Войти");
Данные.Вставить("rememberMe", "on");

Ответ = КоннекторHTTP.Post(Ответ.URL, Данные, Неопределено, Сессия);
```

Что при этом произойдет:
- **Коннектор** выполнит `GET` запрос к URL `https://releases.1c.ru/total`
- Сервер попросит выполнить запрос к URL `https://login.1c.ru/login?service=https%3A%2F%2Freleases.1c.ru%2Fpublic%2Fsecurity_check`
- **Коннектор** сохранит полученные cookies и выполнит `GET` запрос к URL `https://releases.1c.ru/total`
- Сервер вернет форму, в которой нужно авторизоваться
- Извлечем данные из формы и отправим их на сервер вместе с нашим логином и паролем
- **Коннектор** выполнит `POST` запрос и отправит данные формы и ранее полученные cookies
- Сервер проверит параметры формы и если все хорошо, то выдаст тикет и попросит выполнить запрос к URL `https://releases.1c.ru/total`
- **Коннектор** выполнит `GET` запрос к URL `https://releases.1c.ru/total` и передаст установленные ранее cookies
- Сервер вернет нужный нам результат в виде `html`

Далее используя `Сессия` можно выполнять запросы к серверу и скачивать обновления.

## Повторные попытки соединения/отправки запроса
**Коннектор** может автоматически выполнять повторные попытки соединения/отправки запроса с задержкой.
Это бывает полезно если:
- Соединение нестабильное
- Сервер перегружен и может "пятисотить"
- Сервер находится на обслуживании (перезагрузка, изменение конфигов, обновление и т.п.)
- Сервер ограничивает количество запросов от клиента

Включить повторы можно с помощью параметра `ДополнительныеПараметры.МаксимальноеКоличествоПовторов`, задав значение больше 0.

Параметр `ДополнительныеПараметры.МаксимальноеВремяПовторов` позволяет ограничить суммарное время (таймауты + задержки между попытками).
Значение по умолчанию: 10 мин.

Длительность задержки между попытками:
- Растет экспоненциально (1 сек, 2 сек, 4 сек, 8 сек, 16 сек, ...). Можно регулировать с помощью параметра 
`ДополнительныеПараметры.КоэффициентЭкспоненциальнойЗадержки`
- Для кодов состояний `413`, `429` или `503` в качестве задержки используется значение заголовка `Retry-After` (длительность в секундах или конкретная дата).

Параметр `ДополнительныеПараметры.ПовторятьДляКодовСостояний` позволяет задать коды состояний, при которых нужно выполнять повтор.
Если параметр не задан, повтор будет выполняться для всех кодов состояний >=`500`.

```bsl
ДополнительныеПараметры = Новый Структура;
ДополнительныеПараметры.Вставить("МаксимальноеКоличествоПовторов", 5);
ДополнительныеПараметры.Вставить("Заголовки", Заголовки);
    
URL = "http://127.0.0.1:5000/retry_after_date";
Ответ = КоннекторHTTP.Get(URL, Неопределено, ДополнительныеПараметры);
```
