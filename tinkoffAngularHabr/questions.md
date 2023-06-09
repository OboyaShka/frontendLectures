# Вопросы по Angular

## Общие

<details>
<summary>Что такое паттерн контроллера?</summary>
<br>
Бывают ситуации (например форма разбросана на несколько компонентов) и есть контролирующий её класс (компонент или директива). При стратегии OnPush при изменениях контроллера, зависящие дети не обновятся. В таком случае можно добавить Subject() и триггерить его при обновлениях, а в зависимых компонентах прокинуть его через DI и воспользоваться async пайпой.
  <a href="https://stackblitz.com/edit/angular-controller-pattern?file=src%2Fapp%2Fapp.component.html">Пример 1</a>
   <a href="https://github.com/OboyaShka/angular-wbb/tree/master/apps/angular-wbb/src/app/modules/di/examples/di-example8/di-example8/directives/controllers">Пример 2</a>
</details>

<details>
<summary>Какие есть способы взаимодействия между компонентами и директивами?</summary>
<br>
<ul>
<li>Outputs</li>
<li>@Input() ngOnChanges + settes </li>
<li>Shared service</li>
<li>ViewRef trigger</li>
<li>Template trigger</li>
<li>Di get element</li>
</ul>
</details>

<details>
<summary>Что такое HostListener и HostBinding?</summary>
<br>
HostListener нужен для прослушки событий хост элемента. HostListener есть следующая проблема: после триггера события вызвается cdr, что может быть критично для часто повторяющихся событий, например скроллы. Можно написать специальные плагины, с помощью EventManager'а, которые будут влиять на обработку событий, а именно выходить из зоны и возвращаться в неё только при определённом условии, которое можно описать декоратором.
HostBinding используется для привязки аттрибута/класса с переменной шаблона. Иногда нужно работать с rxjs и удобно было бы сделать значение HostBinding стримом, но это недоступно из коробки. Реализовать это можно с помощью тех же плагинов, которые будут завязываться на аттрибуте и генерировать стрим. Тем самым можно вообще отказаться от HostBinding'а и нативно завязывать переменную на стрим.
  <a href="https://github.com/OboyaShka/angular-wbb/blob/master/apps/angular-wbb/src/app/modules/rxjs/examples/rxjs-example23/rxjs-example23.component.ts">Пример</a>  
</details>

## Change Detection

<details>
<summary>Что такое view?</summary>
<br>
Для манипуляций с DOM-элементами в Angular используются так называемые абстракции, которые представлены классами ElementRef, TemplateRef, ViewRef, ComponentRef и ViewContainerRef.

Сами абстракции представляют шаблон компонента или его отдельные части.
</details>

<details>
<summary>Что такое tick()?</summary>
<br>
Механизм, который запускается ngZone при срабатывании асинхроннстей и проходится по дереву View, проверяя их на изменения
</details>

<details>
<summary>В каких ситуациях происходит проверка изменений при OnPush?</summary>
<br>
<ul>
<li>Принудительный вызов cdr</li>
<li>Срабатывание events на компоненте или у потомков</li>
<li>Изменении ссылки в Input()</li>
  </ul>
</details>
  
<details>
<summary>Какие различия между markForCheck и detectChanges?</summary>
<br>
<ul>
  <li>markForCheck срабатывает асинхронно, отмечая, что ноду нужно проверить при следующем тике + помечает верхние ноды как, если бы произошёл event</li>
  <li>detectChanges срабатывает синхронно, на следующией строке все события lifecycle компонента уже произойдут</li>
 </ul>
</details>
  
<details>
<summary>Зачем нужны методы runOutsideAngular и run у ngZone?</summary>
<br>
<ul>
  <li>runOutsideAngular() нужен для выходы из зоны, чтобы лишний раз не создавались тики (например при requestAnimationFrame)</li>
  <li>run() патчит события в зону, нужно например для iframe, события из которого не прокают тики</li>
 </ul>
</details>
  
## Dependency injection
<details>
<summary>Для чего нужен forwardRef()?</summary>
<br>
Для того, чтобы указать, что значение запровайдится позже. В связке с useExisting позволяет указать на сущность, которая ещё не создана, но мы утверждаем, что будет. Например можно запровайдить в компоненте сам компонент, чтобы ниже из диркутивы получить родительский компонент. Но нужно быть осторожно с циклическими зависимостями.  
</details>

<details>
<summary>Какие есть основные способы провайда?</summary>
<br>
<ul>
<li>useClass</li>
<li>useExisting</li>
<li>useFactory</li>
<li>useValue</li>
</ul>
</details>

## Render

<details>
<summary>Что такое Renderer2?</summary>
<br>
Абстрактный класс, который описывает методы работы с render нодами. По стандарту Ангуляр реализует класс для работы в DOM, но можно использовать RendererFactory2 для написания кастомной логики для работы в других средах. Таким образом renderer2 является своего рода адаптерам, который можно подменять своей имплиментацией для работы приложения на бэке/мобилке и т.д. 
В компоненте можно достать из DI Renderer2 и используя различные методы работать с DOM.
</details>

<details>
<summary>Что такое view?</summary>
<br>
View является представлением элементов отображения пользовательского интерфейса. Директивы на компоненте, класс компонента и связанный с ним шаблон определяют представление. Представление конкретно представлено экземпляром ViewRef, связанным с компонентом. Представления обычно объединяются в иерархии представлений, с которыми работает механизм ChangeDetection
</details>

<details>
<summary>Какие есть класс/абстракции для работы с DOM на уровне View?</summary>
<br>
ViewRef, ElementRef, TemplateRef, ContainerViewRef, ComponentRef. 
<ul>
  <li>ElementRef - хранит в себе "оригинальный" HTML-элемент в свойстве nativeElement</li>
  <li>TemplateRef - содержит единственное свойство elementRef, содержащее экземпляр класса ElementRef, который в свою очередь ссылается на host-элемент</li>
  <li>ComponentRef - возвращается при динамическом создании компонента</li>
  <li>ContainerViewRef - является контейнером, в который можно вставлять ViewRef элементы. Может быть любым DOM элементом, но лучше использовать &lt;ng-container/&gt;. Важно, что view вставляются не в сам элемент, а после него.
    С помощью методов createEmbeddedView() и createComponent() можно создавать внутри view внутри контейнераю.
  </li>
  <li>ViewRef - является отображением представления. Делится на 2 вида - Host и Embedded (ng-template). Такие представления могут быть вставлены в ContainerViewRef. Также они составляют иерархию представлений с которой работет cdr.
  ViewRef самого компонента нельзя получить напрямую (Используя @Inject(ChangeDetectorRef) private view: ɵViewRef<YourComponent> можно посмотреть как она выглядит), потому что Angular предполагает работу с View текущего компонента через шаблон и набор декораторв</li>
 </ul>
</details>

<details>
<summary>Какие есть декораторы для работы с view?</summary>
<br>
Для получения элементов из view используются декораторы ViewChild, ViewChildren, ContenChild, ContentChildren. Приписка Child означает, что мы хотим найти первый попавшийся элемент по селектору, а Children - итерируемую коллекцию QueryList.
View достаёт элементы из текущего шаблона, а Content - из шаблона родителя, которую передали контентом в текущий компонент. В декоратор передаётся селектор по которому будут выбраны элементы, read для проверки полученных значений по типу, static для обозначения, что элемент является вёрсткой без динамических значений, чтобы работать с ним начиная с ngOnChanges.
</details>



  
  
  
