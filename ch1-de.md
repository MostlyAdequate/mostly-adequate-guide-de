# Kapitel 1: Worum geht''s uns eigentlich?
## Vorstellung

Hi! Ich bin Professor Franklin Risby. Es freut mich Deine Bekanntschaft zu machen. Wir werden zusammen ein wenig Zeit verbringen, da ich Dir ein wenig über funktionale Programmierung beibringen soll. Aber genug von mir, was ist mir Dir? Ich hoffe Du kennst Dich mit JavaScript aus, hast ein klitzekleines bisschen Erfahrung mit Objektorientierung, und fühlst Dich als Profiprogrammierer. Du brauchst keinen Doktor in Insektenkunde, Du solltest nur wissen, wie Du Bugs finden und auslöschen kannst.

Ich werde keinerlei Vorwissen über funktionale Programmierung voraussetzen, denn wir beide wissen was passiert wenn man etwas voraussetzt. Allerdings gehe ich davon aus, daß Du in einige unvorteilhafte Situationen erlebt hast, die sich aus der Verwendung von veränderlichen Variablen , uneingeschränkten Seiteneffekten und skrupellosem Design ergeben. Jetzt, da wir uns nun ordnungsgemäß vorgestellt haben, sollten wir keine Zeit verlieren.

Dieses Kapitel soll Dir ein Gefühl dafür geben, worum es bei funktionaler Programmierung geht. Wir brauchen eine gmeinsame Vorstellung davon, was ein Programm *funktional* macht oder wir werden ziellos vor uns hin tippern um Objekte um jeden Preis zu meiden - ein wahrlich ungeschicktes Unterfangen. Unser Code muss eine klare Richtung aufweisen, damit wir auch dann noch auf dem rechten Pfad bleiben, wenn uns die Brotkrümel ausgehen.

Nun gibt es eine Menge genereller Programmierprinzipien - diverse akronyme Glaubenssätze, die uns durch die dunklen Gefilde einer Applikation leiten: DRY (dont''t repeat yourself / Wiederhole Dich nicht). YAGNI (ya ain''t gonna need it / Du brauchst es eh nicht), lose Koplung, hohe Kohäsion, das Principle of least Surprise, das Singe-Responsibility-Prinzip, usw. und so fort.

Ich werde jetzt nicht auf jeden Ratschlag eingehen, den ich über die Jahre gehört habe... der Punkt ist, dass sie alle der funktionalen Denke gerecht werden, obwohl sie sich nur tangential unserem Ziel nähern. Bevor wir noch tiefer einsteigen, möchte ich von Dir, dass Du ist die Absicht hinter unseren ersten funktionalen Gehversuchen verstehst; unser funktionaler Nexus.

<!--BREAK-->

## Eine kleine Heranführung

Lass uns mit einem Hauch Wahnsinn beginnen. Hier ist eine Möwen Applikation. Wenn sich mehrere zwei Möwenschwärme verbinden entseht eine größerer Schwarm (engl: flock). Und wenn sie sich paaren, erhöht sich ihre Anzahl um die Anzahl der Paarungspartner. Der folgende Code versucht kein guter objektorientierter Code zu sein, sondern soll vielmehr verdeutlichen, welche Gefahren in unserem modernen Zuweisungs basierten Ansatz lauern. Aufgepasst:

```js
var Flock = function(n) {
  this.seagulls = n;
};

Flock.prototype.conjoin = function(other) {
  this.seagulls += other.seagulls;
  return this;
};

Flock.prototype.breed = function(other) {
  this.seagulls = this.seagulls * other.seagulls;
  return this;
};

var flock_a = new Flock(4);
var flock_b = new Flock(2);
var flock_c = new Flock(0);

var result = flock_a.conjoin(flock_c)
    .breed(flock_b).conjoin(flock_a.breed(flock_b)).seagulls;
//=> 32
```

Wer um alles in der Welt würde so eine garstige Abscheulichkeit schreiben? Es ist unnötig schwierig die Veränderung des internen Zustands im Kopf zu behalten. Ganz zu schweigen davon, dass das Ergebnis falsch ist! Es müsste 16 herauskommen, aber `flock_a` wird ja zichfach überschrieben.

Wenn Du dieses Programm nicht verstehst, ist das okay... ich verstehe es auch nicht. Der Punkt ist, dass es schwer fällt, veränderlichen Variablen zu folgen, selbst in so einem kleinen Beispiel.

Versuchen wir es nochmal mit einem etwas funktionaleren Ansatz:

```js
var conjoin = function(flock_x, flock_y) { return flock_x + flock_y };
var breed = function(flock_x, flock_y) { return flock_x * flock_y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = conjoin(
  breed(flock_b, conjoin(flock_a, flock_c)), breed(flock_a, flock_b)
);
//=>16
```

