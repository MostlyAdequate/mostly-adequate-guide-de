# Chapter 3: Pures Vergnügen mit Pure Functions

## Endlich wieder pur

Eine Sache die wir klar stellen müssen, ist die Idee einer reinen Funktion (pure function).

>Eine pure function ist eine Funktion die mit der selben Eingabe immer die gleiche Ausgabe produzieren wird und keinerlei beobachtbaren Seiteneffekt hat.

Nehmen wir `slice` und `splice`. Dies sind zwei Funktionen, die genau das gleiche tun - auf sehr unterschiedliche Weise, aber doch eben doch das gleiche. Wir sagen `slice` ist *pur* weil sie für jedes mal für jeden input den gleichen output liefert - garantiert. `splice` hingegen wird sein array einmal durchkauen und dann völlig verändert wieder ausspucken, was ein sichtbarer Effekt ist.

```js
var xs = [1,2,3,4,5];

// pur
xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]


// nicht pur
xs.splice(0,3);
//=> [1,2,3]

xs.splice(0,3);
//=> [4,5]

xs.splice(0,3);
//=> []
```

In der Funktionalen Programmierung mögen wir so unhandliche Funktionen wie `splice` gar nicht, die Daten *mutieren*. Wir streben nach verlässlichen Funktionen, die jedes mal das gleiche Ergebnis zurück liefern und nicht Funktionen wie `splice` die hinter sich eine Spur der Verwüstung zurück lassen.

Schauen wir uns noch ein anderes Beispiel an.

```js
// nicht pur
var minimum = 21;

var checkAge = function(age) {
  return age >= minimum;
};



// pur
var checkAge = function(age) {
  var minimum = 21;
  return age >= minimum;
};
```


In der unreinen Variante hängt `checkAge` von der veränderlichen Variable `minimum` ab um das Ergebnis zu ermitteln. Mit anderen Worten, es hängt vom Systemzustand ab, was enttäuscht, da wieder mehr Hirnschmalz benötigt wird um eine externe Umgebung zuberücksichtigen.

Das scheint in diesem Beispiel nicht viel auszumachen, aber die Abhängigkeit von einem Zustand tragt mit am meisten zur Systemkoplexität bei(http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf). Dieses `checkAge` kann zu unterschiedlichen Ergebnissen führen, abhängig von Faktoren die nichts mit dem Input zu tun haben. Das disqualifiziert die Funktion nicht nur als pur Function, sie erfordert auch höhere Aufmerksamkeit bei der Analyse der Software.

Die reine Form andererseits ist völlig autark. Genauso können wir `minimum` unveränderlich machen, was ebenso die Reinheit sichert, da der Zustand sich nie ändert. Dafür müssen wir ein Objekt definieren um zu "freezen"

```js
var immutableState = Object.freeze({
  minimum: 21
});
```

## Seiteneffekte beinhalten auch...


Wir wollen diese "Seiteneffeke" nochmal ansehen um unsere Intuition zu verbessern. Was sind also diese zweifelsohne ruchlosen *Seiteneffekte* die in der Defintion von *reinen Funktionen* vorkommen? Mit *Effekt* meinen wir alles was durch die Funktionsverarbeitung passieren kann, das nichts mit der eigentlichen Berechnung zu tun hat.

Es gibt nichts intrinsisch schlechtes über Effekte zu sagen. Wir verwenden sie ständig auf den kommenden Seiten. Es ist dieses *Seiten...* teil, der negativ konnotîert ist. Ein Gewässer alleine ist noch kein Inkubator für Larven, es ist der *stehende* Teil der die Mückenschwärem hervorbringt. Und ich versichere Dir,  dass *Seiten*effekte ein ähnlicher Nährboden in Deinen Programmen ist.

>Ein *Seiteneffekt* ist eine Änderung des Systemstatus oder *sichtbare Auswirkung* mit der Außenwelt, die während der Berechnung eines Ergebnisses passiert.

Seiteneffekte können, müssen aber nicht, folgende Aspekte beeinhalten:

  * Filesystem verändern.
  * Datenbank Einträge machen
  * Http Aufruf ausführen
  * Mutationen
  * Bildschirm Ausgaben / Logging
  * Usereingaben verarbeiten
  * DOM abfragen
  * System status einlesen

Diese Liste ist bei weitem nicht vollständig. Jede Interaktion mit der Welt außerhalb einer Funktion ist ein Seiteneffekt, was den Verdacht erregt ob man überhaupt ohne sie praktisch programmieren kann. Die Philosophie Funktionaler Programmierung postuliert, das Seiteneffekte der Hauptgrund für inkorrektes Verhalten sind.

