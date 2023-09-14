---
layout: [post, post-xml]
title: "F.I.R.S.T. Prinzipien für solides Unit-Testen"
date: 2023-07-19 23:15
modified_date: 2023-07-19
author_ids: [dkstern]
categories: [Softwareentwicklung]
tags: [Testing, CleanCode]
---

F.I.R.S.T. ist ein Akronym, welches verschiedene Prinzipien beim Schreiben von Tests umfasst.
Die F.I.R.S.T. Prinzipien wurden von Robert C. Martin in Clean Code kurz vorgestellt und in diesem Blog möchte ich weiter in die Tiefe dieser Prinzipien eingehen.
Gerade beim Einstieg in die Programmierung können diese Prinzipien als Leitfaden dienen.
Diese Prinzipien haben das Ziel, dass Unit-Tests verständlich, wartbar und aussagekräftig sind.

Dieser Blog richtet sich an alle frischen und erfahrenen Entwickelnden, welche sich ein wenig mit dem Schreiben von Unit-Tests auseinandersetzen möchten.

# TL;DR #️⃣

- **F**ast: Unit-Tests sind schnell und dauern nur wenige Millisekunden.
- **I**ndependent/**I**solated: Unit-Tests sind unabhängig voneinander und können in beliebiger Reihenfolge ausgeführt werden.
- **R**epeatable: Unit-Tests werden häufig und auf verschiedenen Umgebungen ausgeführt, dabei liefern diese zuverlässig die gleichen Ergebnisse.
- **S**elf Validating: Ergebnisse werden programmatisch ausgewertet und liefern ein binäres Ergebnis (success/fail).
- **T**imely/**T**horough: Unit-Tests werden "rechtzeitig" geschrieben, indem diese schon vor dem produktiven Code erstellt werden. Außerdem wird "gründlich" getestet, indem (neben den Erfolgsfällen) auch Fehlerfälle, Grenzwerte und Äquivalenzklassen getestet werden.

# F - Fast 🏇

Unit-Tests sind schnell!
Möchte man nach Anpassungen im Code die Unit-Tests nicht ausführen, weil diese einem persönlich zu lange dauern, dann ist dies ein Alarmsignal dafür, dass die Tests keine Unit-Tests sind.
Denn ein Unit-Test dauert bei der Ausführung nur wenige Millisekunden und dies sollte keinen signifikanten Einfluss auf die persönliche Wahrnehmung haben.
Alleine das Lesen dieses Satzes dauert länger, als die Zeit die mehrere hundert Unit-Tests für die Ausführung benötigen.
Dauert ein Unit-Test länger als eine Sekunde, sollte man diesen nochmal genauer betrachten, denn meist verbirgt sich dahinter schon ein Integrations-Test.
Bei einem Integrations-Test wird weiterer Code außerhalb unserer zu testenden Methode oder Funktion ausgeführt.
Dies kann sehr schnell passieren, indem bspw. Abhängigkeiten nicht "gemockt" wurden.
Mocks sind Stellvertreter-Objekte/Funktionen, welche nur in Unit-Tests existieren und so tun, als wären diese ein konkretes Objekt von einer Klasse oder einer Funktion.
Im schlimmsten Fall werden ohne Mocks Live-Services aufgerufen und ehe man sich versieht, hat man erfolgreich seinen ersten kleinen DOS-Angriff durchgeführt.
Es sollte somit klar sein, was für Auswirkungen Tests haben können, wenn diese nicht nur den zu testenden Code ausführen.

# I - Isolated / Independent ✊

Unit-Tests sind unabhängig voneinander!
Das Ziel: Unit-Tests können in beliebiger Reihenfolge ausgeführt werden und es spielt keine Rolle, welcher Unit-Test davor ausgeführt wurde.
Dementsprechend sollten keine Variablen und Objekte zwischen Unit-Tests geteilt werden, auch nicht das Testobjekt selbst.
Generell, sollten alle Abhängigkeiten bei unserer zu testenden Methode oder Funktion betrachtet und isoliert werden, indem diese bspw. durch Mocks ersetzt werden.
Dadurch werden unerwünschte Seiteneffekte vermieden und Unit-Tests werden nicht nur aussagekräftiger, sondern auch besser wartbar.

## Die drei A's (oder auch `given`, `when`, `then`)

Zur besseren Struktur und Auffindbarkeit von Abhängigkeiten, kann die Verwendung von `arrange`, `act`, `assert` oder alternativ `given`, `when`, `then` unterstützen.
Dabei wird ein Unit-Test in drei Bereiche aufgeteilt, indem diese drei Schlagwörter als Kommentare in Unit-Test geschrieben werden.

Folgendes sollte in diesen drei Bereichen zu finden sein:

### `arrange` (oder `given`)

Hier findet die Testvorbereitung statt, wo Variablen definiert und alle weiteren Konfigurationen gesetzt werden.
Dies ist quasi unsere Einleitung für unseren Unit-Test und hilft uns die Situation besser zu verstehen, welcher unser Unit-Test abdecken soll.
Wenn alle Unit-Tests dieselbe Testvorbereitung haben, können und sollten diese ausgelagert werden.
Dazu bieten in der Regel alle gängigen Test-Frameworks, wie bspw. JUnit oder Jest, Setup-Möglichkeiten an worin unsere Testvorbereitungen ausgelagert werden können.
Bei Jest gibt es die Funktionen `beforeEach` und `beforeAll`.
Bei JUnit die Annotationen `@BeforeEach` und `@BeforeAll`.

### `act` (oder `when`)

Hier befindet sich die eigentliche Testausführung.
Mit anderen Worten, die zu testende Methode oder Funktion wird hier aufgerufen.

### `assert` (oder `then`)

In diesem Bereich wird das Ergebnis unseres Tests programmatisch ausgewertet, indem das Ergebnis mit unserer Erwartung verglichen wird.
Grundsätzlich sollten Unit-Tests immer nur ein Ergebnis erwarten und auswerten.
Mit anderen Worten, ein Unit-Test enthält in der Regel nur ein `assert()` oder `expect()` Aufruf.

Zur besseren Verständlichkeit ein kleines Beispiel:

Wir stellen uns vor, wir haben einen Online-Shop für Katzenbedarf und möchten allen Premium Kunden 15 % Rabatt gewähren.
Wir haben eine REST-Schnittstelle, welche angesprochen werden kann, um einen User den Rabatt zu gewähren (bspw. GET /v1/discount).
Ich persönlich bevorzuge die `given`, `when`, `then` Schreibweise, denn mit dieser können wir Unit-Tests vorher leicht in Prosa formulieren:

- **Given** a user with premium status
- **When** this user requests, the discount is made
- **Then** the user should receive 15% discount

Dies kann dann in einen konkreten Unit-Test wie folgt aussehen:

```javascript
describe('getDiscountForUser', () => {
    let testSubject: DiscountController;
    
    beforeEach(() => {
        testSubject = new DiscountController();
    });
    
    it('should return 15% discount when user have premium status', () => {
        // given a user with premium status
        const user = UserService.newWithPremiumStatus();
        
        // when this user requests, the discount is made
        const discount = testSubject.getDiscountForUser(user)
        
        // then the user should receive 15% discount
        expect(discount.value).toEqual(15);
    });
    [...]
});
```

Keine Sorge, man braucht nicht mehr wie `given`, `when`, `then` oder `arrange`, `act`, `assert` in den Unit-Test schreiben.
Dies habe ich hier nur gemacht, damit der Übergang von den oben in Prosa geschrieben Text, mit der konkreten Test-Implementierung besser nachvollzogen werden kann.

Durch das Aufteilen der Unit-Tests in diese drei Bereiche wird die Lesbarkeit gesteigert, was uns wiederrum dabei unterstützt Abhängigkeiten aufzudecken, besonders dann, wenn das Test-Setup mehr wie nur eine Zeile Code ist.
Im Prinzip können wir unseren Unit-Test wie eine kleine Geschichte lesen.
Wir haben im `given`-Bereich eine Einleitung in der sich unsere Geschichte (bzw. unser Unit-Test) aufbaut.
Darauf steigt im `when`-Bereich der Spannungsbogen zum Höhepunkt, indem dort eine konkrete Handlung erfolgt (die Testausführung).
Nun neigt sich die Geschichte im `then`-Bereich dem Ende zu und wir vergleichen die Ergebnisse der Handlung mit unseren Erwartungen, die wir vorher zu unserer Geschichte angenommen haben.
Zugegeben, es mag keine besonders spannende Geschichte sein.
Da wir schon im Vorfeld Annahmen treffen, die sich am Ende unserer Geschichte bewahrheiten sollen.
Allerdings ist dies ja schließlich genau das, was wir in einem Unit-Test haben möchten: Erwartungen die sich bewahrheiten.
Letzten Endes sollte ein guter Unit-Test genauso leicht verständlich sein wie eine einfache Geschichte.
Also keine plötzlichen Site-Stories (Sprünge im Code) die mich als Lesenden dazu zwingen in der Geschichte hin und her zu springen, um das Gesamtbild zu durchblicken, sondern ich möchte einen Unit-Test von oben nach unten einfach durchlesen und verstehen können.

# R - Repeatable 🔄

Unsere Unit-Tests sollen häufig und auf verschiedenen Umgebungen ausgeführt werden können, dabei liefern diese immer die gleichen Ergebnisse für die gleichen Eingaben.
Das bedeutet, ein Unit-Test liefert:
- unabhängig von der Uhrzeit der Ausführung
- unabhängig der zuvor ausgeführten Unit-Tests
- unabhängig zur Umgebung 

immer das gleiche nachvollziehbare Ergebnis zurück.

# S - Self Validating 📋

Unit-Tests sollen programmatisch das Ergebnis auswerten.
Wir möchten nicht nach jeder Testausführung selbst das Ergebnis auswerten müssen, indem wir bspw. in logfiles schauen.
Zumal gut getestete Projekte mit Leichtigkeit auf mehrere tausend Unit-Tests kommen.
Man stelle sich vor, man müsse bei dieser Anzahl von Unit-Tests händisch die Ergebnisse auswerten.
Außerdem sollte ein Unit-Test ein explizites und binäres Feedback geben, also `success` oder `fail` und nichts dazwischen.
Manchmal ist die Welt eben doch nur schwarz und weiß, bzw. rot und grün.

# T - Timely / Thorough 🕑

Robert C. Martin definiert das "T" in F.I.R.S.T. mit "timely".
Das bedeutet, Unit-Tests sollen schon vor dem produktiven Code geschrieben werden.
Dies macht das Schreiben von Unit-Tests leichter und der produktive Code wird zwangsläufig testbar.
Es ist einfach schwerer produktiven Code auf Anhieb testbar zu schreiben, als wenn man schon vorher einen Unit-Test hat, welcher einen dazu nötigt, den produktiven Code testbar zu gestalten.

Mit der Zeit hat sich zum "T" noch ein weiteres Prinzip eingeschlichen, und zwar "thorough".
"Thorough" bedeutet übersetzt "gründlich".
Wir schreiben also unsere Unit-Tests gründlich.
Dies erreichen wir, indem nicht nur Testfälle für die Erfolgsfälle, sondern auch die Fehlerfälle, Grenzfälle und [Äquivalenzklassen](https://de.wikipedia.org/wiki/%C3%84quivalenzklassentest) geschrieben werden.

Was dieses Prinzip letzten Endes bedeutet, lässt sich an folgendem Beispiel zeigen:

So nehmen wir eine bekannte Kata (eine kleine abgeschlossene Übung) "FizzBuzz".
Diese Aufgabe sieht vor, dass für:

- natürliche Zahlen die durch 3 teilbar sind mit "fizz" auf der Konsole ausgegeben werden sollen
- natürlichen Zahlen die durch 5 teilbar sind mit "buzz" auf der Konsole ausgeben werden sollen
- natürliche Zahlen die sowohl durch 3 als auch durch 5 teilbar sind, mit "fizzbuzz" auf der Konsole ausgegeben werden sollen
- alle anderen Zahlen, die jeweilige Zahl selbst auf der Konsole ausgeben werden soll

Zusätzlich nehmen wir noch die Bedingung auf, dass nur Zahlen zwischen 1 und 30 berechnet werden sollen.
Bei Zahlen außerhalb unseres Intervalls soll ein Fehler geworfen werden.

Aus dieser Aufgabe ergeben sich nun diverse Testfälle, wie bspw. folgende:

**Positiv-Testfälle**
- **Given** is number 2. **When** fizzbuzz function executes, **Then** the answer should be "2".
- **Given** is number 3. **When** fizzbuzz function executes, **Then** the answer should be "fizz".
- **Given** is number 5. **When** fizzbuzz function executes, **Then** the answer should be "buzz".
- **Given** is number 15. **When** fizzbuzz function executes, **Then** the answer should be "fizzbuzz".

**Fehlerfall-Testfälle**
- **Given** is number -10. **When** fizzbuzz function executes, **Then** an error should be thrown.
- **Given** is number 310. **When** fizzbuzz function executes, **Then** an error should be thrown.

**Grenzwert-Testfälle**
- **Given** is number 1. **When** fizzbuzz function executes, **Then** the answer should be "1".
- **Given** is number 30. **When** fizzbuzz function executes, **Then** the answer should be "fizzbuzz".
- **Given** is number -1. **When** fizzbuzz function executes, **Then** an error should be thrown.
- **Given** is number 31. **When** fizzbuzz function executes, **Then** an error should be thrown.

**Testfälle für Äquivalenzklassen-Abdeckung**
- **Given** is number 6. **When** fizzbuzz function executes, **Then** the answer should be "fizz".
- **Given** is number 10. **When** fizzbuzz function executes, **Then** the answer should be "buzz".
- **Given** is number -5. **When** fizzbuzz function executes, **Then** an error should be thrown.
- **Given** is number 100. **When** fizzbuzz function executes, **Then** an error should be thrown.

Man sieht in diesem Beispiel sehr gut, wie viele Testfälle sich für solch eine einfache Aufgabe schreiben lassen, damit dieses Prinzip erfüllt ist.
Wir haben hier Erfolgreiche-, Fehlerfall- und Grenzwert-Testfälle geschrieben und zusätzlich noch weitere um alle Äquivalenzklassen abzudecken.
Zu beachten ist, dass die hier aufgeführten Testfälle nicht in Stein gemeißelt sind.
Es gibt selbstverständlich noch andere Zahlen, die man hier hätte wählen könnte.
Auch die Anzahl an notwendigen Testfällen ist variabel.
Denn Fakt ist, man kann nicht jedes Szenario Testen und das möchte man auch nicht.
Es würde einfach viel zu lange dauern und irgendwann ist der Mehrwert von zu vielen Unit-Tests auch nicht mehr gegeben, da diese auch gewartet werden müssen.
Aus diesem Grund versuchen wir so viele sinnvolle Tests zu schreiben wie möglich, aber gleichzeitig auch so wenig wie nötig.
Um dies zu erreichen, kann man sich folgende Fragen beim Schreiben von Testfällen stellen:

- Sind alle wichtigen Positiv-Testfälle vorhanden?
- Sind alle wichtigen Fehlerfall-Testfälle vorhanden?
- Habe ich alle Grenzwerte abgedeckt?
- Gibt es noch weitere Äquivalenzklassen, die ich noch testen sollte?

# Fazit 🎉

Nach Verinnerlichung und Verstehen dieser Prinzipien, sollte hoffentlich das Schreiben von Unit-Tests etwas leichter von der Hand gehen.
Dennoch sollte man sich immer Bewusst sein, dass diese Prinzipien nicht dogmatisch angewendet werden sollten.
Man kann sich in einem Projekt auch auf andere Prinzipien festlegen oder diese Prinzipien abwandeln, wichtig ist hier das Verständnis im Team.
Letzten Endes finden sich die Test F.I.R.S.T. Prinzipien in Projekten mit gut strukturierten Unit-Tests immer wieder, auch eventuell in abgewandelter Form.
