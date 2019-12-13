## Źródła
https://basarat.gitbooks.io/typescript/

## powershell
npm install -g @angular/cli
- instaluje nam command line interface dla angulara

ng new TestAppka
- Tworzy nową appke + wszystkie zależności itp

code .
 - Odpala dany folder w VSCode

## vscode terminal
ng serve -- open
- uruchamia serwer angulara i uruchamia appke w nowym oknie

## angular debug
DEBUG appki ANG:
F12, Sources, webpack:// . src app app.component.ts


## JS

### Falsy i Truthy
```
console.log(5 == "5"); // true   , TS Error
console.log(5 === "5"); // false , TS Error
console.log("" == "0"); // false
console.log(0 == ""); // true
```
wartości `""` i `0` są `falsy`
`!!value` - Zmiana wartości z truthy/falsy na prawdziwy boolean

### Porównywanie
`==` porównanie z "naprostownaiem" typów.
`===` to dokładne przyrównanie.

#### Porównywanie obiektów
```
import * as deepEqual from "deep-equal";
console.log(deepEqual({a:123},{a:123})); // True
```
Zwykłe porównanie obiektów, jak i przypisywanie ich, dokonuje się przez referencje.
Ale łatwiej porównać id:
```
type IdDisplay = {
  id: string,
  display: string
}
const list: IdDisplay[] = [
  {
    id: 'foo',
    display: 'Foo Select'
  },
  {
    id: 'bar',
    display: 'Bar Select'
  },
]
```
Lub poprzed mapowanie:
```
const fooIndex = list.map(i => i.id).indexOf('foo');
console.log(fooIndex); // 0
```
### Undefined i null

Something hasn't been initialized : undefined.
Something is currently unavailable: null.
```
// Both null and undefined are only `==` to themselves and each other:
console.log(null == null); // true (of course)
console.log(undefined == undefined); // true (of course)
console.log(null == undefined); // true
```
ALE
```
// You don't have to worry about falsy values making through this check
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```
Zaleca się po prostu korzystanie z ==null, by sprawdzić oba. Jeżeli coś nie jest nullem, to undefinedem też nie będzie.
```
function foo(arg: string | null | undefined) {
  if (arg != null) {
    żeby tu wejść, arg musi być stringiem. Na pewno nie jest null i undefined.
  }
}
```
W strict mode nie można użyć undefined zmiennych. Wpadnie ReferenceError exception i wyjebie wszystko.
```
if (typeof someglobal !== 'undefined') {
  // someglobal is now safe to use
  console.log(someglobal);
}
```
Zaleca się też ograniczanie użycia undefined explicit w typescript. Skoro mamy typy i możliwości, to zamiast:
```
function foo(){
  // if Something
  return {a:1,b:2};
  // else
  return {a:1,b:undefined};
}
```
Możemy:
```
function foo():{a:number,b?:number}{
  // if Something
  return {a:1,b:2};
  // else
  return {a:1};
}
```
Funkcje w stylu node zazwyczaj są wywoływane z parametrem err na null zatem możemy:
```
fs.readFile('someFile', 'utf8', (err,data) => {
  if (err) {
    // do something
  } else {
    // no error
  }
});
```
Brakujące wartości w erorrach można też wyłapywać .then lub .catch
OKROPNOŚCI:
```
function toInt(str: string) {
  return str ? parseInt(str) : undefined;
}
```
PYSZNOŚCI:
```
function toInt(str: string): { valid: boolean, int?: number } {
  const int = parseInt(str);
  if (isNaN(int)) {
    return { valid: false };
  }
  else {
    return { valid: true, int };
  }
}
```
Dlaczego okropności? Nie powinno się używać undefined aby sprawdzić czy coś jest valid.
parseFloat i parseInt zwracają NaN, kiedy wyliczą wartość nie będącą liczbą.
NaN to wartosć, która nie jest reprezentowalna poprawnym numerem. Np. "Math.sqrt(-1)"

Poza tym json - Atrybuty z wartością undefined będą pominięte przy Json.stringify.
```
JSON.stringify({willStay: null, willBeGone: undefined}); // {"willStay":null}
```