Es ist nicht verboten sie zu nutzen, eher wollen wir sie in einer kontrollierten Weise verwenden. Wir werden lernen , wie das geht, wenn wir in späteren Kapiteln auf Fuktoren und Monaden zu sprechen kommen. Für's erste wollen wir erstmal diese heimtückischen Funktionen von unseren reinen Funktionen aussondern.

Seiteffekte schließen eine Funktion davon aus *rein* zu sein. Und das macht Sinn: reine Funktionen müssen per definition bei gleichem input immer den gleichen output liefer, was nicht gewährleistet ist, wenn auf Dinge bezug genommne wird, die außerhalb der lokalen Funktion sind.

Lass uns genauer untersuchen, warum bei gleichem input der gleiche output so wichtig ist. Schlag den Kragen auf, wir machne jetzt 8. Klasse Mathematik.

## 8. Klasse Mathe

Von mathisfun.com:

> Eine Funktion ist eine spezielle Beziehung zwischen Werten:
> Jeder ihrer Eingabewerte liefert genau einen Ausgabewert

Mit anderen Worten, sie ist nur eine Relation zwisch zwei Werten: dem Input und dem Output. Obwohl jede Eingabe genau eine Ausgabe hat, heißt das nicht, dass die Ausgabe eindeutig pro Eingabe sein muss.
In other words, it's just a relation between two values: the input and the output. Though each input has exactly one output, that output doesn't necessarily have to be unique per input. Below shows a diagram of a perfectly valid function from `x` to `y`;

