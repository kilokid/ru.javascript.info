
# Устаревшее ключевое слово "var"

```smart header="Эта статья предназначена для понимания старых скриптов"
Информация, приведенная в этой статье, полезна для понимания старых скриптов.

Мы не пишем современный код таким образом.
```

В самой первой главе про [переменные](info:variables) мы ознакомились с тремя способами объявления переменных:

1. `let`
2. `const`
3. `var`

`let` и `const` ведут себя одинаково по отношению к лексическому окружению, области видимости.

Но `var` - это совершенно другой зверь, берущий своё начало с давних времён. Обычно `var` не используется в современных скриптах, но всё ещё может скрываться в старых.

Если в данный момент вы не работаете с подобными скриптами, вы можете пропустить или отложить прочтение данной главы, однако, есть шанс, что вы столкнётесь с `var` в будущем.

На первый взгляд, поведение `var` похоже на `let`. Например, объявление переменной:

```js run
function sayHi() {
  var phrase = "Привет"; // локальная переменная, "var" вместо "let"

  alert(phrase); // Привет
}

sayHi();

alert(phrase); // Ошибка: phrase не определена
```

...Однако, отличия всё же есть.

## Для "var" не существует блочной области видимости

Область видимости переменных `var` ограничивается либо функцией, либо, если переменная глобальная, то скриптом. Такие переменные доступны за пределами блока.

Например:

```js run
if (true) {
  var test = true; // используем var вместо let
}

*!*
alert(test); // true, переменная существует вне блока if
*/!*
```

Так как `var` игнорирует блоки, мы получили глобальную переменную `test`.

А если бы мы использовали `let test` вместо `var test`, тогда переменная была бы видна только внутри `if`:

```js run
if (true) {
  let test = true; // используем let
}

*!*
alert(test); // Error: test is not defined
*/!*
```

Аналогично для циклов: `var` не может быть блочной или локальной внутри цикла:

```js
for (var i = 0; i < 10; i++) {
  // ...
}

*!*
alert(i); // 10, переменная i доступна вне цикла, т.к. является глобальной переменной
*/!*
```

Если блок кода находится внутри функции, то `var` становится локальной переменной в этой функции:

```js run
function sayHi() {
  if (true) {
    var phrase = "Привет";
  }

  alert(phrase); // срабатывает и выводит "Привет"
}

sayHi();
alert(phrase); // Ошибка: phrase не определена (видна в консоли разработчика)
```

Как мы видим, `var` выходит за пределы блоков `if`, `for` и подобных. Это происходит потому, что на заре развития JavaScript блоки кода не имели лексического окружения. Поэтому можно сказать, что `var` - это пережиток прошлого.

## "var" допускает повторное объявление

Если в блоке кода дважды объявить одну и ту же переменную `let`, будет ошибка:

```js run
let user;
let user; // SyntaxError: 'user' has already been declared
```

Используя `var`, можно переобъявлять переменную сколько угодно раз. Повторные `var` игнорируются:
```js run
var user = "Pete";

var user; // ничего не делает, переменная объявлена раньше
// ...нет ошибки

alert(user); // Pete
```

Если дополнительно присвоить значение, то переменная примет новое значение:

```js run
var user = "Pete";

var user = "John";

alert(user); // John
```

## "var" обрабатываются в начале запуска функции

Объявления переменных `var` обрабатываются в начале выполнения функции (или запуска скрипта, если переменная является глобальной).

Другими словами, переменные `var` считаются объявленными с самого начала исполнения функции вне зависимости от того, в каком месте функции реально находятся их объявления (при условии, что они не находятся во вложенной функции).

Т.е. этот код:

```js run
function sayHi() {
  phrase = "Привет";

  alert(phrase);

*!*
  var phrase;
*/!*
}
sayHi();
```

...Технически полностью эквивалентен следующему (объявление переменной `var phrase` перемещено в начало функции):

```js run
function sayHi() {
*!*
  var phrase;
*/!*

  phrase = "Привет";

  alert(phrase);
}
sayHi();
```

...И даже коду ниже (как вы помните, блочная область видимости игнорируется):

```js run
function sayHi() {
  phrase = "Привет"; // (*)

  *!*
  if (false) {
    var phrase;
  }
  */!*

  alert(phrase);
}
sayHi();
```

Это поведение называется "hoisting" (всплытие, поднятие), потому что все объявления переменных `var` "всплывают" в самый верх функции.