### Calling context - this

```
function foo() {
  console.log(this);
}

foo(); // logs out the global e.g. `window` in browsers
let bar = {
  foo
}
bar.foo(); // Logs out `bar` as `foo` was called on `bar`
```

### Closure
```
function createCounter() {
    let val = 0;
    return {
        increment() { val++ },
        getVal() { return val }
    }
}

let counter = createCounter();
counter.increment();
console.log(counter.getVal()); // 1
counter.increment();
console.log(counter.getVal()); // 2
```
Spoko przykład closure:
https://itnext.io/javascript-closure-for-privacy-8a40c274192e
Serio fajny, poczytaj go pare razy xD

## JS NOWE FUNKCJE

### Klasy
```
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```
Użycie właściwości statycznych w klasach:
```
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```
Uproszczony konstruktor:
```
class Foo
{
    x: number;
    constructor(x:number)
    {
        this.x = x;
    }
}

RÓWNOZNACZNE Z

class Foo
{
    constructor(public x:number)
    {
    }
}
```

### Arrow Function
Zapis strzałkowy zachowuje kontekst this:
```
function Person(age) {
    this.age = age;
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000);
// 1, should have been 2 (growOld jest wykonywany przez window)

function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000);
// 2
```

Niektóre biblioteki jak mocha czy jquery używają `this` jako calling context.
Moższ zrobić bezpieczną wersję:
```
let _self = this;
something.each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```
funkcje strzałkowe działają spoko z dziedziczeniem, ale słabo z super. Rozwiązanko:
```
class Adder {
    constructor(public a: number) {}
    // This function is now safe to pass around
    add = (b: number): number => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Create a copy of parent before creating our own
    private superAdd = this.add;
    // Now create our override
    add = (b: number): number => {
        return this.superAdd(b);
    }
}
```

### Przejmowanie pozostałości:
```
function iTakeItAll(first, second, ...allOthers) {
    console.log(allOthers);
}
iTakeItAll('foo', 'bar'); // []
iTakeItAll('foo', 'bar', 'bas', 'qux'); // ['bas','qux']
```

### Scope funkcji:

```
var foo = 123;
if (true) {
    var foo = 456;
}
console.log(foo); // 456
```
Wąsy nie powodują powstania nowego scope dla zmiennej `var` - zmienne zadeklarowane varem są scopowane do całej funkcji, nie do bloku.
```
var foo = 123;
function test() {
    var foo = 456;
}
test();
console.log(foo); // 123
```
Rozwiązaniem jest słowo kluczowe `let`

