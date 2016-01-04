# Kapitel 2: Functionen Erster Klasse

## Ein kurzer Rückblickk
Wenn wir sagen, Funktionen sind "Erster Klasse" meinen wir, dass sie wie alle anderen sind...also normaler Klasse (Herr Schaffner?). Wir können Funktionen wie jeden anderen Datentype verwenden und es gibt keine Sonderlockenn an ihnen - sie können in Arrays gespeichert werden, herumgereicht werden, Variablen zugeordnet werden, was immer Du willst.

Das sind JavaScript Grundlagen, aber es sollte erwähnt werden, dass eine kurze Suche auf Github das kollektive Vermeiden oder viemehr die weitverbreitete Ignoranz dieses Konzepts.Sollen wir uns ein vorgetäuschtes Beispiel ansehen? Ja sollten wir.

```js
var hi = function(name){
  return "Hi " + name;
};

var greeting = function(name) {
  return hi(name);
};
```

Hier ist der Funktion Wrapper um `hi`in `greeting` komplett redundant. Warum? Weil Funktionen in JavaScript *aufrufbar* sind. Wenn `hi` mit `()` am Ende aufgerufen wird, läuft die Funktion durch und gibt einen Wert zurück. Wenn die Klammern fehlen, wird lediglich die Funktion, die in der Variable gespeichert ist, zurück gegeben. Nur um sicher zu gehen, wirf mal einen Blick darauf:

```js
hi;
// function(name){
//  return "Hi " + name
// }

hi("jonas");
// "Hi jonas"
```

Weil `greeting` nichts anderes tut, als `hi`mit dem selben Argument aufzurufen, können wir genauso schreiben:

```js
var greeting = hi;


greeting("times");
// "Hi times"
```

Mit anderen Worten, `hi` ist bereits eine Funktion, die ein Argument erwartet, warum sollten wir eine andere Funktion darum packen, die schlicht `hi` mit dem dummen Argument aufruft? Es mahct einfach keinen Sinn. Es ist so, als wolle man seinen schwersten Parka im heißesten Juli, nur um die Luft zu verdrängen und einen gekühlten Lolly zu bestellen.

Es ist widerlich verbos und, wie könnte es anders sein, ein schlechter Stil eine Funktion mit einer anderen Funktion zu umgeben, nur um ihre Verarbeitung zu verzögern. (Wir werden noch sehen warum, aber es hat was mit Wartbarkeit zu tun.)

Ein solides Verständnis dieses Konzepts ist extrem wichtig, bevor es weitergeht. Deshalb untersuchen wir noch ein paar Beispiele aus den Tiefen des npm repositories.

```js
// ignorant
var getServerStuff = function(callback){
  return ajaxCall(function(json){
    return callback(json);
  });
};

// enlightened
var getServerStuff = ajaxCall;
```

Die Welt ist zugemüllt mit ajax coder der genau so aussieht. Hier sieht man, warum beide varianten gleich sind.

```js
// diese Zeile
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

Und so wird's gemacht Leute. Jetzt nocheinmal, damit wir sehen warum ich so darauf herumreite.

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

Dieser witzige Controller besteht zu 99% aus heißer Luft. Wir könnten genaus schreiben:

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

## Warum wir die erste Klasse bevorzugen?

Okey, lass uns anschauen, warum wir first class Funktionen bevorzugen. Wie uns die Beispiele `getServerStuff` und `BlogController` Beispiele gezeigt haben, passiert es schnell, dass man man Funktionsverschachtelungen aufbaut, die keinen Mehrwert haben und nur den Aufwand der Wartung erhöhen.

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

Wenn wir das als eine first class function geschrieben, wären viel weniger Änderungen nötig gewesen.

```js
// renderPost wird aus httpGet heraus mit wer so wieviel Argumenten wie es will aufgerufen
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

In dem wir spezifische namen vergeben, haben wir uns selbst an spezifische daten gebunden (in diesem Fall `articles`). Das passiert nur allzuofft und ist der Grnd für häufiges neuerfinden.

Ich muss erwähnen, dass genauso wie im Objekt-Orientierten Code, man aufpassen muss, nicht vom 'this in die Halsschlagader gebissen zu werden. Wenn eine zugrundeliegende Funktion `this` verwendet, werden wir Opfer des Zorns einer undichten Abstraktion.

```js
var fs = require('fs');

// erschreckend
fs.readFile('freaky_friday.txt', Db.save);

// schon besser
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

Wenn es an sich selbst gebunden ist, kann `Db` frei auf seinen schrottigen Code zugreifen. Ich meide `this` wie eine dreckige Windel. Dafür gibt es keinen Grund, wenn man funktionalen Code schreibt. Wie dem auch sei, wenn Du andere Libraries verwendest, musst Du die bescheuerte Welt um uns akzeptieren.

Mancher wird sagen, dass `this` besser für die Performance ist. Wenn Du zu der micro-optimization Fraktion gehörst, mach dieses Buch zu. Falls Du Dein Geld nicht zurück bekommst, kannst du es vielleicht gegen etwas fummeligeres eintauschen.

Und jetzt sind wir soweit, dass wir fort fahren können.

[Chapter 3: Pure Happiness with Pure Functions](ch3.md)
