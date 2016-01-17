# Kapitel 3: Pures Vergnügen mit Pure Functions

## Endlich wieder pur

Eine Sache die wir klar stellen müssen, ist die Idee einer Pure Function.

>Eine Pure Function ist eine Funktion die mit der selben Eingabe immer die gleiche Ausgabe produzieren wird und keinerlei beobachtbaren Seiteneffekt hat.

Nehmen wir `slice` und `splice`. Diese zwei Funktionen machen genau das Gleiche - auf sehr unterschiedliche Weise, also eben doch nicht genau das Gleiche. Wir sagen `slice` ist *pure* weil sie bei jedem Aufruf für jeden Input den gleichen Output liefert - garantiert. `splice` hingegen wird sein Array einmal durchkauen und dann völlig verändert wieder ausspucken, was man deutlich sehen kann.

```js
var xs = [1,2,3,4,5];

// pure
xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]


// impure
xs.splice(0,3);
//=> [1,2,3]

xs.splice(0,3);
//=> [4,5]

xs.splice(0,3);
//=> []
```

In der Funktionalen Programmierung mögen wir so unhandliche Funktionen wie `splice` gar nicht. Denn solche Funktionen *mutieren* Daten. Wir streben nach verlässlichen Funktionen, die jedes mal das gleiche Ergebnis zurück liefern und nicht Funktionen wie `splice` die hinter sich eine Spur der Verwüstung zurück lassen.

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


In der impure Variante hängt `checkAge` von der veränderlichen Variable `minimum` ab um das Ergebnis zu ermitteln. Mit anderen Worten, es hängt vom Systemzustand ab. Das ist sehr ungünstig, da wieder mehr Hirnschmalz benötigt wird um eine externe Umgebung zu berücksichtigen.

Das scheint in diesem Beispiel nicht viel auszumachen. Aber die Abhängigkeit von einem Zustand trägt hauptsächlich zur Systemkomplexität bei(siehe hierzu: http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf). Dieses `checkAge` kann zu unterschiedlichen Ergebnissen führen, abhängig von Faktoren die nichts mit dem Input zu tun haben. Das disqualifiziert die Funktion nicht nur als Pure Function, sie erfordert auch höhere Aufmerksamkeit bei der Analyse der Software.

Die pure Form andererseits ist völlig autark. Ebenso können wir `minimum` unveränderlich machen, was genauso die Reinheit sichert, da der Zustand sich nie ändert. Dafür müssen wir ein Objekt definieren um zu "freezen"

```js
var immutableState = Object.freeze({
  minimum: 21
});
```

## Seiteneffekte beinhalten auch...


Wir wollen diese "Seiteneffeke" nochmal genauer betrachten um unsere Intuition zu schulen. Was sind also diese zweifelsohne ruchlosen *Seiteneffekte* die in der Defintion von *Pure Functions* vorkommen? Mit *Effekt* meinen wir alles was durch die Funktionsverarbeitung passiert, das nichts mit der eigentlichen Berechnung zu tun hat.

Es gibt an sich nichts schlechtes über Effekte zu sagen. Wir verwenden sie ständig auf den kommenden Seiten. Es ist dieser *Seiten...*-Anteil, der problematisch ist. Ein Gewässer alleine ist noch keine Brutstätte für Larven, es ist das *stehende* Gewässer, das die Mückenschwärme hervorbringt. Und ich versichere Dir, dass *Seiten*effekte ein ähnlich ungünstiger Nährboden in Deinen Programmen ist.

>Ein *Seiteneffekt* ist eine Änderung des Systemstatus oder *sichtbare Wechselwirkung* mit der Außenwelt, die während der Berechnung eines Ergebnisses passiert.

Seiteneffekte können, müssen aber nicht, folgende Aspekte beeinhalten:

  * Filesystem verändern.
  * Datenbank Einträge machen
  * Http Aufruf ausführen
  * Mutationen
  * Bildschirm Ausgaben / Logging
  * Usereingaben verarbeiten
  * DOM abfragen
  * System Status einlesen