```
let foo = 123;
if (true) {
    let foo = 456;
}
console.log(foo); // 123
```
Ogólnie najlepiej używać `let`.
let jest tłumaczony na var - W razie czego zmienia mu się nazwę zmiennej. (np. `foo` -> `foo_1`

let pomaga też w takich sytuacjach:
```
var funcs = [];
// create a bunch of functions
for (var i = 0; i < 3; i++) {
    funcs.push(function() {
        console.log(i);
    })
}
// call them
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```
Wynik to "3 3 3" - var i nie jest zamknięty w funkcji `console.log(i)`, więc jego stan jest tutaj globalny (Zakładam, że to cały kod). Będziemy mieli zatem w pamięci ostatni stan i.

Zamiast tworzyć jakieś porąbane closure dla wpychanej funkcji, można po prostu użyć `let`
```
var funcs = [];
// create a bunch of functions
for (let i = 0; i < 3; i++) { // Note the use of let
    funcs.push(function() {
        console.log(i);
    })
}
// call them
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```

### Const:
- Block scoped, niezmienialny, trzeba go inicjalizować, można constować objekty.
Zatme jeżeli nie planujemy zmieniać danej zmiennej, to używajmy const zamiast let.

### Destrukturyzacja:

```
var rect = { x: 0, y: 10, width: 15, height: 20 };

// Destructuring assignment
var {x, y, width, height} = rect; // Zmienne zapełniamy danymi z obiektu rect
console.log(x, y, width, height); // 0,10,15,20

rect.x = 10;
({x, y, width, height} = rect); // istniejące zmienne zapełniamy danymi z obiektu rect
console.log(x, y, width, height); // 10,10,15,20
```
```
// structure
const obj = {"prop": "value"};

// destructure
const {"prop": zmienna} = obj;
console.log(zmienna === "value"); // true
```
```
var foo = { bar: { bas: 123 } };
var {bar: {bas}} = foo; // Effectively `var bas = foo.bar.bas;`
console.log(foo) //objekt bar z property bas o wartości 123
```
```
var {w, x, ...remaining} = {w: 1, x: 2, y: 3, z: 4};
console.log(w, x, remaining); // 1, 2, {y:3,z:4}

var [x, , ...remaining] = [1, 2, 3, 4];
console.log(x, remaining); // 1, [3,4]
```

I jeszcze taka fajna sztuczka od Filipa:

```
a = { b:4 }
a.b //4
zz = a
zz.b = 10
a.b //10 (referencja)

// WTEM!

zz = {...a}
zz.b = 20
a.b //10
z.b //20

```

### Spread:

Zamiast wrzucać arraya jako kolejne argumenty:
```
function foo(x, y, z) { }
var args = [0, 1, 2];
foo.apply(null, args);
```
Można użyć tzw. spreada:
```
function foo(x, y, z) { }
var args = [0, 1, 2];
foo(...args);
```
Używaliśmy tego też w destructuryazcji.

```
var list = [1, 2];
list = [0, ...list, 4];
console.log(list); // [0,1,2,4]
```

Kolejność jest ważna (To co jest później nadpisuje to co wcześniej):
```
const point2D = {x: 1, y: 2};
const anotherPoint3D = {x: 5, z: 4, ...point2D};
console.log(anotherPoint3D); // {x: 1, y: 2, z: 4}
const yetAnotherPoint3D = {...point2D, x: 5, z: 4}
console.log(yetAnotherPoint3D); // {x: 5, y: 2, z: 4}
```

### In vs Of:

```
var someArray = [9, 2, 5];
for (var item in someArray) {
    console.log(item); // 0,1,2
}
// Natomiast nowe wersje JS pozwalają na:
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item); // 9,2,5
}
```

### Iteratory:
https://basarat.gitbooks.io/typescript/content/docs/iterators.html
Jak to zrozumiem o co biega, to tu wrzucę xD

### Interpolacja stringów:
Używa się do tego backticka (Tam gdzie tylda)

```
var lyrics = 'Never gonna give you up';
var html = `<div>${lyrics}</div>`;

console.log(`2 plus 2 to ${2 + 2)`);

var lyrics = `Never gonna give you up
Never gonna let you down`;
```

### Promise

![alt text](https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png "Jak działa promise")

Mógłbym tutaj dać przykład durnej metody, gdzie dwie różne rzeczy mogą się wywalić i trzeba brać to pod uwagę i są tysiące callbacków, bla bla bla.
Dlatego przechodzę do mięsa:

```
const promise = new Promise((resolve, reject) => {
    // the resolve / reject functions control the fate of the promise
});
```

Następnie `.then` łapie resolve a `.catch` łapie errora.

Przykładowo:
```
const promise = new Promise((resolve, reject) => {
    resolve(123); //Udało się i zwracamy 123
});
promise.then((res) => {
    console.log('I get called:', res === 123); // I get called: true
});
promise.catch((err) => {
    // This is never called
});
const promise = new Promise((resolve, reject) => {
    reject(new Error("Something awful happened")); // Zjebawszy i zwracamy errora.
});
promise.then((res) => {
    // This is never called
});
promise.catch((err) => {
    console.log('I get called:', err.message); // I get called: 'Something awful happened'
});
```

Ogólnie koleś odpierdala niezłe fixy
```
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
    })
```
Zatem najpierw rzucamy errorem, następnie na tej podstawie logujemy resolve (Ale że nie ma resolve, to nie logujemy i to sie nie wywoła - Tak samo jak zwrotka 456), następnie łapiemy nasz error, mówimy o błędzie i zwracamy jak człowiek 123. Na koniec przejmujemy 123 i je wypisujemy.
```
Promise.resolve(123)
    .then((res) => {
        return 456;
    })
    .catch((err) => {
        console.log("HERE"); // never called
    })
```

Zatem - ERROR skacze do `.catch` a resolve do `.then`

Typescript jest na tyle mądry, że ogarnia, że możemy w łańcuchu zwracać kolejno inne typy danych.

Ogarnia również jak sie zachować, jeżeli dostaniemy zwrotkę promise:

```
function iReturnPromiseAfter1Second(): Promise<string> {
    return new Promise((resolve) => {
        setTimeout(() => resolve("Hello world!"), 5000);
    });
}

Promise.resolve(123)
    .then((res) => {
        // res is inferred to be of type `number`
        return iReturnPromiseAfter1Second(); // We are returning `Promise<string>`
    })
    .then((res) => {
        // res is inferred to be of type `string`
        console.log(res); // Hello world!
    });
```

Nasza poprzednio pojebana funkcja, której nie chciało mi się pisać wyglądałaby tak:
```
import fs = require('fs');
function readFileAsync(filename: string): Promise<any> {
    return new Promise((resolve,reject) => {
        fs.readFile(filename,(err,result) => {
            if (err) reject(err);
            else resolve(result);
        });
    });
}
```

Zatem jest to funkcja, która przyjmuje stringa a zwraca Promise od czegokolwiek.
Zwraca nową obietnicę na podstawie odczytania pliku za pomocą fs.readFile - Jeżeli fs.readFile napotka error, to Promise zwróci reject i error a jeżeli się uda, to zwróci resolve i result.

Możemy napisać takiego potworka:
```
// good json file
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // never called
    })

// non-existent json file
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })

// invalid json file
    .then(function () {
        return loadJSONAsync('invalid.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```
Mamy rozważone 3 opcje - Poprawny json, nieistniejący json, invalid json. Odpalamy je po kolei (Tą samą metodą jakbyś się tak przyjrzał) i mamy rozważone wszystkie wypadki, nie odpalamy callbacków po parę razy itp.

Nie musimy koniecznie się łańcuchować, możemy też użyć `Promise.all' jeżeli zależy nam po prostu na wynikach wszystkich promisów z łańcucha.
```
// an async function to simulate loading an item from some server
function loadItem(id: number): Promise<{ id: number }> {
    return new Promise((resolve) => {
        console.log('loading item', id);
        setTimeout(() => { // simulate a server delay
            resolve({ id: id });
        }, 1000);
    });
}

// Wersja kolejkowa
let item1, item2;
loadItem(1)
    .then((res) => {
        item1 = res;
        return loadItem(2);
    })
    .then((res) => {
        item2 = res;
        console.log('done');
    }); // overall time will be around 2s

// Użycie promise all
Promise.all([loadItem(1), loadItem(2)])
    .then((res) => {
        [item1, item2] = res;
        console.log('done');
    }); // overall time will be around 1s
```

Można też zrobić zjebane wyścigi rydwanów za pomocą `Promise.race`:
```
var task1 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 1000, 'Rydwan 1 wygrywa');
});
var task2 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 2000, 'Rydwan 2 wygrywa');
});