Diesmal haben wir das richtige Ergebnis erhalten. Der Code ist viel kürzer. Die Verschachtelung von Funktionen ist etwas verwirrend (wir werden Kapitel 5 darauf zurück kommen). Es ist besser, aber wir sollten etwas genauer hinsehen. Es ist vorteilhaft, das Kind beim Namen zu nennen. Hätten wir das getan, dann hätten wir erkannt, dass wir es hier mit simpler Addition ('conjoin') und Multiplikation ('breed') zu tun haben.

Es ist nichts besonderes an diesen Funktionen, außer ihrer Namen. Lass uns unsere Funktionen umbenennen, um ihren wahren Charakter zu enthüllen.

```js
var add = function(x, y) { return x + y };
var multiply = function(x, y) { return x * y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = add(
  multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b)
);
//=>16
```
Und damit erlangen wir das Wissen unserer Vorväter:

```js
// Assoziativ-Gesetz
add(add(x, y), z) == add(x, add(y, z));

// Kommutativ-Gesetz
add(x, y) == add(y, x);

// Neutrales Element
add(x, 0) == x;

// Distributiv-Gesetz
multiply(x, add(y,z)) == add(multiply(x, y), multiply(x, z));
```

Ach ja richtig, diese althergebrachten mathematischen Gesätze könnten hilfreich sein. Mach Dir keine Sorgen, wenn Du sie nicht aus dem FF runterspulen kannst. Für viele von uns ist es ne ganze Weile her, seit wir uns damit befasst haben. Mal sehen, ob wir diese Eigenschaften nutzen können, um unser kleines Möwenprogramm zu vereinfachen.

```js
// Original Zeile
add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));

// Da flock_c dem neutralen Element entspricht, können wir uns eine Addition sparen
// (add(flock_a, flock_c) == flock_a)
add(multiply(flock_b, flock_a), multiply(flock_a, flock_b));

// Durch das Distributivgesetz erhalten wir unser Ergebnis
multiply(flock_b, add(flock_a, flock_a));
```

Brilliant! Wir mussten nicht einen Strich Anpassungscode schreiben, außer den Aufruf unser Funktion. Der Volständigkeit halber haben wir hier `add` und `multiply` nochmal definiert, aber es gibt keinen Grund sie zu schreiben - es gibt sicherlich ein `add` und ein `multiply` das wir aus einer existierenden Lib verwenden können.

Du wirst jetzt denken "Da hat er es sich ja einfach gemacht, so ein mathe-lastiges Beispiel voranzustellen". Oder: "Echte Programme sind nicht so einfach und können nicht auf diese Weise verstanden werden". Ich habe dieses Beispiel gewählt, weil die meisten von uns wohl mit Addition und Multiplikation vertraut sind, so dass es einfach sein würde zu verstehen, wie uns Mathematik hier helfen kann.

Geb nicht auf - wir werden überall im Buch Kathegorientheorie, Mengenlehre und das Lambda-Kalkül einstreuen, um "Echte Welt Beispiele" zu schreiben, die genauso elegant gelöst sind wie unser Möwenschwarm-Beispiel. Du brauchst auch kein Mathematiker zu sein. Es wird sich anfühlen wie die Verwendung eines Frameworks oder einer API.

Es mag überraschen, dass man vollständige alltägliche Applikationen genauso funktional schreiben kann wie oben. Programme die Lesequalität haben. Programme, die prägnant und trotzdem leicht verständlich sind. Progamme, die nicht das Rad immer wieder neu erfinden. Regelfreiheit ist gut, wenn man kriminel ist. aber in diesem Buch wollen wir die Gesetze der Mathematik anerkennen und ihnen gehorchen.

Wir möchten die Theorie nutzen, in der alle Teile so gut zusammen passen. Wir wollen unser spezifisches Problem als generisch, kombinierbare Häppchen darstellen um dann ihre Eigenschaften für unseren eigennützigen Vorteil auszuschlachten. Es wird etwas mehr Disziplin nötig sein, als der "alles geht" Ansatz imperativer Programmierung (wir werden später in diesem Buch noch genau definieren, was imperativ bedeutet. Im Momment heißt das alles was nicht Funktionale Programmierung ist), aber Du wirst staunen, welchen Vorteil das Arbeiten in einem prinzipientreuen, mathematischen Rahmen bringt.

Wir haben schon einen Funken unseres funktionalen Polarsterns gesehen, aber wir müssen erst ein paar konkrete Konzepte verstehen bevor wir unsere Reise wirklich beginnen können.

[Chapter 2: First Class Functions](ch2.md)
