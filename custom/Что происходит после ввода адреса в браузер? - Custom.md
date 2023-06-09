# Что происходит после ввода адреса в браузер? - Custom

У нас есть адрес сайта, на который мы хотим попасть - https://github.com/OboyaShka

Для браузера важны 3 вещи
- Протокол (http)
- Домен (github.com)
- Ресурс (OboyaShka / Может быть физическим или виртуальным)

## Шаг первый - DNS Resolve.

Для открытия TCP/IP соединения и получения нужного ресурса, браузеру нужен IP адрес, стоящий за доменным именем. Для этого браузер использует DNS (Domain Name System).

Делает это он по следующему алгоритму:
1) Есть ли нужный ip в локальном кеше браузера
2) Есть ли нужный ip в /etc/hosts
3) Браузер подключается к Local DNS Resolver (Предоставляет провайдер связи) и проверяет кеш
4) Local DNS Resolver обращается к Root server, который возвращает адрес TLD (Top level domain / .com / .ru / etc...)
5) Local DNS Resolver обращается к TLD, который возвращает адрес Host server
6) Local DNS Resolver обращается к Host server, который возвращает нужный адрес
7) Local DNS Resolver обращается к выданному ip адресу и проверяет его на доступность/соединение и при удачном ответе кеширует
8) Local DNS Resolver возвращает нужный браузеру ip адрес

![image](https://user-images.githubusercontent.com/66056854/236718962-3651d72e-8fda-4b0e-ab5c-40afebc33d80.png)

## Шаг второй - Handshake.

Так как для TCP/IP соединения важен порт подключения, его нужно узнать. Для протокола http используется 80, а для https 433 порт.
Имея IP адрес, порт и ресурс можно открыть TCP/IP соединение. Но так как https это протокол предоставляющий защищённое соединение с шифрованием отправленных данных, между сервером и клиентом должна пройти фаза рукопожатия (Handshake).

Все TCP-соединения начинаются с тройного рукопожатия. До того как клиент и сервер могут обменяться любыми данными приложения, они должны «договориться» о начальном числе последовательности пакетов, а также о ряде других переменных, связанных с этим соединением. Числа последовательностей выбираются случайно на обоих сторонах ради безопасности.
- SYN Клиент выбирает случайное число Х и отправляет SYN-пакет, который может также содержать дополнительные флаги TCP и значения опций.
- SYN ACK Сервер выбирает свое собственное случайное число Y, прибавляет 1 к значению Х, добавляет свои флаги и опции и отправляет ответ.
- АСК Клиент прибавляет 1 к значениям Х и Y и завершает хэндшейк, отправляя АСК-пакет.

1) Браузер отправляет серверу набор алгоритмов шифрования с которыми он умеет работать (Для аутенфикации соединения используется асимметричные алгоритмы)
2) Сервер в ответ на это отправляет свой SSL сертификат и публичный ключ
3) Браузер проверяет SSL сертификат и отправляет свой публичный ключ
4) Благодаря алгоритму обмена ключами сервер и клиент обладают общим секретным ключом с помощью которого могут шифровать весь последующий трафик (симметричное шифрование) 