Promise.race([task1, task2]).then(function(value) {
  console.log(value); // "Brawo dla rydwana 1"
  // Both resolve, but task1 resolves faster
});
```

### Funkcje generujące:

```
function* generator(){
    console.log('Execution started');
    yield 0;
    console.log('Execution resumed');
    yield 1;
    console.log('Execution resumed');
}

var iterator = generator();
console.log('Starting iteration'); // This will execute before anything in the generator function body executes
console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```
Możemy stworzyć sobie funkcje, a ona się odpala częściowo za każdym odpytaniem (W sumie jak iteratory)

```
function* generator() {
    const bar = yield 'foo'; // bar may be *any* type
    console.log(bar); // bar!
}

const iterator = generator();
// Start execution till we get first yield value
const foo = iterator.next();
console.log(foo.value); // foo
// Resume execution injecting bar
const nextThing = iterator.next('bar');
```

niezły hardkor, ale chyba grokłem - generator zatrzymuje się przy `yield 'foo'`. Jeżeli teraz odpalimy `next.('bar')` to `yield 'foo'` zmieni się w `'bar'`

### A na chuj mi ten generator? (Async Await)

```
function delay(milliseconds: number, count: number): Promise<number> {
    return new Promise<number>(resolve => {
        setTimeout(() => {
            resolve(count);
        }, milliseconds);
    });
}