<img src="images/function-sets.gif" />(http://www.mathsisfun.com/sets/function.html)

To contrast, the following diagram shows a relation that is *not* a function since the input value `5` points to several outputs:

<img src="images/relation-not-function.gif" />(http://www.mathsisfun.com/sets/function.html)

Functions can be described as a set of pairs with the position (input, output): `[(1,2), (3,6), (5,10)]` (It appears this function doubles its input).

Or perhaps a table:
<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

Or even as a graph with `x` as the input and `y` as the output:

<img src="images/fn_graph.png" width="300" height="300" />


There's no need for implementation details if the input dictates the output. Since functions are simply mappings of input to output, one could simply jot down object literals and run them with `[]` instead of `()`.

```js
var toLowerCase = {"A":"a", "B": "b", "C": "c", "D": "d", "E": "e", "D": "d"};

toLowerCase["C"];
//=> "c"

var isPrime = {1:false, 2: true, 3: true, 4: false, 5: true, 6:false};

isPrime[3];
//=> true
```

Of course, you might want to calculate instead of hand writing things out, but this illustrates a different way to think about functions. (You may be thinking "what about functions with multiple arguments?". Indeed, that presents a bit of an inconvenience when thinking in terms of mathematics. For now, we can bundle them up in an array or just think of the `arguments` object as the input. When we learn about *currying*, we'll see how we can directly model the mathematical definition of a function.)

Here comes the dramatic reveal: Pure functions *are* mathematical functions and they're what functional programming is all about. Programming with these little angels can provide huge benefits. Let's look at some reasons why we're willing to go to great lengths to preserve purity.

## The case for purity

### Cacheable

For starters, pure functions can always be cached by input. This is typically done using a technique called memoization:

```js
var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4);
//=> 16

squareNumber(4); // returns cache for input 4
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // returns cache for input 5
//=> 25
```

Here is a simplified implementation, though there are plenty of more robust versions available.

```js
var memoize = function(f) {
  var cache = {};

  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};
```

Something to note is that you can transform some impure functions into pure ones by delaying evaluation:

```js
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

The interesting thing here is that we don't actually make the http call - we instead return a function that will do so when called. This function is pure because it will always return the same output given the same input: the function that will make the particular http call given the `url` and `params`.

Our `memoize` function works just fine, though it doesn't cache the results of the http call, rather it caches the generated function.

This is not very useful yet, but we'll soon learn some tricks that will make it so. The takeaway is that we can cache every function no matter how destructive they seem.

### Portable / Self-Documenting

Pure functions are completely self contained. Everything the function needs is handed to it on a silver platter. Ponder this for a moment... How might this be beneficial? For starters, a function's dependencies are explicit and therefore easier to see and understand - no funny business going on under the hood.

```js
//impure
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};

//pure
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};
```

The example here demonstrates that the pure function must be honest about its dependencies and, as such, tell us exactly what it's up to. Just from its signature, we know that it will use a `Db`, `Email`, and `attrs` which should be telling to say the least.

We'll learn how to make functions like this pure without merely deferring evaluation, but the point should be clear that the pure form is much more informative than its sneaky impure counterpart which is up to God knows what.

Something else to notice is that we're forced to "inject" dependencies, or pass them in as arguments, which makes our app much more flexible because we've parameterized our database or mail client or what have you (don't worry, we'll see a way to make this less tedious than it sounds). Should we choose to use a different Db we need only to call our function with it. Should we find ourselves writing a new application in which we'd like to reuse this reliable function, we simply give this function whatever `Db` and `Email` we have at the time.

In a JavaScript setting, portability could mean serializing and sending functions over a socket. It could mean running all our app code in web workers. Portability is a powerful trait.

Contrary to "typical" methods and procedures in imperative programming rooted deep in their environment via state, dependencies, and available effects, pure functions can be run anywhere our hearts desire.

When was the last time you copied a method into a new app? One of my favorite quotes comes from Erlang creator, Joe Armstrong: "The problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana... and the entire jungle".

### Testable

Next, we come to realize pure functions make testing much easier. We don't have to mock a "real" payment gateway or setup and assert the state of the world after each test. We simply give the function input and assert output.

In fact, we find the functional community pioneering new test tools that can blast our functions with generated input and assert that properties hold on the output. It's beyond the scope of this book, but I strongly encourage you to search for and try *Quickcheck* - a testing tool that is tailored for a purely functional environment.

### Reasonable

Many believe the biggest win when working with pure functions is *referential transparency*. A spot of code is referentially transparent when it can be substituted for its evaluated value without changing the behavior of the program.

Since pure functions always return the same output given the same input, we can rely on them to always return the same results and thus preserve referential transparency. Let's see an example.

```js

var Immutable = require("immutable");

var decrementHP = function(player) {
  return player.set("hp", player.get("hp")-1);
};

var isSameTeam = function(player1, player2) {
  return player1.get("team") === player2.get("team");
};

var punch = function(player, target) {
  if (isSameTeam(player, target)) {
    return target;
  } else {
    return decrementHP(target);
  }
};

var jobe = Immutable.Map({name:"Jobe", hp:20, team: "red"});
var michael = Immutable.Map({name:"Michael", hp:20, team: "green"});

punch(jobe, michael);
//=> Immutable.Map({name:"Michael", hp:19, team: "green"})
```

`decrementHP`, `isSameTeam` and `punch` are all pure and therefore referentially transparent. We can use a technique called *equational reasoning* wherein one substitutes "equals for equals" to reason about code. It's a bit like manually evaluating the code without taking into account the quirks of programmatic evaluation. Using referential transparency, let's play with this code a bit.

First we'll inline the function `isSameTeam`.

```js
var punch = function(player, target) {
  if (player.get("team") === target.get("team")) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Since our data is immutable, we can simply replace the teams with their actual value

```js
var punch = function(player, target) {
  if ("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

We see that it is false in this case so we can remove the entire if branch

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

And if we inline `decrementHP`, we see that, in this case, punch becomes a call to decrement the `hp` by 1.

```js
var punch = function(player, target) {
  return target.set("hp", target.get("hp")-1);
};
```

This ability to reason about code is terrific for refactoring and understanding code in general. In fact, we used this technique to refactor our flock of seagulls program. We used equational reasoning to harness the properties of addition and multiplication. Indeed, we'll be using these techniques throughout the book.

### Parallel Code

Finally, and here's the coup de grâce, we can run any pure function in parallel since it does not need access to shared memory and it cannot, by definition, have a race condition due to some side effect.

This is very much possible in a server side js environment with threads as well as in the browser with web workers though current culture seems to avoid it due to complexity when dealing with impure functions.


## In Summary

We've seen what pure functions are and why we, as functional programmers, believe they are the cat's evening wear. From this point on, we'll strive to write all our functions in a pure way. We'll require some extra tools to help us do so, but in the meantime, we'll try to separate the impure functions from the rest of the pure code.

Writing programs with pure functions is a tad laborious without some extra tools in our belt. We have to juggle data by passing arguments all over the place, we're forbidden to use state, not to mention effects. How does one go about writing these masochistic programs? Let's acquire a new tool called curry.

[Chapter 4: Currying](ch4.md)
