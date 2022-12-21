# Белов Вадим, 11-907

Для простоты работы с ролями ключи ```AWS_ACCESS_KEY_ID``` и ```AWS_SECRET_ACCESS_KEY``` от аккаунта ```oper-admin``` с ролью ```admin``` (у меня в облаке у этого аккаунта перечислены все админские роли, ибо тогда не знал, что можно просто админа бахнуть)

Что не получилось: подключить базу к serverless container, выходила странная не поддающаяся поиску ошибка. Из-за этого триггер шлёт сообщения из очереди кучу раз (ибо каждый раз запрос завершается ошибкой), поэтому при запуске в бакете faces будет много одинаковых фотографий.

1. Создать бакет ```itis-2022-2023-vvot02-faces``` и ```itis-2022-2023-vvot02-photos```.
2. Для бакета ```itis-2022-2023-vvot02-photos``` создаём триггер ```vvot02-photo-trigger``` на создание объекта с суффиксом .jpg, который вызывает функцию vvot02-face-detection.
3. Функцию ```vvot02-face-detection``` из предыдущего пункта нужно создать. У неё прописать переменные окружения ```AWS_ACCESS_KEY_ID``` и ```AWS_SECRET_ACCESS_KEY```. Cреда выполнения - python. Исходный код в файле ```vvot02-face-detection.py```. Там есть захардкоженная ссылка на очередь ```vvot00-tasks```. Так же нужно создать файл ```requirements.txt``` с текстом ```boto3```.
4. Предыдущая функцию помещает сообщения в очередь ```vvot00-tasks```, которую нужно создать.
5. На очередь ```vvot00-tasks``` навешиваем триггер ```vvot02-task-trigger```, который запускает контейнер ```vvot02-face-cut```.
6. Была создана база ```vvot02-db-photo-face``` с тремя столбцами: ```face_key```, ```face_name``` и ```original_key```. Все типа string.
7. Был создан реестр Container Registry ```yc container registry create --name my-first-registry``` и сконфигурирован докер командой ```yc container registry configure-docker```.
8. Был создан докер образ по файлу ```dockerfile``` (есть в гитхабе). Для этого образа был написан исходный код (```index.py```) и сгенерирован файл сервисного аккаунта командой: 
```yc iam key create --folder-id b1gperkgm8uqpk9heg41 --service-account-name oper-admin --output /oper-admin.json```. Оба этих файла должны лежать рядом с докер-файлом. В файле с исходным кодом захардкожено название бакета (```itis-2022-2023-vvot02-photos```, в двух местах) и данные базы данных (эндпоинт и база). Так же была выполнена подготовка докера командой  Образ был создан коммандой ```docker build . -t cr.yandex/crpeus0hurv22o1jdcbu/vvot02``` и загружен ```docker push cr.yandex/crpeus0hurv22o1jdcbu/vvot02:latest```
9. Затем был создан контейнер ```vvot00-face-cut``` по загруженному образу и с переменными окружения ```AWS_ACCESS_KEY_ID```, ```AWS_SECRET_ACCESS_KEY``` и ```SA_KEY_FILE```=```/oper-admin.json```
10. Создан API Gateway ```itis-2022-2023-vvot02-api``` (спецификация в файле ```itis-2022-2023-vvot02-api.yml```).
11. Была создана публичная облачная функция ```vvot00-boot``` (исходный код в файле ```vvot00-boot.py```). В файл ```requirements.txt``` была добавлена либа ```boto3```. Так же прописаны переменные окружения ```API_GATEWAY``` (ссылка на gateway),
```AWS_ACCESS_KEY_ID```, ```AWS_SECRET_ACCESS_KEY```, ```YDB_DATABASE``` (база данных), ```YDB_ENDPOINT``` (эндпоинт базы данных).
12. Бот (```https://t.me/vvot02_bot```) был создан через ```@BotFather```. Функция была зарегана как вебхук по ссылке ```https://api.telegram.org/bot{токен_бота}/setWebhook?url={Ссылка_на_функцию}```