// async function always returns a Promise
async function dramaticWelcome(): Promise<void> {
    console.log("Hello");

    for (let i = 0; i < 5; i++) {
        // await is converting Promise<number> into number
        const count: number = await delay(500, i);
        console.log(count);
    }

    console.log("World!");
}

dramaticWelcome();
```

A zatem mamy funkcję, która czeka x `milliseconds` a następnie wykonuje zwraca `count`.
W połączeniu z awaitem czekamy aż się wykona (Czekamy a nie puszczamy równolegle!)


### Import/Export

Domyślnie jeżeli mamy dwa pliki:
foo.ts:
```
var foo = 123;
```
oraz bar.ts:
```
var bar = foo;
```
To jest to dozwolone, bo oba są w "globalnym" namespace.

Natomiast możemy skorzystać z export:

foo.ts:
```
export var foo = 123;
```
i teraz bar.ts nie ma do tego dostępu.
Natomiast możemy to zaimportować:
```
import {foo} from "./foo"
var bar = foo;
```

Dodatkowo taki import sprawia, że bar.ts staje się MODUŁEM i nie zaśmieca on globalnego namespace.

## Typescript

Tu Michciu nie dotarł, bo uznał że początek był łatwy :D

### Annotacje
Do wszystkiego (tak jakby) możesz dopisać adnotację tłumaczącą jaki jest to typ:

```
var num: number = 123;
function identity(num: number): number {
    return num;
}
```
Z prymitywnych typów masz:
```
var num: number;
var str: string;
var bool: boolean;
```

Masz też robienie array:
```
var boolArray: boolean[];
```

Są też INTERFEJSY :D

```
interface Name {
    first: string;
    second: string;
}

var name: Name;
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```

Możemy też tworzyć interfejs w biegu przy deklaracji zmiennej:
```
var name: {
    first: string;
    second: string;
};
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```

```
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// Safety!
reversed[0] = '1';     // Error!
reversed = ['1', '2']; // Error!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

### Union

Można zrobić typ, który trzyma jeden z x typów. Rodzielamy je |
```
function formatCommandline(command: string[]|string) {
    var line = '';
    if (typeof command === 'string') {
        line = command.trim();
    } else {
        line = command.join(' ').trim();
    }

    // Do stuff with line: string
}
```

### Intersection

Można też jebnąć typ, który jest złożeniem dwóch:
```
function extend<T, U>(first: T, second: U): T & U {
  return { ...first, ...second };
}

const x = extend({ a: "hello" }, { b: 42 });

// x now has both `a` and `b`
const a = x.a;
const b = x.b;
```

Swoją drogą to to jest bardziej union ale już chooy :p

### Tuple

Jest i krotka:
```
var nameNumber: [string, number];
// Okay
nameNumber = ['Jenny', 8675309];

// Error!
nameNumber = ['Jenny', '867-5309'];
```

### TypeAlias

I to w sumie jest też taki union

```
type StrOrNum = string|number;

// Usage: just like any other notation
var sample: StrOrNum;
sample = 123;
sample = '123';

// Just checking
sample = true; // Error!
```

###Dobudówki interfejsów

```
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;

interface Point {
    z: number;
}

// Your code
myPoint.z; //luz
myPoint.x; //luzzzik!
```
E no WOW :D

### Enumy jako flagi

```
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3
}
```

A tutaj do groknięcia:

