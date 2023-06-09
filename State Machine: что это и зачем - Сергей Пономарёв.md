# State Machine: что это и зачем - Сергей Пономарёв

Проблема - при работе с приложениями мы часто сталкиваемся с различными состояниями приложения. При маштабировании и поддержке приложения следить за ними становится всё труднее, например куча флагов порождают n^2 состояний, при том многие из них не плоские (Некоторые состояния логически не могут существовать).

Как вариант, при работе с состояниями можно использовать State machine, это математическая абстракция, имеющая один вход и один выход и прибывающая в одном состоянии из конечно возможных.

В контексте фронтенда можно использовать [xState](https://xstate.js.org/) как реализацию state machine вместе с фреймворком. Вся бизнес логика связанная с состояниями выносится в state machine и записывается через редюсеры в общий state (детали реализации общего стейта). Тем самым логика работы с состояниями инкапсулируется на отдельный слой абстракции.

Плюсы
* Удобство работы с состояними
* Визуализация состояний (удобный dev tools)
* Маштабируемость сложности состояний

Минусы
* Некий overhead над бизнес логикой (Заранее подумать, нужно ли это в проекте)
* Подход не очень популярный, поэтому сложнее набирать людей в команду (Повышается порог входа в проект и время ознакомления с технологией + нужен особый тип мышления для работы со стейтами)
* Сложность с автотестами

[Лекция](https://www.youtube.com/watch?v=vlqtNtTMdpk)

Note 24.04.2023
Сама идея очень крутая! Состояния вынести в абстракцию, где можно удобно с ними работать. Не уверен, что это найдёт применение для большинства проектов, потому что обычной работы с состояниями хватает, но сразу на ум приходит событийная модель документа.

Есть куча состояний, которые зависят друг от друга и при этом генерируют уникальное поведение для каждого состояния. Например состояние "В работе/Ожидает подтверждения/Черновик/Утверждено" + роль сотрудника, который работает с документом "Рабочий/Начальник/Админ/Бугалтер" и фаза документа "Создание/редактирование/просмотр". Мне кажется тут интересно было бы использовать данный концепт.