В примере выше `if (false)` условие никогда не выполнится. Но это никаким образом не препятствует созданию переменной `var phrase`, которая находится внутри него, поскольку объявления `var` "всплывают" в начало функции. Т.е. в момент присвоения значения `(*)` переменная уже существует.

**Объявления переменных "всплывают", но присваивания значений - нет.**

Это проще всего продемонстрировать на примере:

```js run
function sayHi() {
  alert(phrase);  

*!*
  var phrase = "Привет";
*/!*
}

sayHi();
```

Строка `var phrase = "Привет"` состоит из двух действий:

1. Объявление переменной `var`
2. Присвоение значения в переменную `=`.

Объявление переменной обрабатывается в начале выполнения функции ("всплывает"), однако присвоение значения всегда происходит в той строке кода, где оно указано. Т.е. код выполняется по следующему сценарию:

```js run
function sayHi() {
*!*
  var phrase; // объявление переменной срабатывает вначале...
*/!*

  alert(phrase); // undefined

*!*
  phrase = "Привет"; // ...присвоение - в момент, когда исполнится данная строка кода.
*/!*
}

sayHi();
```

Поскольку все объявления переменных `var` обрабатываются в начале функции, мы можем ссылаться на них в любом месте. Однако, переменные имеют значение `undefined` до строки с присвоением значения.

В обоих примерах выше вызов `alert` происходил без ошибки, потому что переменная `phrase` уже существовала. Но её значение ещё не было присвоено, поэтому мы получали `undefined`.

## IIFE

В прошлом, поскольку существовал только `var`, а он не имел блочной области видимости, программисты придумали способ её эмулировать. Этот способ получил название "Immediately-invoked function expressions" (сокращенно IIFE).

Это не то, что мы должны использовать сегодня, но, так как вы можете встретить это в старых скриптах, полезно понимать принцип работы.

IIFE выглядит следующим образом:

```js run
(function() {

  var message = "Hello";

  alert(message); // Hello

})();
```

Здесь создаётся и немедленно вызывается Function Expression. Так что код выполняется сразу же и у него есть свои локальные переменные.

Function Expression обёрнуто в скобки `(function {...})`, потому что, когда JavaScript встречает `"function"` в основном потоке кода, он воспринимает это как начало Function Declaration. Но у Function Declaration должно быть имя, так что такой код вызовет ошибку:

```js run
// Пробуем объявить и сразу же вызвать функцию
function() { // <-- SyntaxError: Function statements require a function name

  var message = "Hello";

  alert(message); // Hello

}();
```

Даже если мы скажем: "хорошо, давайте добавим имя", -- это не сработает, потому что JavaScript не позволяет вызывать Function Declaration немедленно.

```js run
// ошибка синтаксиса из-за скобок ниже
function go() {

}(); // <-- нельзя вызывать Function Declaration немедленно
```

Так что скобки вокруг функции -- это трюк, который позволяет объяснить JavaScript, что функция была создана в контексте другого выражения, а значит, что это Function Expression: ей не нужно имя и её можно вызвать немедленно.

Помимо круглых скобок существуют и другие способы сообщить JavaScript, что мы имеем в виду Function Expression:

```js run
// Способы создания IIFE

*!*(*/!*function() {
  alert("Круглые скобки вокруг функции");
}*!*)*/!*();

*!*(*/!*function() {
  alert("Круглые скобки вокруг всего выражения");
}()*!*)*/!*;

*!*!*/!*function() {
  alert("Выражение начинается с логического оператора НЕ");
}();

*!*+*/!*function() {
  alert("Выражение начинается с унарного плюса");
}();
```

Во всех перечисленных случаях мы объявляем Function Expression и немедленно запускаем его. Ещё раз отметим: в настоящее время необходимости писать подобный код нет.

## Итого

Существует 2 основных отличия `var` от `let/const`:

1. Переменные `var` не имеют блочной области видимости, они ограничены, как минимум, телом функции.
2. Объявления (инициализация) переменных `var` производится в начале исполнения функции (или скрипта для глобальных переменных).

Есть ещё одно небольшое отличие, относящееся к глобальному объекту, мы рассмотрим его в следующей главе.

Эти особенности, как правило, не очень хорошо влияют на код. Блочная область видимости - это удобно. Поэтому много лет назад `let` и `const` были введены в стандарт и сейчас являются основным способом объявления переменных.