```
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
}
type Animal = {
    flags: AnimalFlags
}

function printAnimalAbilities(animal: Animal) {
    var animalFlags = animal.flags;
    if (animalFlags & AnimalFlags.HasClaws) {
        console.log('animal has claws');
    }
    if (animalFlags & AnimalFlags.CanFly) {
        console.log('animal can fly');
    }
    if (animalFlags == AnimalFlags.None) {
        console.log('nothing');
    }
}

let animal: Animal = { flags: AnimalFlags.None };
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws;
printAnimalAbilities(animal); // animal has claws
animal.flags &= ~AnimalFlags.HasClaws;
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws | AnimalFlags.CanFly;
printAnimalAbilities(animal); // animal has claws, animal can fly
```

W tym wypadku
|= dodaje flage
&= oraz ~  ją usuwa
| łączy flagi

### Adnotacje parametrów:

```
function foo(sampleParameter: { bar: number }) { }
```
Nie wiem co mi dajesz, ale ma mieć parametr `bar` będący liczbą.

### Wpychanie typów

```
var foo = {};
foo.bar = 123; // Error: property 'bar' does not exist on `{}`
foo.bas = 'hello'; // Error: property 'bas' does not exist on `{}`
```
ALE
```
interface Foo {
    bar: number;
    bas: string;
}
var foo = {} as Foo;
foo.bar = 123;
foo.bas = 'hello';
```

### Literały

Wygląda mi to na hackowanie, użyłbym enuma, ALE NO SKORO KTOŚ TAK CHCE

```
type CardinalDirection =
    | "North"
    | "East"
    | "South"
    | "West";

function move(distance: number, direction: CardinalDirection) {
    // ...
}

move(1,"North"); // Okay
move(1,"Nurth"); // Error!
```

### Never
```
function foo(x: string | number): boolean {
  if (typeof x === "string") {
    return true;
  } else if (typeof x === "number") {
    return false;
  }

  // Without a never type we would error :
  // - Not all code paths return a value (strict null checks)
  // - Or Unreachable code detected
  // But because TypeScript understands that `fail` function returns `never`
  // It can allow you to call it as you might be using it for runtime safety / exhaustive checks.
  return fail("Unexhaustive!");
}

function fail(message: string): never { throw new Error(message); }
```

Na co to komu xD

Ano na przykład:

```
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else {
        // ERROR : `Circle` is not assignable to `never`
        const _exhaustiveCheck: never = s;
    }
}
```

Teraz jeżeli dorzucimy Shape "Circle" to program nam nie wypierdoli. Zamiast tego dostaniemy infoska, że Circle nie może być neverem - A więc będziemy wiedzieli, że gdzieś coś! Wersja bez tego else byłaby akceptowalna (O zgrozo)

### Retrospective Versioning

Mamy obiekt
```
type DTO = {
  name: string
}
```

Ale osiem lat później odwala nam i stwierdzamy, że `name` to nazwa dla lamusów.
Możemy dodać wersjonowanie takiego obiektu.

```
type DTO = 
| { 
   version: undefined, // version 0
   name: string,
 }
| {
   version: 1,
   firstName: string,
   lastName: string, 
}
// Even later 
| {
    version: 2,
    firstName: string,
    middleName: string,
    lastName: string, 
} 
// So on

function printDTO(dto:DTO) {
  if (dto.version == null) {
      console.log(dto.name);
  } else if (dto.version == 1) {
      console.log(dto.firstName,dto.lastName);
  } else if (dto.version == 2) {
      console.log(dto.firstName, dto.middleName, dto.lastName);
  } else {
      const _exhaustiveCheck: never = dto;
  }
}
```

## Angular

W trakcie wykonywania tutoriala powstał taki klawy kod html:
```
<h2>Products</h2>

<div *ngFor="let product of products">
  <h3>

    <a [title]="product.name + ' details'">
    {{product.name}}
    </a>
    </h3>

    <p *ngIf="product.description">
      Description: {{product.description}}
    </p>

    <button (click)="share()">
      Share
    </button>
    
</div>
```

