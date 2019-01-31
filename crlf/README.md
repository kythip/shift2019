# CRLF injection

## Описание
CRLF injection представляет собой тип атаки, использующий давно известную уязвимость. Данная уязвимость представляет собой возможность добавления злоумышленниками через пользовательский ввод новых заголовков в ответ HTTP-сервера с использованием символа переноса строки `%0d%0a`. Ведь сервер отделяет заголовки друг от друга, используя именно CRLF (Carriage Return/Line Feed)

Самый простой способ применения данной атаки - внедрение дополнительных заголовков в ответ от сервера.
Атака предназначена для обмана браузера клиента с использованием добавленных или измененных данных с целью проведения дальнейших махинаций.

## Условия
 - ОС: любая
 - язык: любой
 - компоненты: веб-сервер
 - настройки: любые

## Детектирование
Проверить наличие данной уязвимости довольно просто: добавлять в каждом пользовательском вводе символ переноса строки `%0d%0a` и, к примеру, заголовок `Set-Cookie`. Далее можем просмотреть ответ HTTP-сервера и если добавился cookie, то это свидетельствует о наличии уязвимости.
Пример: такой пользовательский ввод - `%0d%0aSet-Cookie:test=hacked`
Результат:
 - Появился новый заголовок в ответе сервера `Set-Cookie:test=hacked`. Вывод:  CRLF-уязвимость есть.
 - Новый заголовок не появиля. Вывод: CRLF-уязвимости нет

## Эксплуатация
Чтобы эксплуатировать CRLF-инъекцию необходимо в пользовательский ввод, который отражается в ответе, ввести запрашиваемую информацию вместе с символом переноса строки и последующим заголовком, который необходимо внедрить в ответ HTTP-сервера. Например, `Set-Cookie`.

### Добавление cookie
Для выполнения данной операции введем в заголовке следующее:
`%0D%0ASet-Cookie:crlf_team=tanya_masha_denis_edik`

![Запрос и ответ с добавление cookie.png](https://github.com/karpuna3/shift2019/blob/master/crlf/img/Запрос%20и%20ответ%20с%20добавление%20cookie.png)

Рисунок 1. Запрос и ответ с добавлением cookie

![Результат CRLF-инъекции с добавлением cookie.png](https://github.com/karpuna3/shift2019/blob/master/crlf/img/Результат%20CRLF-инъекции%20с%20добавлением%20cookie.png)

Рисунок 2. Результат CRLF-инъекции с добавлением cookie

### Получение XSS

Для достижения данной цели делаем все то же самое, что и в пункте 1 с той оговоркой, что необходимо после символа переноса строки использовать иные заголовки
`%0d%0aContent-Length%3A35%0d%0aX-XSS-Protection%3A0%0d%0a%0d%0a%3Csvg%20onload%3Dalert%28"XSS"%29%3E%0d%0a0%0d%0a%2F%2f%2e%2e`

Здесь используются следующие заголовки:
 - `Content-Length` несет в себе информацию о размере тела сообщения в байтах
 - `X-XSS-Protection` - фильтр, предотвращающий XSS-атаку, при значении 0 отключен
 
 Также реализована функция на языке JavaScript `alert`, отвечающая за вывод сообщения в отдельном окне и прекращающая обработку, пока окно не будет закрыто

![Реализации CRLF-инъекции для XSS.png](https://github.com/karpuna3/shift2019/blob/master/crlf/img/Реализации%20CRLF-инъекции%20для%20XSS.png)

Рисунок 3. Реализации CRLF-инъекции для XSS

![](https://github.com/karpuna3/shift2019/blob/master/crlf/img/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA.PNG?raw=true)

Рисунок 4. Результат получения XSS
### Замена информации полученной пользователем
 
 ![Информация, которую получает пользователь до подмены.png](https://github.com/karpuna3/shift2019/blob/master/crlf/img/Информация%2C%20которую%20получает%20пользователь%20до%20подмены.png)
 
 Рисунок 5. Информация, которую получает пользователь до подмены.

Меняем заголовок: `%0d%0aContent-Length:35%0d%0a%0d%0aYou are hacked%0d%0a<svg%20onload=alert(document.domain)>%0d%0a0%0d%0a/%2f%2e%2e`

![Процесс подмены информации1.png](https://github.com/karpuna3/shift2019/blob/master/crlf/img/Процесс%20подмены%20информации1.png)

Рисунок 6. Процесс подмены информации

![Результат подмены информации.png](https://github.com/karpuna3/shift2019/blob/master/crlf/img/Результат%20подмены%20информации.png)

Рисунок 7. Результат подмены информации.

## Ущерб
Данный тип атаки позволяет фальсифицировать данные в ответе от веб-сервера, добавлять cookie, проводить XSS, фишинг и т.д.

Также CRLF атака может открыть брешь для иных атак:
 - XSS, которая возможно приведет к краже сессии
 - Session Fixation
 - Open Redirect
 - Обход Stateless-защиты от CSRF - проверка сервером значения cookie и заголовка
 - Proxy and web server cache poisoning
 - Client web browser poisoning

Также стоит отметить тот факт, что CRLF может использоваться для фишинга.

## Защита

В большинстве веб-серверов реализована защита от CRLF инъекций. Если есть необходимость в реализации собственного функционала веб-сервера, то лучшим способом устранения уязвимостей разделения ответов HTTP является поиск во всех введенных пользователями данных символов CR/LF, т.е, `\r\n`, `%0d%0a` или любой другой формы их кодирования (или других небезопасных символов) перед использованием этих данных в любых заголовках HTTP.