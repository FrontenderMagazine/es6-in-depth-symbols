*[ES6 In Depth][1] - это серия новых фич, которые добавлены были в JavaScript в 6й версии редакции стандарта ECMAScript (кратко ES6)*

Заметка: Здесь представлен перевод статьи с [Вьетнамского][2], написанной
Джулией Дуонг из [команды Coupofy][3].

Что такое символы в ES6?

Символы - это не картинки.

Это и не смайлы, которые вы можете использовать в вашем коде.

`
let 😻 = 😺 × 😍;  // SyntaxError
`

Они также не являются литературным приемом для описания чего-либо.

И, безусловно, они не похоже на такие вещи как тарелки (прим. переводчика cymbals - англ. тарелка).

![][4]

Так что же *такое* символы?

### Седьмой тип

С тех пор как JavaScript был впервые стандартизирован в 1997 году, в нем было 6 типов. До ES6 каждое значение в JS приложении попадало в одну из категорий:

*   Undefined
*   Null
*   Boolean
*   Number
*   String
*   Object

Каждый тип представляет собой набор значений. Первые пять наборов конечны. Конечно, существует только два Логических значения - `true` и `false` и они не порождают новые. С другой стороны, существует большое количество значений Числовых и Строковых типов. Стандарт говорит, что существует 18437736874454810627 - разных чисел (включая `NaN` - это Числовой тип, чье сокращение можно расшифровать как “Не число”). Это ничто по сравнению с числом различных возможных Строковых значений, которых я думаю (2^(144,115,188,075,855, 872) − 1) ÷ 65,535 ... хотя возможно, я просчитался.

Набор значений типа Объект неограничен. Каждый объект является уникальным. Каждый раз при открытии веб страницы создается множество новых обектов.

ES6 символы это значения, которые не являются ни строками ни объектами. Они что-то новое - седьмой тип значений.

Давайте поговорим о сценариях, где их использование могло бы нам пригодиться.

### One simple little boolean

Иногда бывает очень удобно спрятать некоторые дополнительные данные в JavaScript объект, который в действительности принадлежит кому-то другому.

Предположим, что вы пишете JS библиотеку, которая использует CSS переходы, чтобы сделать элементы DOM движущимися по экрану. Попытка применить несколько переходов CSS к одному `div` не работает, что вызывает некрасивые, прерывистые "прыжки". Возникает вопрос, как можно исправить эту проблему. Сначала нужно найти способ определить, движется ли уже данный элемент?

Как можно решить эту проблему?

Одним из возможных путей решения является использование CSS API, для определения движения элемента в браузере. Но это выглядит излишне. Ваша библиотека *должна сама знать* движется ли элемент; it’s the code that set it moving in the first place!

То, что вам нужно это найти способ за тем, какие элементы движутся. Вы
может держать массив всех движущихся элементов. Каждый раз, когда ваша библиотека вызывается при анимации элемента, вы можете просто искать этот элемент в массиве.

Хмм.. Но линейный поиск будет медленным, если массив большой.

Другой вариант - это просто установить флаг на элементе:

`
if (element.isMoving) {
  smoothAnimations(element);
}
element.isMoving = true;
`

При таком подходе существует несколько потенциальных проблем. Все они связаны с тем, что ваш код работает не только с DOM'ом.

1.  Другой код, использующий `for-in` или `Object.keys()`, может наткнуться на свойства, уже созданные вами.

2.  Некоторые другие умные библиотеки, где автор возможно думал об этой технике вначале. 
    И Ваша библиотека возможно будет плохо взаимодействовать с ней.

3.  Некоторые другие умные библиотеки, где автор возможно думал об этой технике в будущем.
    И Ваша библиотека возможно будет плохо взаимодействовать с ней в будущем.

4.  Обычный комитет может решить добавить `.isMoving()` ко всем элементам. 
    Тогда у вас будут *реальные* проблемы с нагрузкой!

Конечно, вы можете сказать, что последние три проблемы связаны с выбором строки, поэтому кажется глупо, что никто никогда не будет так называть свойства:

`
if (element.__$jorendorff_animation_library$PLEASE_DO_NOT_USE_THIS_PROPERTY$isMoving__) {
  smoothAnimations(element);
}
element.__$jorendorff_animation_library$PLEASE_DO_NOT_USE_THIS_PROPERTY$isMoving__ = true;
`

Кажется, это не стоит того напряжения для глаз.

Вы можете генерировать практически уникальное имя для свойства, используя криптографию:

`
// получить 1024 Юникод символа абракадабры
var isMoving = SecureRandom.generateName();

...

if (element[isMoving]) {
  smoothAnimations(element);
}
element[isMoving] = true;
`