Oczywiście leży pod nim jeszcze odpowiedni kod css i ts, ale możemy tutaj znaleźć kilka rzeczy:
`*ngFor` - Informujemy, że chcemy powtórzyć coś x razy po pętli. A jakiej? Tej opisanej. A skąd to całe "products"? Otóż jest w pliku `.ts` pod tą stronką.
`[title]` - Property binding, teraz nasz element `<a>` będzie miał właściwość `title` ustawioną na nazwę produktu + `'details'`
'*ngIf' - ten element powwstanie tylko, jeżeli `product.description` będzie truthy.
`{{ }}` - Interpolacja
`()` - Event Binding (`share()` jest metodą w kodzie w `.ts`)


Natomiast plik `.ts` (Innej klasy już) wygląda mniej więcej tak:
```
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-product-alerts',
  templateUrl: './product-alerts.component.html',
  styleUrls: ['./product-alerts.component.css']
})
export class ProductAlertsComponent implements OnInit {
  constructor() { }

  ngOnInit() {
  }

}
```
Zwraca uwagę tzw. dekorator `@Component` - trzyma on metadane o klasie, oraz oznacza ją jako komponent angularowy.
*selector* to nazwa komponentu. Zwyczajowo poprzedzana prefixem `app-`
*templateUrl* to link do htmla
*styleUrls* link do arkuszy styli

### Interpolacja bajerki
```
<h2>{{hero.name | uppercase}} Details</h2>
```

Tak zapisana interpolacja zawiera "interpolation bidding" - uppercase. Wiadomka co robi, warto pamiętać.
Znaczek "|" to tak zwany pipe. Więcej o nim tutaj: https://angular.io/guide/pipes

### Elegancki two-way binding

Przykład stąd: https://stackblitz.com/angular/pxxgmdalqpmb

Potrzebujemy `<input [(ngModel)]="hero.name" placeholder="name"/>`
(Oraz dodatkowo `import { FormsModule } from '@angular/forms';` ale to tak swojo drogo)

Teraz jesteśmy połączeni w dwie strony - Jeżeli coś zmieni się w inpucie, to od razu znajdzie to odwzorowanie w obiekcie i na odwrót.

### Observable

W serwisie:
```
  getHeroes(): Observable<Hero[]> {
    return of(HEROES);
  }
```
Teraz ta metoda zwraca nam HEROES (jest to lista obiektów hero) asynchronicznie.
Możemy ją obserwować w klasie właściwej:
```
getHeroes(): void {
  this.heroService.getHeroes()
      .subscribe(heroes => this.heroes = heroes);
}
```

### entryComponents

W WidgetTableModule:

```
@NgModule({
    declarations: [
cośtam
    ],
    entryComponents: [
        TableMaximizedComponent
    ],
    exports: [
        CustomizationFormColumnsComponent,
        WidgetTableHeaderComponent, //Jeżeli zaimportuję WidgetTableModule gdzie indziej, to mam dostep do tych komponentów
        WidgetTableComponent
    ],
    imports: [
cośtam
    ]
})
export class WidgetTableModule { }
```

Teraz nasz serwis (WidgetTableModule) może "wyrzucić" z siebie TableMaximizedComponent - Czyli w jquery byłby to modal - Okienko nad obecnym oknem.
Exports natomiast sprawiają, że Jeżeli zaimportuję sobie WidgetTableModule, to będę miał dostęp do komponentów z tej listy.
A jak z teog korzystać? Otóż w `Widget-table.component.ts`:

```
private openTableMaximized(): void {
        const dialog = this.dialog.open(TableMaximizedComponent, {
            height: "90%",
            width: "90%",
        });
        const instance = dialog.componentInstance;

        const dataClone = JSON.parse(JSON.stringify(this.data)) as TableDataModel;
        this.initializeData(dataClone);

        instance.dataModel = dataClone.typeSpecificData;
        instance.dataSourceData = this.newGeneratedDataSourceModel;
        instance.errorMessage = this.errorMessage;
    }
```

dialog.componentInstance wydaje się kluczowym momentem.
Co ciekawe można sięsubskrybować do dialogu :D