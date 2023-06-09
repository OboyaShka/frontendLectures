# Angular CLI не нужен, используйте Nx - Илья Зяблицев

Angular CLI - npm-модуль, реализующий интерфейс командной строки для создания, разработки и поддержки Angular приложений.
* Стандартизация
* Кодогенерация
* Таск-раннер (dev/build/tests)

NX - open-souce система сборки, предоставляющая разработчкику набор инструментов и техник для в увеличения продуктивности и поддержки кода. 

Чем nx лучше angular cli?
* Можно настроить структуру проекта
* Больше полезного тулинга (eslint/cypress/pretier/storybook/jest)
* Project.json (замена angular.json, но под каждый проект)
* Generators/Executors (Замена schematics/builders)
* Кэширование (NX - cloud с помощью графа зависимостей. Платная фича)

[Сборщики](https://habr.com/ru/articles/450746) – это функции, реализующие логику и поведение для задачи, которая может заменить команду CLI. – Например, запустить линтер. Функция сборщика принимает два аргумента: входное значение (или опции) и контекст, который обеспечивает взаимосвязь между CLI и самим сборщиком. Разделение ответственности здесь такое же, как в Schematics – опции задает пользователь CLI, за контекст отвечает API, а вы (разработчик) устанавливаете необходимое поведение. Поведение может быть реализовано синхронно, асинхронно, либо же просто выводить определенное количество значений. Вывод обязательно должен иметь тип BuilderOutput, содержащий логическое поле success и необязательное поле error, содержащее сообщение об ошибке.

[Схематики](https://habr.com/ru/articles/469509) - это генератор кода на основе шаблонов, поддерживающий сложную логику. Это набор инструкций по преобразованию программного проекта путем создания или изменения кода. Схемы упаковываются в коллекции и устанавливаются с помощью npm.

Недостатки:
* nx migrate работает на более низком уровне, чем ng update
* нет ng add, всё ручками
* стабильность (+ очень частые релизы)
* версия angular и nx связаны

[Лекция](https://www.youtube.com/watch?v=d4h-pBUXdaM)