![image](https://user-images.githubusercontent.com/66056854/236722595-3562e569-403f-402a-8f54-6af2fe28fa06.png)

![image](https://user-images.githubusercontent.com/66056854/236721514-9bce6ea5-3a9e-40af-bbc2-81feb923fd83.png)

## Шаг третий - WebServer

После того как соединение установлено, мы можем получить нужные нам данные, например статический html файл. Но современные веб-приложения обслуживают большое количество пользователей, поэтому появилась надобность в прослойке веб-сервере (nginx/apache/etc...), которая играет роль распределителя нагрузки.

Веб-сервер перенаправляет запросы на менее загруженные сервера.

![image](https://user-images.githubusercontent.com/66056854/236723672-5ba8a76e-c327-4911-ab56-3dc695d5f6b6.png)

## Шаг четвёртый - Page rendering

На сервер посылается HTTP запрос. Важную часть играют заголовки, сообщающие серверу, как можно работать с клиентом.

Вот некоторые из них:
- Method (get)
- HTTP version (2.0, SPDY)
- Host (Начиная с 1.0 есть поддержка виртуальных хостов)
- connection (close / keep-alive, отключать ли tcp соединение)

Если всё успешно, возвращается статус код 200 вместе с html страницей, после чего запускается процесс парсинга. Парсинг не происходит за один обход и его алгоритм достаточно сложный. Происходит синтаксический, лексический анализ, создание DOM и CSSOM. После чего происходит создание rendering tree, вычисление позиций каждого элемента и отрисовки.

Под каждую вкладку браузер запускает собственный rendering process, чтобы каждая вкладка была изолированна от других и имела собственный поток выполнения. За это отвечают браузерный движок.

![image](https://user-images.githubusercontent.com/66056854/236738786-6714d27a-723e-4c4b-9304-2797f6f15a4b.png)
![image](https://user-images.githubusercontent.com/66056854/236739204-db44bea7-321e-4ccb-a01f-8d6447acf880.png)


В целом схема рендера выглядит следующим образом.
![image](https://user-images.githubusercontent.com/66056854/236726335-a0c59ca1-1f1c-4444-9a66-8cd94c821a44.png)
![image](https://user-images.githubusercontent.com/66056854/236732464-b72ebfa3-ae3f-4f1a-bf20-58304e34661b.png)

А теперь подробнее про каждый этап.

### HTML parsing

Браузер с помощью специального алгоритма проходится по html файлу и составляет DOM дерево, которое является результатом работы парсера. Натыкаясь на скрипты, главный поток блокируется на момент выполнения js кодом и парсинг останавливается. Именно поэтому раньше скрипты писали в конце тега body. Сейчас есть аттрибуты async и defer, которые не блокируют html парсер. При этому async не гарантирует очерёдность выполнения скриптов, а также может исполниться до создания DOM дерева, что может привести к ошибкам. В свою очередь defer тоже загружает скрипты асинхронно, но выполняет их после создания DOM дерева, но до события DOMContentLoaded.

Также html парсинг пытается исправить синтаксические ошибки при их наличии, что замедляет процесс парсинга.
Натыкаясь на файлы стилей, происходит их скачивание, после чего запускается процесс css парсинга

### CSS parsing

Браузер парсит css файлы, после чего состовляет CSSOM. Css тоже блокирует основной поток, но парсинг выполняется быстро, поэтому это не так критично.

### Rendering tree
CSSOM, в которой находятся только ноды,
После создания DOM и CSSOM, создаётся rendering tree. Это сущность, которая является слиянием DOM и CSSOM, в которой находятся только те ноды, которые должны быть отрисованы. Например нода с display: none или <head> не будут отрисованы. Также для текста создаются отдельные ноды.
Rendering tree объекты нужны для последующих стадий layout и painting.
  
### Styles calculation
Процесс, при котором стили применяются к DOM элементам.
Браузер пересчитывает стили, которые должны примениться из-за изменений, запланированных JS. Здесь же происходит вычисление активных media queries  
Важно, что css селекторы имеют разную производительность, что влияет на скорость вычисления стилей.
  
![image](https://user-images.githubusercontent.com/66056854/236742361-5229a25a-94a8-49f4-a486-53abaf96072d.png)

### Layout 
Вычисление слоев, расчет положения элементов на странице, их размеров, взаимного влияния друг на друга.
  
### Paint
Создание чертежа, описывающий как должна выглядеть итоговая страница, после чего его отправка GPU для отрисовки пикселей.
  
### Compositing
Работа со слоями. Браузер может выносить в отдельные, независимые слои какие-то элементы, при условии, что для их расчёта не нужно расчитывать положение других элементов.
Можно создать с помощью css свойств will-change: transfor/opacity + transform translate. Используется для анимация, чтобы они не останавливались при загрузке главного потока.
  
### Всё готово?

На этом страничка готова, но нужно сказать пару слов про события в renderQueue. Дело в том, что большинство экранов поддеживают 60fps, то есть обновлять содержимое страницы мы должны 1000/60 = ~ 16ms. Каждый 16ms операционная система просит браузер отдать актуальную информацию для рендера (просит выполнить renderQueue), но если главный поток занят, информация в gpu не уйдёт, что будет выглядеть как фриз. Соответственно нужно пытаться укладывать выполенения js скриптов в 16ms.
  
![image](https://user-images.githubusercontent.com/66056854/236754140-28615e6c-2452-4c92-82c2-62c0bc12179f.png) 
![image](https://user-images.githubusercontent.com/66056854/236754092-6cc8d9b7-d1ae-46be-b4a3-dd969c0764b2.png)
  
Соответственно renderQueue таски напрямую связаны с механизмом event loop. В какой-то момент event loop отвлекается (доделывает все микротаски), после чего выполняет RenderQueue состоящий из (requestAnimationFrame => Styles calculation => Layout => Paint => Compositing), если там что-то есть. 
- RequestAnimationFrame() - выполняется каждый раз в начале renderQueue
- Style calculation - Наиболее тяжёлая таска при первом рендере. Также добавляется при добавлении/изменении стилей.
- Layout - Первый раз при рендере страницы. Вызвается при чтении свойств влияющих на размер и положение элементов (force layout). 
- Paint - Покраска элементов, наибольшая нагрузка при первом рендере, потом проходит быстро.
- Compositing - Исполняется с помощью GPU, занимается слоями.
  
![image](https://user-images.githubusercontent.com/66056854/236745809-48a9041d-c1e6-4233-b4f1-80e275a6c009.png)  
  
Вот полная работа Event Loop

![image](https://user-images.githubusercontent.com/66056854/236752124-e42776f9-1699-492b-a6d0-df04796cbaef.png)
  
********************************************************************************************************************************
Источники:
- [Крутая статья по Event Loop](https://habr.com/ru/articles/517594/)
- [Крутая видос Event Loop](https://www.youtube.com/watch?v=zDlg64fsQow)
- [Что делает браузер, чтобы загрузить страницу?](https://www.youtube.com/watch?v=ylG8_d9Qk1U)
- [Как работает браузер?](https://www.youtube.com/watch?v=6aNT-ZmY9rU)
- [Клиентская оптимизация](https://www.youtube.com/watch?v=sMPO-DVJCH4)
- [Процесс загрузки веб-страницы](https://www.youtube.com/watch?v=jBvkN8_c7t8)
- [Шифрование в TLS/SSL](https://www.youtube.com/watch?v=kCkQRH5eweg)