Diese Liste ist bei weitem nicht vollständig. Jede Interaktion mit der Welt außerhalb einer Funktion ist ein Seiteneffekt, was den Verdacht erregt ob man überhaupt ohne sie praktisch programmieren kann. Die Philosophie Funktionaler Programmierung postuliert, das Seiteneffekte der Hauptgrund für inkorrektes Verhalten sind.

Es ist nicht verboten sie zu nutzen, eher wollen wir sie in einer kontrollierten Weise verwenden. Wir werden lernen , wie das geht, wenn wir in späteren Kapiteln auf Fuktoren und Monaden zu sprechen kommen. Für's erste wollen wir erstmal diese heimtückischen Funktionen von unseren reinen Funktionen aussondern.

Seitenffekte bewirken, daß eine Funktion *impure* wird. Und das macht Sinn: Pure Functions müssen per definition bei gleichem Input immer den gleichen Output liefern. Das ist nicht gewährleistet, wenn auf Dinge Bezug genommnen wird, die außerhalb des lokalen Function Scope sind.

Wir wollen noch genauer untersuchen, warum es so wichtig ist, dass bei gleichem Input der gleiche Output zu erhalten. Zieh Dich warm an, wir machen jetzt 8. Klasse Mathematik.

## 8. Klasse Mathe

Von mathisfun.com:

> Eine Funktion ist eine spezielle Beziehung zwischen Werten:
> Jeder ihrer Eingabewerte liefert genau einen Ausgabewert

Mit anderen Worten, sie ist nur eine Relation zwischen zwei Werten: dem Input und dem Output. Obwohl jede Eingabe genau eine Ausgabe hat, heißt das nicht, dass die Ausgabe eindeutig pro Eingabe sein muss. Das unten stehende Diagram zeigt eine absolut valide Funktion von `x` nach `y`;

