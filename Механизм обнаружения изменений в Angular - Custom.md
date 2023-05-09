# Механизм обнаружения изменений в Angular - Custom

Современные веб-приложения очень динамичны и в них постоянно что-то происходит. Изменения данных должны отслеживаться, для того чтобы показать пользователю актуальное состояние приложения. Так как Angular является фреймвоком, который работает с Incremental DOM, то эти изменения нужно сначала заметить, потом собрать инструкции, которые произведут изменения в DOM. Так вот за поиск изменений отвечает механизм ChangeDetection совместно с NgZone.

Асинхронностей, которые могут изменить данные не так много. Запросы на сервер, таймаунты/интервалы и прослушиватели событий. И если мы будем знать, когда и где произошла асинхронность, то мы можем предположить где могли произойти изменения и проверить узкие места. 

Так и работает механизм ChangeDetection. Однако при создании асинхронностей, которые попадают в микро/макротаски мы можем потерять контекст выполнения и не знать, откуда асинхронность была вызвана и в какой момент она закончилась. Для этого была написана библиотека zone.js, которая создаёт единый контекст выполнения и переопределяет все асихнронности (monkey patching), трекая момент их завершения.

~~~js
class Context {
  constructor(parentContext) {
    let context;

    if (parentContext) {
      // Создаем копию
      context = Object.create(parentContext);
      context.parent = parentContext;
    } else {
      // Возвращаем текущий контекст
      context = this;
    }
    return context;
  }

  fork() {
    // Возвращаем копию
    return new Context(this);
  }

  bind(fn) {
    // Получаем текущий контекст
    const context = this.fork();
    // Возвращаем функцию в которой уже замкнут контекст
    return () => {
      return context.run(() => fn.apply(this, arguments), this);
    };
  }

  run(fn, context) {
    // Заменяем текущий контекст на наш
    let oldContext = context;
    context = this;
    const result = fn.call(); // Выполняем функцию в контексте
    context = oldContext; // Возвращаем как было
    return result; // Результат выполнения
  }
}

let context = new Context();

var nativeSetTimeout = window.setTimeout; // Подменяем setTimeout

context.setTimeout = (callback, time) => {
  callback = context.bind(callback);
  return nativeSetTimeout.call(window, callback.bind(context), time);
};

window.setTimeout = function () {
  return context.setTimeout.apply(this, arguments);
};

context.fork({} /* пустой объект, чтобы склонировалось*/).run(() => {
  context.message = "Привет!";
  setTimeout(() => {
    console.log(
      `%cСообщение в контексте: «${context.message}»`,
      "font-size: x-large"
    );
  }, 0);
});

console.log(
  `%cСообщение вне контекста: «${context.message}»`,
  "font-size: x-large"
);

var feedback = {
  message: "Привет!",
  send: function () {
    console.log(this.message);
  }
};

setTimeout(feedback.send, 0);

~~~

В свою очередь NgZone является обёрткой для zone.js и при завершении асинхронностей вызывает applicationRef.tick().

У каждого компонента есть свой ChangeDetection и при вызове метода detectChanges() происходит вызов движка рендеринга, который уже вызывает новую генерацию инструкций.
В свою очередь .tick вызывает у корневого элемента ngOnCheck, при котором компонент должен с помощью стратегии проверить, если у него изменения и если есть, то вызвать detectChanges().
Таким образом при ChangeDetectionStrategy.OnDefault, если ngOnCheck был вызван на нём, то все его дети тоже должны быть проверены, таким образом на проверке изменений подвержены все компоненты, что не является хорошим решением с точки зрения производительности так как render дорогостоящая процедура. 
При ChangeDetectionStrategy.OnPush компонент вызывает detectChanges() только при трёх случаях (напоминаю, что каждое асихронное событие триггерит tick()):
- изменение ссылок у переменных с декоратором @Input
- async pipe в шаблоне
- event вызванный в шаблоне компонента или его потомках

Таким образом, если в компоненте не будет выполнено хотя бы одно из условий, вся ветка компонентов останется нетронута ререндером. Однако, как уже было сказано можно вызывать detectChanges() принудительно или помечать (markForCheck()) компонент, чтобы при следубщем ngOnCheck компонент был проверен по дефолтной стратегии.

************************************************************************************************************************************************************************
Источники:
[Angular Ivy change detection execution: are you prepared?](https://medium.com/angular-in-depth/angular-ivy-change-detection-execution-are-you-prepared-ab68d4231f2c)
[Как работает NgZone?](https://www.youtube.com/watch?v=KVeX7oKQxlQ)
[Стратегий обнаружения изменений](https://www.youtube.com/watch?v=2cV4i-g6Oxc)
 





