Следующий синтакс: `object[name]` позволяет Вам использовать буквально любую строчку в качестве имени свойства.
Поэтому это будет работать: коллизии практически невозможны, и Ваш код выглядит намного лучше.

Но это приводит к неудобному дебаггингу. Каждый раз, когда вы делаете `console.log()` 
для элемента с таким свойством, Вы будете видеть большую строчку. А если Вам нужно больше, чем одно подобное свойство?
Как Вы будете следить за ними? При каждой перезагрузке они будут иметь разные имена.

Почему это так сложно? Нам нужен просто один маленький boolean!

### Символы как ответ

Символы - это значения, которые программа может создать и использовать как названия свойств без риска повтора (коллизий).

`
var mySymbol = Symbol();
`

Вызывая `Symbol()`, Вы создаете новый символ, значения которого не равно любому другому объекту.

Так же, как и строчку из цифр, Вы можете использовать символ как имя свойства. 
Потоу что оно уникально, и свойство с таким именем гарантированно не будет повторяться с любым другим свойством.

`
obj[mySymbol] = "ok!";  // гарантированно уникально
console.log(obj[mySymbol]);  // ok!
`

Здесь Вы видите, как можно использовать символ в ситуации, описанной выше:

`
// создать уникальный символ
var isMoving = Symbol("isMoving");

...

if (element[isMoving]) {
  smoothAnimations(element);
}
element[isMoving] = true;
`

Несколько замечаний по этому коду:

*   Строка `"isMoving"` в `Symbol("isMoving")` называется "описание". Это облегает дебаггинг.
    Это значение будет показано, когда Вы выводите символ на консоль, и когда Вы конертируете 
    символ в строчку с помощью `.toString()`, а так же в возможных сообщениях ошибок.   

*   `element[isMoving]` называется свойство с ключом-символом. Это просто свойство, у которого имя - это не строчка, а символ.
	Кроме этого, это обычное свойство.   

*   Так же как и массивы элементов, свойства с ключами-символами не могут быть доступны через точку, как `obj.name`. 
	Она должны быть доступны через квадратные скобки.
	
*   Если у Вас есть символ, то получить свойство с ключом-симолом элементарно. 
	Пример выше показывает, как получить и задать значение `element[isMoving]`,
    и мы так же можем спросить `if (isMoving in element)` или даже `delete element[isMoving]`, если нам это требуется.

*   С другой стороны, всё это возможно до тех пор, пока `isMoving` находится в текущем блоке.
	Это делает механизм символов слабым к инкапсуляции: модуль, который создает несколько символов для себя,
	может использовать для любых объектов **без страха, что они повторятся** со свойствами, созданными в другом коде.

Т.к. символы были созданы для того, чтобы избежать коллизий, наиболее распространенный 
просмотр объектов в JS просто игнорирует символы. Например, в цикле A`for-in` происходит перебор всех свойств-строк.
Свойства-символы пропускаются.
`Object.keys(obj)` и `Object.getOwnPropertyNames(obj)` делают то же самое. Но символы не приватны: с помощью нового АПИ 
`Object.getOwnPropertySymbols(obj)` можно получить список ключей-символов объекта.
Другое новое АПИ, `Reflect.ownKeys(obj)`, возвращает все свойства: и строки, и символы. (Мы обсудим `Reflect` АПИ подробнее в следующих статьях)

Библиотеки и фреймворки скорее всего найдут множество применений символам, и мы увидим это позже, 
язык сам по себе использует их для широкого круга целей.

### Но конкретно, что такое символы?

`
> typeof Symbol()
"symbol"
`

Символы не похожи ни на что другое.

После создания из невозможно изменить Вы не можете задать их свойства 
(и если вы попробуете сделать это в strict mode, вы получите TypeError).
Они могут быть именами свойств. Здесь они похожи на строки.

С другой стороны, каждый символ уникален, отличен от всех других символов
(даже если другие имеют то же самое описание), и Вы можете легко создать новый.
Здесь они похожи на объекты.

Символы в ES6 похожи на [традиционные символы][5] в языках List, Ruby,
но 

ES6 symbols are similar to the [more traditional symbols][5] in languages like
Lisp and Ruby, but not so closely integrated into the language. In Lisp, all 
identifiers are symbols. In JS, identifiers and most property keys are still 
considered strings. Symbols are just an extra option.

One quick caveat about symbols: unlike almost anything else in the language,
they can’t be automatically converted to strings. Trying to concatenate a symbol
with strings will result in a TypeError.


``
> var sym = Symbol("<3");
> "your symbol is " + sym
// TypeError: can't convert symbol to string
> `your symbol is ${sym}`
// TypeError: can't convert symbol to string
</pre>
``

