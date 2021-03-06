# Yet another deploy tool (yadt)

Развертывание новой версии конфигурации на базах подключенных к хранилищу 1С.

Предполагается схема поставки с использованием хранилища разработки и рабочего (production) хранилища. Соответственно предполагается следующий порядок развертывания:
1. Все доработки выполняются в базах, подключенных к хранилищу разработки
2. В момент релиза из хранилища разработки выгружается последняя версия конфигурации.
3. Конфигурация заливается в служебную базу для обновления рабочего (production) хранилища.
4. Рабочая база (или базы) обновляются из рабочего хранилища

![Storage Flow](./img/storageflow.png)

## Возможные команды
|||
|-|-|
| * **help** | - Вывод справки по параметрам |
| * **incver** | - Изменение версии подсистемы конфигурации |
| * **makecf** | - Создание cf-файла из последней версии указанного хранилища |
| * **updstorage** | - Обновление хранилища конфигурации из указанного cf-файла |
| * **updib** | - Обновление конфигурации ИБ из указанного хранилища конфигурации |


Для подсказки по конкретной команде наберите help <команда>

## incver - Изменение версии подсистемы конфигурации

Исходим из того, что выполняется существенная доработка конфигурации, но при этом предполагается регулярное обновление типовой. Нужно как-то версионировать систему, для чего создается общий макет, в котором хранится "наша" версия системы.
Данная команда выполняет:
1. Извлечение макета, содержащего версию из хранилища конфигурации
2. Изменение номера версии (инкремент) в соответствии с маской
3. Загрузку измененного макета в конфигурацию и помещение в хранилище


| Параметры: ||
|-|-|
| **-storage-path** | - Адрес хранилища конфигурации |
| **-storage-user** | - Имя пользователя хранилища конфигурации |
| **-storage-pwd** | - Пароль пользователя хранилища конфигурации |
| **-ver-tmplt** | - Имя общего текстового макета конфигурации, содержащего номер версии подсистемы |
| **-ver-mask** | - Маска версии подсистемы|
               # - оставить значение без изменения
               * - увеличить значение на 1
               $ - сбросить номер версии на 0 (для последнего числа на 1)
               <любые символы> - вставить указанные символы
       по умолчанию - "#.#.#.*"
|||
|-|-|
| **-ver-comment** | - Комментарий к изменению версии подсистемы в хранилище |
                    по умолчанию: "Изменена версия <Номер новой версии>"
                    для подстановки номера новой версии может использоваться символ подстановки %version%
|||
|-|-|
| **-v8version** | - Версия платформы 1С |


#### Пример:
```
yadt incver -storage-path "tcp://StorgaeServer/MyRepo" -storage-user MyStorageUser -ver-tmplt мое_ВерсияПодсистемы -ver-mask #.#.*.$ -ver-comment "Установлена версия %version%"
```

## makecf - Создание cf-файл из последней версии указанного хранилища

Команда выполняет выгрузку cf-файла из хранилища разработки для последующего обновления рабочего хранилища.


| Параметры: ||
|-|-|
| **-storage-path** | - Адрес хранилища конфигурации |
| **-storage-user** | - Имя пользователя хранилища конфигурации |
| **-storage-pwd** | - Пароль пользователя хранилища конфигурации |
| **-cf-path** | - Путь к выгружаемому cf-файлу |
| **-v8version** | - Версия платформы 1С |


#### Пример:
```
yadt makecf -storage-path "tcp://StorgaeServer/MyRepo" -storage-user MyStorageUser -cf-path d:\tmp\1cv8.cf
```

## updstorage - Обновление хранилища конфигурации из указанного cf-файла

Команда выполняет обновление рабочего хранилища конфигурацией из указанного cf-файла

| Параметры: ||
|-|-|
| **-storage-path** | - Адрес хранилища конфигурации |
| **-storage-user** | - Имя пользователя хранилища конфигурации |
| **-storage-pwd** | - Пароль пользователя хранилища конфигурации |
| **-ib-path** | - Адрес служебной ИБ для выполнения обновления (если не указана, то используется временная ИБ) |
| **-ib-user** | - Имя пользователя служебной ИБ |
| **-ib-pwd** | - Пароль пользователя служебной ИБ |
| **-upd-comment** | - Комментарий обновления |
| **-cf-path** | - Путь к cf-файлу обновления |
| **-delcf** | - Флаг удаления cf-файла после обновления |
| **-v8version** | - Версия платформы 1С |


#### Пример:
```
yadt updstorage  -storage-path "tcp://StorageServer/MyRepo" -storage-user MyStorageUser -ib-path "/FD:/data/MyDatabase" -cf-path d:\tmp\1cv8.cf -delcf
```

## updib - Обновление ИБ из хранилища

Команда выполняет обновление рабочей базы из рабочего хранилища.


| Параметры: ||
|-|-|
| **-ib-path** | - Адрес ИБ для обновления |
| **-ib-user** | - Имя пользователя ИБ |
| **-ib-pwd** | - Пароль пользователя ИБ |
| **-storage-path** | - Адрес хранилища конфигурации |
| **-storage-user** | - Имя пользователя хранилища конфигурации |
| **-storage-pwd** | - Пароль пользователя хранилища конфигурации |
| **-upd-db** | - Флаг обновления конфигурации БД |
| **-uccode** | - Код разрешения доступа к ИБ |
| **-v8version** | - Версия платформы 1С |


#### Пример:
```
yadt updib  -ib-path "/FMyServer\MyProductionDB" -ib-user Admin -ib-pwd P@ssw0rd -storage-path "tcp://StorageServer/MyRepo" -storage-user MyStorageUser
```

## Использование c Jenkins
В jenkinsfile описан конвейр выполняющий следующий сценарий:
* Изменение версии конфигурации в хранилище разработки
* Создание cf-файла для обновления рабочего хранилища
* Обновление рабочего хранилища
* Обновление рабочей ИБ из рабочего хранилища


| Переменные окружения конвейера ||
|-|-|
| **dev_storage_path** | - Адрес хранилища разработки |
| **dev_storage_cred** | - Идентификатор credentials для доступа к хранилищу разработки |
|||
| **ver_tmplt** | - Имя общего макета конфигурации, содержащего номер версии подсистемы |
| **ver_mask** | - Маска версии |
| **ver_comment** | - Комментарий к изменению версии |
|||
| **dev_cf_path** | - путь к файлу обновления конфигурации для выгрузки | 
| **upd_cf_path** | - путь к файлу обновления конфигурации для обновления | 
|||
| **dev_agent_label** | - Метка агента Jenkins, где будут выполнятся манипуляции с хранилищем разработки | 
| **upd_agent_label** | - Метка агента Jenkins, где будет выполнятся обновление рабочего хранилища |
| **prd_agent_label** | - Метка агента Jenkins, где будет выполнятся обновление рабочей базы |
|||
| **prd_storage_path** | - Адрес рабочего хранилища |
| **upd_storage_cred** | - Идентификатор credentials для обновления рабочего хранилища |
| **prd_storage_cred** | - Идентификатор credentials для получения конфигурации из рабочего хранилища |
|||
| **prd_ib_path** | - Путь к рабочей базе для обновления |
| **prd_ib_cred** | - Идентификатор credentials для доступа к рабочей базе |