<img src="images/function-sets.gif" />(http://www.mathsisfun.com/sets/function.html)

Im Gegensatz dazu zeigt das folgende Diagram eine Beziehung, die *keine* Funktion darstellt, da der Eingabewert `5` mehrere Ausgaben aufweist:

<img src="images/relation-not-function.gif" />(http://www.mathsisfun.com/sets/function.html)

Funktionen können als eine Menge von Paaren beschrieben werden, mit den Koordinaten (Input, Output): `[(1,2), (3,6), (5,10)]` (Es sieht so aus, als würde diese Funktion Ihre Eingaben verdoppeln).

Oder vielleicht als Tabelle
<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

Oder sogar ein Graph mit `x` als Eingabe und `y` als Ausgabe:

<img src="images/fn_graph.png" width="300" height="300" />


Implementierungsdetails spielen keine Rolle wenn der Input den Output diktiert. Da Funktionen einfache Abbildungen von Eingaben auf Ausgaben sind, könnte man auch einfach Objekt Literale definieren und sie mit `[]` statt mit `()` aufrufen.

```js
var toLowerCase = {"A":"a", "B": "b", "C": "c", "D": "d", "E": "e", "D": "d"};

toLowerCase["C"];
//=> "c"

var isPrime = {1:false, 2: true, 3: true, 4: false, 5: true, 6:false};

isPrime[3];
//=> true
```

Natürlich berechnet man lieber ein Ergebnis, als das man alle ausschreibt. Aber so kann man Funktionen auch betrachten. (Du wirst jetzt sicher denken "Und was ist mit Funktionen die mehrere Argumente benötigen?" Sicher, das passt noch nicht ganz in die mathematische Betrachtungsweise. Für's erste können wir mehrere Parameter als einen Eingabeparameter in Form eines Arrays oder auch als das `arguments` Objekt betrachten. Wenn wir *currying* kennen lernen werden, wird deutlich werden, wie wir Funktionen modellieren können, dass sie der mathematischen Definition gerecht werden.)

Hier kommt die dramaturgische Wendung: Pure Functions *sind* mathematische Funktionen und sie dreht sich alles in Funktionaler Programmierung. Mit diesen kleinen Engeln zu programmieren kann rießige Vorteile bringen. Untersuchen wir ein paar Gründe warum es so wichtig ist die Purity sicher zu stellen.

## Ein Plädoyer für Purity

### Cacheable

Zum Anfang, Pure Functions können immer anhand des Inputs gecacht werden. Dies macht man typischerweise mit einer Technik namens memoization:

```js
var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4);
//=> 16

squareNumber(4); // liefert das gecachte Ergebnis für die Eingabe 4
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // liefert das gecachte Ergebnis für die Eingabe 5
//=> 25
```

Hier mal eine kleine Implementierung, obwohl es jede Menge robustere Versionen gibt...

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

Was man sich merken sollte ist, dass man Impure Functions in Pure Functions transformieren kann, in dem man ihre Ausführung verzögert.

```js
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

Interessant ist, dass wir hier eigentlich den http-Aufruf nicht ausführen - stattdessen liefern wir eine Funktion zurück die das bei ihrem Aufruf erledigt. Diese Funktion ist Pure weil sie bei gegebenem Input immer das gleiche Ergebnis abliefert: Die Funktion, die den entsprechenden http Aufruf mit den gegebenen `url` und `params` durchführt.

Unsere `memoize` Funktion arbeitet wunderbar, obwohl sie nicht das Ergebnis des http Aufrufs cacht, sondern sie cacht die generierte Funktion.

Das ist noch nicht wirklich nützlich, aber wir werden bald mehr Tricks kennen lernen, die den Nutzen aufzeigen werden. Was wir uns merken sollten ist, dass wir jede Funktion cachen können, egal wie destruktiv sie sein mag.

### Portierbar, Selbsterklärend

Pure Functions sind komplett unabhängig. Alles was die Funktion benötigt, wird ihr auf einem Silbertablett übergeben. Lass das mal auf Dich wirken... Wie könnte man davon profitiern? Zum Anfang: die Abhängikeiten einer Funktion sind explizit und deshalb leichter zu erkennen - kein schräges Gedöns unter der Haube.

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

Das Beispiel hier demonstriert, dass pure Functions ehrlich bezüglich iherer Abhängigkeiten sind und uns so auch genau sagen, was sie tun. Alleine aus der Signatur erkennen wir dass wir ein `Db`, ein `Email`und `attrs` nutzen was schon mal das Wichtigste klar stellt.

Wir werden lernen, Funktionen genauso Pure zu machen, ohne die Ausführung zu verzögern. Aber klar sollte sein, dass die Pure Form viel informativer ist, als sein hinterlistiger unreiner counterpart, der Gott weiß was tut.

Außerdem fällt auf, dass wir gezwungen sind Abhängigkeiten zu "injecten", oder sie als Argumente zu übergeben. Das macht unsere App viel flexibler, weil wir unsere Datenbank, unseren Mail Client oder was auch immer parametrisiert haben. (Keine Angst, wir werden einen Weg finden, das weit weniger umständlich zu machen als es klingt.) Vielleicht werden wir irgendwann eine andere Db nutzen die wir unserer Funktion übergeben wollen. Vielleicht schreiben wir irgendwann eine neue Applikation in der wir diese verlässliche Funktion wiederverwenden wollen, an die wir schlicht das `Db` und `Email` übergeben, dass wir zu dem Zeitpunkt verwenden.

Im JavaScript Umfeld kann Portabilität das Serialisieren und über einen Socket senden bedeuten. Es kann bedeuten unseren ganzen App Code in WebWorkern laufen zu lassen. Portierbarkeit ist ein wertvolles Gut.

Im Gegensatz zur imperativen Programmierung, in der typische Methoden und Prozeduren tief mit ihrer Umgebung durch den Zustand, den Abhängigkeiten und verfügbaren Effekten verwurzelt sind, können Pure Functions überall laufen wo es uns beliebt.

Wann hast Du das letzte Mal eine Methode in eine neue App kopiert? Eines meiner Lieblingszitate kommt von dem Erfinder von Erlang, Joe Armstrong: "Das Problem mit objektorientierten Sprachen ist, dass sie implizit immer ihre Umgebung mit sich herumtragen. Wenn Du eine Banane willst, bekommst Du statt dessen den Gorilla der die Banane hält... und den ganzen Dschungel".

### Testbarkeit

Als nächstes stellen wir fest, dass Pure Functions viel leichter zu testen sind. Wir müssen kein "echtes" Payment Gateway oder Setup weg mocken und den Zustand der Welt nach jedem Test prüfen. Wir geben der Funktion einfach Input und überprüfen den Output.


Genaugenommen beobachten wir gerade, wie die funktionale Gemeinschaft neue Testtools an den Start bringt, die unsere Funktionen mit generiertem Input bombardieren und sicherstellen dass der Output bestimmte Eigenschaften erfüllt. Es würde den Rahmen des Buches sprengen, aber ich empfehle sehr nach *Quickcheck* zu suchen - einem Test Tool das für rein Funktionale Umgebungen zugeschnitten ist.

### Nachvollziehbar

Viele glauben, der größte Gewinn durch das Arbeiten mit Pure Functions ist *referenzielle Transparenz*. Ein Stück Code ist referenziell transparent, wenn er durch seinen evaluierten Wert ersetzt werden kann ohne das Verhalten des Programms zu verändern.

Da Pure Functions immer den gleichen Output zurückliefern wenn sie den gleichen Input erhalten, können wir uns darauf verlassen, dass sie immer die selben Ergebnisse zurück liefern. Damit ist die referentielle Transparenz sichergestellt ist. Schauen wir uns ein Beispiel an.

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

`decrementHP`, `isSameTeam` und `punch` sind alle Pure und daher referentiell transparent. Wir können eine Technik namens *equational reasoning* anwenden, bei der ein "Istgleich ein Istgleich" ersetzt um über den Code nachzudenken. Man könnte beinahe sagen, dass wir dann den Code selbst auswerten können ohne die Probleme der programmatischen Auswertung zu haben. Referentielle Transparenz zu nutzen heißt mit dem Code zu spielen.

Zu erst werden wir die `isSameTeam` Funktion inlinen.

```js
var punch = function(player, target) {
  if (player.get("team") === target.get("team")) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Da unsere Daten unveränderlich sind, können wir einfach die Teams mit ihrem eigentlichen Wert ersetzen

```js
var punch = function(player, target) {
  if ("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Wir sehen, dass sich in diesem Fall false ergibt, daher können wir den kompletten if-Zweig entfernen.

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

Und wenn wir `decrementHP` inlinen, sehen wir dass in diesem Fall aus punch ein Aufruf wird, der `hp`um 1 verringert.

```js
var punch = function(player, target) {
  return target.set("hp", target.get("hp")-1);
};
```

Diese Fähigkeit so Code zu durchdenken ist grandios für Refactoring und das Codeverständnis im Allgemeinen. Genaugenommen haben wir diese Technik genutzt um unser Möwenprogramm zu refactoren. Wir nutzten equational reasoning um die Eigenschaft  von Addition und Multiplikation zu extrahieren. Tatsächlich werden wir diese Techniken im ganzen Buch verwenden.

### Paralleler Code

Zuletzt hab ich noch ein Sahnehäubchen, wir können jede Pure Function parallel laufen lassen, weil sie keinen Speicher aufrufen muss der noch von anderen Genutzt wird und kann per Definition keine Raceconditions wegen irgendwelcher Seiteneffekte produzieren.

Das ist besonders in einer Serverumgebung mit Threads möglich, aber auch im Browser mit WebWorkern. Trotzdem vermeiden dies die meisten, weil sie die Komplexität bei Arbeiten mit Impure Functions befürchten.


## Zusammenfassung

Wir haben gesehen was pure Functions sind und warum wir, als funktionale Programmierer, sie für die Stiefel des gestiefelten Katers halten. Ab hier streben wir danach alle unsere Funktionen Pure zu schreiben. Wir werden ein paar extra Tools kennen lernen um uns zu helfen. Aber in der Zwischenzeit werden wir die Impure Functions vom Rest des Codes extrahieren.

Programme mit Pure Functions zu schreiben ist ein bisschen umständlich ohne extra Tools in unserem Werkzeugkasten. Wir müssen mit Daten jonglieren und haben das Verbot Zustände zu verwenden ganz zu schweigen von Effekten. Wie tickt jemand, der so masochistische Programme schreibt? Wir brauchen ein neues Tool namens curry.

[Kapitel 4: Currying](ch4-de.md)
