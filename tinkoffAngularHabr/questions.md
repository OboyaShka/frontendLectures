# Вопросы по Angular

## Общие

<details>
<summary>Что такое паттерн контроллера?</summary>
<br>
Бывают ситуации (например форма разбросана на несколько компонентов) и есть контролирующий её класс (компонент или директива). При стратегии OnPush при изменениях контроллера, зависящие дети не обновятся. В таком случае можно добавить Subject() и триггерить его при обновлениях, а в зависимых компонентах прокинуть его через DI и воспользоваться async пайпой.
  <a href="https://stackblitz.com/edit/angular-controller-pattern?file=src%2Fapp%2Fapp.component.html">Пример</a>
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
  
 
  

  
  
  
