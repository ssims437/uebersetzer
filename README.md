# Übersetzer

**Von Text zu Maschine, Schritt für Schritt.** Eine kleine Programmiersprache namens **Kiesel** —
und die ganze Kette dahinter offen auf dem Tisch: Zeichen werden Wörter, Wörter werden ein Baum,
der Baum wird Bytecode, der Bytecode wird Bewegung auf einem Stapel.

→ **[Blatt öffnen](https://ssims437.github.io/uebersetzer/)**

- **Lexer** — Zeichen zu Marken, jede mit Zeile und Spalte
- **Pratt-Parser** — Vorrang als Tabelle statt als Grammatikregel je Stufe
- **Syntaxbaum** — gezeichnet, seitlich rollbar, mit Knoten- und Tiefenzähler
- **Übersetzer** — Bytecode für eine Stapelmaschine, inklusive vorab gerechneter Stapeltiefe
- **Optimierer** — Konstantenfaltung, doppelte Vorzeichen, toter Code nach `zurück`, Guckloch
- **Maschine** — Schritt-Debugger mit Stapel, Aufrufkeller, Variablen und laufender Zeile im Bytecode
- **Prüflauf** — zehn Zeilen, darunter 10 000 mutwillig zerstörte Quelltexte

## Die Sprache

Deutsche Schlüsselwörter, weil das Blatt deutsch ist. Genug für echte Programme:

```
funktion fib(n)
  wenn n < 2 dann
    zurück n
  ende
  zurück fib(n - 1) + fib(n - 2)
ende

sei i = 0
solange i <= 10 tue
  zeige "fib(" + i + ") = " + fib(i)
  i = i + 1
ende
```

Zahlen, Wahrheiten, Text, Variablen (`sei`), Verzweigung (`wenn … dann … sonst … ende`), Schleife
(`solange … tue … ende`), Funktionen mit Rekursion, `zeige`, `zurück`. Operatoren `+ - * / %`,
Vergleiche, `und` / `oder` / `nicht` — die beiden ersten mit Kurzschluss.

## Was der Prüflauf zeigt

| Behauptung | Ergebnis |
|---|---|
| Pratt-Parser = Rangierbahnhof | **2000 zufällige Ausdrücke**, Baum für Baum identisch |
| Baum → Text → Baum trifft sich selbst | 1500 Programme ausgeschrieben und neu zerlegt, kein Unterschied |
| **Maschine = Auswerter auf dem Baum** | **1200 erzeugte Programme**, Ausgabe Zeile für Zeile verglichen |
| Tempo gemessen | fib(21): 389 634 Befehle in ~9 ms (**41–50 Mio Befehle/s**), Baum ~11 ms — Faktor **1,2–1,4** |
| bei Einmalprogrammen gewinnt der Baum | 1200 winzige Programme mit Übersetzen: Faktor **0,88** — das Übersetzen muss sich erst lohnen |
| Optimierer lässt das Ergebnis in Ruhe | 1200 Programme mit und ohne, identische Ausgabe · 19 939 → 17 649 Befehle (**11,5 % weniger**) |
| gerechnete Stapeltiefe hält | 319 Ketten gemessen, keine brauchte mehr als vorab gerechnet, kein Rest auf dem Stapel |
| bekannte Ergebnisse treffen | fib(20)=6765 · 10!=3628800 · ggt(1071,462)=21 · Collatz(27)=111 Schritte · 25 Primzahlen unter 100 |
| **zerstörte Quelltexte stürzen nicht ab** | **10 000 mutierte Programme**, alle sauber behandelt, keiner ohne Fundstelle |
| Fehler zeigen auf die richtige Zeile | 5 kaputte Programme, jede Fundstelle stimmt |

Der Kern sind die ersten drei Zeilen: **zwei Parser und zwei Auswerter, unabhängig geschrieben.**
Ein Übersetzer, der falschen Code erzeugt, fällt sonst nicht auf — er rechnet einfach etwas anderes.

## Was mich das gekostet hat

**Meine Tempo-Behauptung war falsch, und der eigene Prüflauf hat sie kassiert.** Im Blatt stand
„dieselbe Rechnung, andere Form, und ein Vielfaches an Tempo". Der erste Lauf meldete **Faktor
0,9** — der Baum-Auswerter war schneller. Der Grund war die Messung selbst: sie hatte das
Übersetzen mitgestoppt, und bei winzigen, einmal ausgeführten Programmen kostet das mehr, als der
Bytecode einbringt. Getrennt gemessen ergeben sich zwei Zahlen statt einer:

| Regime | Baum | Maschine | Faktor |
|---|---|---|---|
| 1200 winzige Programme, je einmal (inkl. Übersetzen) | 35 ms | 39 ms | **0,88** |
| fib(21), fertig übersetzt | 11 ms | 9 ms | **1,26** |

Beide stehen jetzt im Blatt, und der Absatz behauptet nicht mehr „ein Vielfaches", sondern nennt
den gemessenen Wert. Die Regel dahinter ist die unbequeme: **wenn Messung und Text sich
widersprechen, ändert man den Text.**

**Faktor 1,26 statt der erhofften 5.** Das ist die zweite unbequeme Zahl. Ein Bytecode-Interpreter
in JavaScript gewinnt gegen einen Baum-Auswerter in JavaScript wenig, weil beide dieselben
Objekt-Zugriffe und dieselbe Sprache benutzen. Ein Schnellweg für den Normalfall — beide Operanden
sind Zahlen, also direkt rechnen statt über die Zeichenkette des Operators und eine zweite
Funktion — hat immerhin **24 → 41–50 Mio Befehle/s** gebracht. Der Rest des Lehrbuchvorsprungs
steckt in Dingen, die dieses Blatt bewusst nicht tut: typisierte Felder statt JavaScript-Werten,
Sprungtabellen statt `switch` über Zeichenketten, kein Objekt je Rahmen.

**Eine Tempo-Messung ist ein schlechter Test.** Erst stand die Zeile als Bestehen/Durchfallen im
Prüflauf — mit Faktor 1,05 wäre sie auf einem anderen Rechner rot geworden und hätte die CI
grundlos scheitern lassen. Jetzt ist sie eine **Messung**, kein Urteil: sie zeigt die Zahl, und
bestanden ist sie, wenn beide Wege dasselbe Ergebnis liefern. Dazu misst sie sauberer — ein
Aufwärmlauf, dann der beste von drei Durchgängen, weil Einzelmessungen zwischen **0,98 und 1,86**
schwankten, ohne dass sich am Code etwas geändert hätte.

**`und` und `oder` lieferten zweierlei.** Der Baum-Auswerter gab für `1 und 3` die Wahrheit
`wahr` zurück, die Stapelmaschine die Zahl `3` — beide „richtig", nur eben nicht dasselbe, und der
Differenztest hätte sie als Widerspruch gemeldet. Aufgefallen ist es beim Durchdenken der
Sprungbefehle, bevor der Test lief: der Kurzschluss-Sprung legt `wahr`/`falsch` ab, der
Durchlauf-Fall aber den rohen Wert. Die Lösung ist ein eigener Befehl `WAHRHEIT`, der den obersten
Wert auf eine Wahrheit bringt — eine Zeile Bytecode mehr je `und`, dafür eine Sprache, in der eine
Frage eine Antwort hat.

**Der Bytecode blieb nach einem Syntaxfehler stehen.** Bei kaputtem Quelltext wurden Wörterliste
und Baum geleert, die Befehlsliste aber nicht — daneben stand dann eine rote Fehlermeldung, und
die 29 Befehle des **vorigen** Programms behaupteten, dazuzugehören. Genau die Sorte Anzeige, der
man beim Debuggen glaubt.

**`<Funktion fib>` war unsichtbar.** Konstanten werden per `innerHTML` in die Befehlstabelle
geschrieben, und der Browser hat die spitzen Klammern als Auszeichnung geschluckt: in der Zeile
stand `KONST` und dahinter nichts. Jetzt läuft alles, was aus Werten kommt, durch eine Maskierung.

**Was der Fuzz-Test wert war:** 10 000 mutierte Quelltexte — Zeichen gelöscht, eingefügt,
vertauscht — liefen im ersten Anlauf **ohne einen einzigen Absturz** durch. Das klingt nach einem
Test, der nichts findet. Er ist trotzdem der wertvollste im Blatt, weil er die Zusage prüft, die
ein Übersetzer eigentlich geben muss: *auf jede Eingabe entweder ein Ergebnis oder eine
Fehlermeldung mit Fundstelle — nie ein Absturz.* Ohne ihn wüsste ich nur, dass meine sechs
Beispiele funktionieren.

**Eine Schwäche, die stehen bleibt:** Bei `sei b = a * (2 +` zeigt die Meldung auf die **nächste**
Zeile (`zeige b`), nicht auf die offene Klammer. Der Parser merkt sich nicht, wo eine Klammer
aufging — er merkt nur, dass da, wo ein Ausdruck stehen müsste, ein `zeige` steht. Das
ordentlich zu machen hieße, Klammerpositionen mitzuführen und beim Scheitern die älteste offene
zu melden. Wäre ein halber Tag; die Meldung ist auch so richtig, nur unfreundlich.

**Was das Blatt nicht kann:** keine Closures (Funktionen sehen nur ihre Parameter und die globale
Ebene), keine Listen, keine Objekte, keine Zeichenketten-Operationen außer Verketten, keine
Typprüfung vor der Laufzeit, keine Register-Zwischenform, kein JIT. Die Rekursion ist bei 200
Rahmen gedeckelt, und die Zahlen sind JavaScript-Zahlen mit allem, was das bedeutet — siehe
[Nachkomma](https://github.com/ssims437/nachkomma).

## Technik

Eine einzelne HTML-Datei. Kein Build, keine Bibliothek, nichts verlässt den Browser.
Canvas 2D für den Syntaxbaum, reines JavaScript, hell und dunkel.

## Die ganze Sammlung

Alle Blätter nach Feld geordnet, jedes mit eigenem Repo:
**[ssims437.github.io](https://ssims437.github.io/)**

## Lizenz

MIT