You can avoid this by explicitly converting the symbol to a string, writing 
`String(sym)` or `sym.toString()`.

### Three sets of symbols

There are three ways to obtain a symbol.

*   **Call `Symbol()`.** As we already discussed, this returns a new unique
    symbol each time it’s called.
   

*   **Call `Symbol.for(string)`.** This accesses a set of existing symbols
    called the
   symbol registry. Unlike the unique symbols defined by `Symbol()`, symbols in
    the symbol registry are shared. If you call
   `Symbol.for("cat")` thirty times, it will return the *same* symbol each time
    . The registry is useful when multiple web pages, or multiple modules within the
    same web page, need to share a symbol.
   

*   **Use symbols like `Symbol.iterator`, defined by the standard.** A few
    symbols are defined by the standard itself. Each one has its own special purpose.
   

If you still aren’t sure if symbols will be all that useful, this last
category is interesting, because they show how symbols have already proven 
useful in practice.

### How the ES6 spec is using well-known symbols

We’ve already seen one way that ES6 uses a symbol to avoid conflicts with
existing code. A few weeks ago, in[the post on iterators][6], we saw that the
loop`for (var item of myArray)` starts by calling `myArray[Symbol.iterator]()`
`myArray.iterator()`, but a symbol is better for backward compatibility.

Now that we know what symbols are all about, it’s easy to understand why this
was done and what it means.

Here are a few of the other places where ES6 uses well-known symbols. (These
features are not implemented in Firefox yet.
)

*   **Making `instanceof` extensible.** In ES6, the expression 
    `<var>object</var> instanceof <var>constructor</var>` is
    specified as a method of the constructor:
   `constructor[Symbol.hasInstance](object)`. This means it is extensible.

*   **Eliminating conflicts between new features and old code.** This is
    seriously obscure, but we found that certain ES6
   `Array` methods broke existing web sites *just by being there.* Other Web
    standards had similar problems: simply adding new methods in the browser would 
    break existing sites. However, the breakage was mainly caused by something 
    called
   dynamic scoping, so ES6 introduces a special symbol, `Symbol.unscopables`,
    that Web standards can use to prevent certain methods from getting involved in 
    dynamic scoping.
   

*   **Supporting new kinds of string-matching.** In ES5, `str.match(myObject)`
    tried to convert
   `myObject` to a `RegExp`. In ES6, it first checks to see if `myObject` has a
    method
   `myObject[Symbol.match](str)`. Now libraries can provide custom string-
    parsing classes that work in all the places where
   `RegExp` objects work.

Each of these uses is quite narrow. It’s hard to see any of these features by
themselves having a major impact in my day-to-day code. The long view is more 
interesting. Well-known symbols are JavaScript’s improved version of the
`__doubleUnderscores` in PHP and Python. The standard will use them in the
future to add new hooks into the language with no risk to your existing code.

### Когда я смогу использовать ES6 символы?

Символы уже реализованы в Firefox 36 and Chrome 38. Я реализовал их для Firefox самостоятельно, поэтому если символы у вас не работают как положено, вы знаете к кому обратиться.

Для подержки браузеров которые еще не имеют нативной поддержки ES6 символов, вы можете использовать полифилы, такие как [core.js][7]. Тем ни менее символы не похожи ни на что, что было ранее в языке, поэтому полифилы также не совершенны. [См. предостережения][8]

На следующей неделе будут две новых статьи. В первой, мы рассмотрим некоторые долгожданные функции, которые, наконец, пришли в JavaScript в ES6 - и жалуемся на них. Мы начнем с двух функций, которые датируются почти до "рассвета программирования". И продолжим двумя новыми фичами, которые очень похожи, но *работают по новому*. Поэтому присоединяйтесь к нам на следующей неделе и мы рассмотрим ES6 коллекции изнутри.

*And,* stick around for a bonus post by Gastón Silva on a topic that isn’t
an ES6 feature at all, but might provide the nudge you need to start using ES6 
in your own projects. See you then!

[Другие статьи Jason Orendorff…][9]

 [1]: https://hacks.mozilla.org/category/es6-in-depth/
 [2]: http://translate.coupofy.com/es6-in-depth-symbols/
 [3]: http://www.coupofy.com
 [4]: img/302555333_c3f827fd75_z-250x188.jpg
 [5]: https://en.wikipedia.org/wiki/Symbol_%28programming%29

 [6]: https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/
 [7]: https://github.com/zloirock/core-js#ecmascript-6-symbols
 [8]: https://github.com/zloirock/core-js#caveats-when-using-symbol-polyfill
 [9]: https://hacks.mozilla.org/author/jorendorffmozillacom/