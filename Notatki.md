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

## TYPESCRIPT

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