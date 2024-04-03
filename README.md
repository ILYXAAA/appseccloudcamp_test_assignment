# Тестовое задание в AppSec Cloud Camp

## :question: 1. Вопросы для разогрева
### 1. Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались?
&emsp; :key:С безопасной разработкой я сталкивался во время разработки своих пет-проектов. К примеру проект смены паролей от всех учётных записей по нажатию на одну кнопку, с последующим сохранением всех паролей в `.kdbx` хранилище `KeePass`. Приходилось реализовывать безопасное хранение API-ключей от сервисов, а также безопасное хранение ключей `.kdbx` базы данных. (При помощи хэширования)

### 3. Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было?
&emsp; :page_facing_up: На работе проводил моделирование угроз для описания пунктов акта проверки. 
Предполагаемые векторы атаки и модель нарушителя (разделение на внутренних/внешних, уязвимости по степени критичности и т.п.)

### 4. Если у вас был опыт поиска уязвимостей, расскажите, как это было?
&emsp; :iphone: Да, есть. Во время работы аудитором я отвечал за выявление уязвимостей в мобильных приложениях и внутренних API сервисов. 
Анализ приложений производился при помощи фреймворка `Frida`, `ADB` и утилиты `BurpSuite`. (Посредством установки сертификатов Burp).
Для автоматизации динамического анализа мобильных приложений под `Android` я сделал небольшую программу, которая позволяет внедрять js-скрипты в приложения. 
(такие скрипты, как `SSL Unpinning` и другие, подробнее: [codeshare](https://codeshare.frida.re/)). Ссылка на мой проект: [Frida Script Executor](https://github.com/ILYXAAA/frida_script_executor)

### 5. Почему вы хотите участвовать в стажировке?
&emsp; :bulb: Хочу получить больше практических навыков в AppSec для дальнейшей работы в этой сфере.

&emsp;
&emsp;

# :eyes: 2. Security code review.
## Часть 1. Security code review: GO.
### 1. Какие уязвимости присутствуют в этом фрагменте кода?
В данном фрагменте кода присутствуют уязвимости SQL-injection.

### 2. Указать строки, в которых присутствуют уязвимости.
```Go
searchQuery := r.URL.Query().Get("query")
```
> В данной строке формируется SQL-запрос с использованием пользовательского ввода

```Go
query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)
```
> В данной строке SQL-запрос формируется с помощью функции `fmt.Sprintf`, что может привести к инъекции SQL, если переменная `searchQuery` содержит вредоносные символы.

### 3. К каким последствиям может привести эксплуатация найденных уязвимостей злоумышленником?
Злоумышленник может осуществить атаку SQL-инъекции, в результате чего он может изменять, удалять или извлекать данные из БД. 

### 4. Описать способы исправления уязвимостей.
Для предотвращения SQL-инъекций можно использовать параметризованные или экранированные запросы. Это нужно чтобы разделять данные пользователя от кода SQL. В данном случае можно использовать параметризованные запросы для обработки пользовательского ввода.

Исправленная уязвимость:

```Go
query := "SELECT * FROM table_name WHERE name LIKE ?"
rows, err := db.Query(query, "%"+searchQuery+"%")
```

> Этот подход позволяет использовать "?" для параметров запроса. 

### 5. Если уязвимость можно исправить несколькими способами, необходимо перечислить их, выбрать лучший по вашему мнению и аргументировать свой выбор.
В параметризированных запросах значения параметров передаются отдельно от самого запроса, что позволяет базе данных правильно обрабатывать входные данные и предотвращает возможность внедрения вредоносного SQL кода.
* При использовании параметризованных запросов библиотека SQL сама обрабатывает все входные данные таким образом, чтобы они не могли изменить структуру запроса.
* Параметризованные запросы чаще всего более читаемы, так как они явно показывают, где и какие данные вставляются в запрос.
* Использование параметризованных запросов является более производительным вариантом.
&emsp;
&emsp;

## Часть 2: Security code review: Python
### Пример №2.1
### 1. Указать строки, в которых присутствуют уязвимости.
```Go
output = Template('Hello ' + name + '! Your age is ' + age + '.').render()
```

### 2. К каким последствиям может привести эксплуатация данных уязвимостей злоумышленником?
Если пользовательский ввод содержит специальные символы, такие как фигурные скобки "{}" или операторы `Jinja2`, то он может быть интерпретирован как код `Template` и выполнен. Это может привести к выполнению вредоносного кода на сервере.

### 3. Описать способы исправления уязвимостей.
Для исправления этой уязвимости следует использовать безопасные методы форматирования строк. Можно использовать метод `safe_substitute` из модуля `string.Template` для безопасного форматирования строк:
```Go
output = Template('Hello $name! Your age is $age.').safe_substitute(name=name, age=age)
```

> [!IMPORTANT]
> В режиме отладки Flask предоставляет информацию об ошибках, в теории эта информация может быть использована
> злоумышленниками для получения дополнительных данных во время _«разведки»_.
```python
app.run(debug=True)
```

&emsp;

### Пример №2.2
### 1. Указать строки, в которых присутствуют уязвимости.
```python
cmd = 'nslookup ' + hostname
output = subprocess.check_output(cmd, shell=True, text=True)
```

### 2. К каким последствиям может привести эксплуатация данных уязвимостей злоумышленником?
Злоумышленник может использовать уязвимость для выполнения произвольных команд на сервере, что может привести к компрометации системы, краже данных или причинению иного вреда. (Для примера, можно просмотреть список файлов в текущем каталоге, если передать строку _«google.com; dir»_ или _«google.com; ls»_ в качестве `hostname`).

### 3. Описать способы исправления уязвимостей.
Уязвимость можно исправить при помощи экранирования запроса. К примеру можно использовать метод `quote()` модуля `shlex`:
```python
import shlex
cmd = 'nslookup ' + shlex.quote(hostname)
```

### 4. Если уязвимость можно исправить несколькими способами, необходимо перечислить их, выбрать лучший по вашему мнению и аргументировать свой выбор.
Если в данной задаче в поле hostname требуется настоящий IP адрес пользователя – можно воспользоваться библиотекой `request.remote_addr` библиотеки Flask

# :beetle: 3. Моделирование угроз
## Проанализируйте диаграмму потоков данных приложения и ответьте на следующий вопросы
### Расскажите, какие потенциальные проблемы безопасности существуют для данного сервиса?
Честно говоря, сложно сказать что-то конкретное по данной схеме, т.к. по ней нельзя понять точную структуру сервиса (происходит ли обращение к сервису через API Gateway nginx и т.д.), поэтому перечислю то, что потенциально может бросаться в глаза.
1. Открытый доступ к `PostgreSQL`. Поток данных между **Backend Application** и `PostgreSQL` указан двунаправленным. Backend application отправляет запросы к хранилищу, не проходя через блок `Auth`. 
2. Открытый доступ к `S3`. **Backend application** отправляет запросы к хранилищу, не проходя через блок `Auth`.
3. Микрофронтенд напрямую общается с **Backend application**. Поток данных между `Microfront` и `Backend application` происходит, не проходя через блок `Auth`, а также возможно без шифрования (сложно точно сказать по схеме).
4. **Backend application** напрямую обменивается данными с внешними сервисами `Telegram` и `Slack` без дополнительной безопасности в виде шифрования (сложно точно сказать по схеме), а также без предварительной проверки передаваемых данных, во избежание содержания конфиденциальной информацию (предварительная очистка данных от ПД и тд).

### Расскажите, к каким последствиям может привести эксплуатация проблем, найденных вами?
1. Если злоумышленник получит несанкционированный доступ к базе данных `PostgreSQL` или хранилищу `S3` это может привести к утечке конфиденциальной информации. Это может нанести ущерб репутации компании, а также повлечь за собой финансовые взыскания.
2. Несанкционированный доступ к БД `PostgreSQL` может привести к нарушению целостности данных. Злоумышленник может управлять БД (удалять, изменять и добавлять данные).
3. Отсутствие блока `Auth` перед доступом в Backend часть может давать возможность **DDOS-атак** на сервера и компоненты внутреннего API.
4. Если кто-то получит доступ к недостаточно защищенным учетным записям пользователей, это может привести к компрометации их личных данных, а также к использованию их учетных записей для совершения незаконных действий.

### Расскажите, какие способы исправления уязвимостей и смягчения рисков вы можете предложить по отмеченным вами проблемам безопасности?
1. Выделить блок `Auth` в отдельный сервис, который будет стоять перед **Backend Application**. Это позволит обращаться к бэкэнду приложения только в случае успешной аутентификации и авторизации пользователем. Помимо **DDOS-атак**, это также поможет избежать несанкционированного доступа к логике внутреннего API неавторизованным пользователям. 
2. Настроить шифрование передаваемых, хранящихся в `S3` и `PostgreSQL` (если шифрования нет).
3. Организовать отдельный сервис «очистки» передаваемых данных от конфиденциальных или персональных данных, во избежание передачи таких данных на внешние ресурсы (`Telegram`, `Slack`).
5. Внедрить систему мониторинга безопасности, которая будет отслеживать подозрительную активность и аномалии в поведении системы. Реализовать процессы реагирования на инциденты.

### Обновленная схема с учетом исправленных проблем
![dgrm(4)](https://github.com/ILYXAAA/appseccloudcamp_test_assignment/assets/107761814/9457c18e-28af-4a90-b4f1-4e90c1e389f3)

### Напишите список уточняющих вопросов, которые вы бы задали разработчикам данного сервиса?
1. Какие механизмы аутентификации и авторизации используются в сервисе? 
2. Есть ли в сервисе отдельный микросервис выдачи временных сессионных ключей?
3. Происходит ли аутентификация на каждом микросервисе системы или же она происходит единоразово?
4. Предусмотрено ли в вашей системе сегментирование сети? Насколько компоненты системы изолированы друг от друга? 
5. Как происходит система контроля и учета доступа на физических объектах, хранилищах?
6. Как обеспечивается защита данных при передаче между компонентами системы?
7. Как обрабатываются и хранятся учетные данные пользователей, включая пароли и другие конфиденциальные данные, соответствуют ли условиям 152-ФЗ? 
8. Как разграничиваются права доступа к БД?
9. Предусмотрен ли в сервисе мониторинг подозрительной активности и реагирования на инциденты кибербезопасности?


