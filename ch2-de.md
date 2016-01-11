# Kapitel 2: Funktionen Erster Klasse

## Ein kurzer Rückblick
Wenn wir sagen, Funktionen sind "Erster Klasse" (First Class Functions) meinen wir, dass sie wie alle anderen Daten sind... also normaler Klasse (ähhh Trainer?). Wir können Funktionen wie jeden anderen Datentyp verwenden und es gibt nichts Besonderes an ihnen - sie können in Arrays gespeichert werden, herumgereicht werden, Variablen zugewiesen werden, was immer Du willst.

Das sind JavaScript Grundlagen. Aber es sollte erwähnt werden, dass eine kurze Suche auf Github das kollektive Vermeiden oder viemehr die weitverbreitete Ignoranz dieses Konzepts aufzeigt. Sollen wir uns ein scherzhaftes Beispiel ansehen? Ja sollten wir.

```js
var hi = function(name){
  return "Hi " + name;
};

var greeting = function(name) {
  return hi(name);
};
```

Hier ist der Funktion Wrapper um `hi`in `greeting` völlig unnötig. Warum? Weil Funktionen in JavaScript *aufrufbar* sind. Wenn man `hi` mit `()` am Ende aufruft, wird die Funktion ausgeführt und gibt einen Wert zurück. Fehlen aber die Klammern, wird lediglich die Funktion, die in der Variable gespeichert ist, zurück gegeben. Nur um sicher zu gehen, noch kurz das hier... :

```js
hi;
// function(name){
//  return "Hi " + name
// }

hi("jonas");
// "Hi jonas"
```

Weil `greeting` also nichts anderes tut, als `hi`mit dem selben Argument aufzurufen, können wir genauso gut schreiben:

```js
var greeting = hi;


greeting("times");
// "Hi times"
```

Mit anderen Worten, `hi` ist bereits eine Funktion, die ein Argument erwartet, warum sollten wir eine weitere Funktion darum packen, die schlicht `hi` mit dem einen Argument aufruft? Es macht einfach keinen Sinn. Das wäre so, als zöge man den schwersten Parka im heißesten Juli an, nur um die Luft zu verdrängen und einen gekühlten Lolly zu bestellen.

Es ist gerade zu widerlich verbos und, wie könnte es anders sein, ein schlechter Stil eine Funktion mit einer anderen Funktion zu umgeben, nur um ihre Verarbeitung zu verzögern. (Wir werden noch sehen warum, aber es hat was mit Wartbarkeit zu tun.)

Ein solides Verständnis dieses Konzepts ist extrem wichtig, bevor wir weiter machen. Deshalb untersuchen wir noch ein paar Beispiele aus den Tiefen des npm Repositories.

```js
// ignorant
var getServerStuff = function(callback){
  return ajaxCall(function(json){
    return callback(json);
  });
};

// erhaben
var getServerStuff = ajaxCall;
```

Die Welt ist zugemüllt mit ajax code der genau so aussieht. Hier sieht man, warum beide varianten gleich sind.

```js
// diese Zeilen
return ajaxCall(function(json){
  return callback(json);
});

// entsprechen dieser Zeile
return ajaxCall(callback);

//  getServerStuff refactored
var getServerStuff = function(callback){
  return ajaxCall(callback);
};

// ...was dem hier entspricht
var getServerStuff = ajaxCall; // <-- guck Mutti, keine Klammern
```

Und so wird's gemacht Leute. Jetzt nochmal, damit ihr erkennt warum ich so darauf herumreite.

```js
var BlogController = (function() {
  var index = function(posts) {
    return Views.index(posts);
  };

  var show = function(post) {
    return Views.show(post);
  };

  var create = function(attrs) {
    return Db.create(attrs);
  };

  var update = function(post, attrs) {
    return Db.update(post, attrs);
  };

  var destroy = function(post) {
    return Db.destroy(post);
  };

  return {
    index: index, show: show, create: create, update: update, destroy: destroy
  };
})();
```

Dieser lachhafte Controller besteht zu 99% aus heißer Luft. Wir könnten ebenso schreiben:

```js
var BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy
};
```

...oder alles zusammen gemantsch, macht das nichts anderes als unser Views und Db zu bündeln.

## Warum die erste Klasse bevorzugen?

Okay, mal sehen, warum wir First Class Functions bevorzugen sollten. Wie uns die Beispiele `getServerStuff` und `BlogController` gezeigt haben, baut man schnell Funktionsverschachtelungen auf, die keinen Mehrwert haben und nur den Aufwand der Wartung erhöhen.

Zusätzlich  müssen wir eine einbettende Funktion auch ändern, wenn sich die eingebettete Funktion ändert.

```js
httpGet('/post/2', function(json){
  return renderPost(json);
});
```

Wenn `httpGet` einen weiteren Parameter `err` verarbeiten soll, müssen wir alle entsprechenden Aufrufe mit anpssen.

```js
// gehe zu jedem httpGet-Aufruf in der Applikation und übergebe explizit err
httpGet('/post/2', function(json, err){
  return renderPost(json, err);
});
```

Wenn wir das als eine first class function geschrieben hätten, wären viel weniger Änderungen nötig gewesen.

```js
// renderPost wird aus httpGet heraus mit wer weiß wieviel Argumenten aufgerufen
httpGet('/post/2', renderPost);
```

Neben dem beseitigen unnötiger Funktionen, müssen wir Argumente benennen und referenzieren. Namensgebung ist ein Problem. Wir haben potentielle falsch Benamsumgen - besonders wenn unsere Code basis altert und verändert werden muss.

Verschiedene Namen für ein und das selbe Konzept zu haben ist ein verbreiteter Grund für Verwirring in Projekten. Dann gibt es noch das Problem generischen Codes. Zum Beispiel machen diese zwei Funktionen exakt das gleiche, wobei die einer generischer und wiederverwendbarer ist:

```js
// spezifisch für unseren aktuellen Blog
var validArticles = function(articles) {
  return articles.filter(function(article){
    return article !== null && article !== undefined;
  });
};

// für zukünftige Projekte viel relevanter
var compact = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

In dem wir spezifische Namen vergeben, haben wir uns selbst an spezifische Daten gebunden (in diesem Fall `articles`). Das passiert nur allzuofft und ist der Grund für häufiges neuerfinden.

Es muss erwähnt werden, dass man genauso wie in der Objekt-Orientierten Schreibweise aufpassen muss, nicht vom `this` in die Halsschlagader gebissen zu werden. Wenn eine zugrundeliegende Funktion `this` verwendet, werden wir Opfer einer undichten Abstraktion.

```js
var fs = require('fs');

// erschreckend
fs.readFile('freaky_friday.txt', Db.save);

// schon besser
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

Wenn es an sich selbst gebunden ist, kann `Db` frei auf seinen schrottigen Code zugreifen. Ich meide `this` wie eine dreckige Windel. Dafür gibt es keinen Grund, wenn man funktionalen Code schreibt. Wie dem auch sei, wenn Du andere Libraries verwendest, musst Du die bescheuerte Welt um uns herum akzeptieren.

Mancher wird sagen, dass `this` besser für die Performance ist. Wenn Du zu der micro-optimization Fraktion gehörst, mach dieses Buch bitte zu. Falls Du Dein Geld nicht zurück bekommst, kannst du es vielleicht gegen etwas fummeligeres eintauschen.

Und jetzt sind wir soweit, dass wir fort fahren können.

[Kapitel 3: Pures Vergnügen mit Pure Functions](ch3-de.md)
