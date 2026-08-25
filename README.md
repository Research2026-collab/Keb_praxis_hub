# PraxisHub – README

Diese Datei hält fest, wie der PraxisHub aufgebaut ist, welche Datei wofür zuständig ist und welche Entscheidungen bereits getroffen wurden. Sie soll verhindern, dass bei einer künftigen Änderung wieder Unklarheit über den aktuellen Stand entsteht.

## Fachliche Grundlage

Die inhaltliche Struktur des PraxisHub, also alle Feldnamen, Kategorien und Pflichtfelder, ist an `Praxis_Hub_Kriterien_digitale_Tauschbörse_final.docx` ausgerichtet. Bei einer künftigen inhaltlichen Änderung, etwa einer neuen Kategorie oder einem geänderten Pflichtfeld, sollte zuerst dieses Dokument aktualisiert werden, und erst danach die Wertelisten, die Beiträge-Tabelle und die beiden HTML-Seiten. Andernfalls entsteht wieder derselbe Auseinanderlauf zwischen Konzept und Umsetzung, der die Überarbeitung im August 2026 nötig gemacht hat.

## Architektur

Der PraxisHub besteht aus drei Teilen, die zusammenspielen, aber an unterschiedlichen Orten liegen.

Die beiden sichtbaren Seiten sind reine, statische HTML-Dateien und liegen im GitHub-Repository Keb_praxis_hub, veröffentlicht über GitHub Pages. `praxishub-eingabe.html` zeigt das Formular zum Einstellen eines Beitrags, `praxishub-whiteboard.html` zeigt die durchsuchbare Übersicht mit Filtern, Detailansicht und der Jahresehrung. Beide Seiten enthalten keinerlei serverseitigen Code und könnten grundsätzlich auf jedem beliebigen Webserver liegen, nicht nur auf GitHub Pages.

Die eigentliche Datenhaltung erfolgt in einem Google Sheet mit den Tabellenblättern Wertelisten, Beiträge und Badges. Zwischen den beiden HTML-Seiten und diesem Sheet vermittelt ein Google-Apps-Script-Projekt mit der Datei `Code.gs`. Dieses Skript liest und beschreibt das Sheet und stellt seine Funktionen als Web-App über eine feste URL bereit, die mit `/exec` endet.

Die Verbindung zwischen den GitHub-Seiten und dem Apps-Script-Projekt läuft ausschließlich über den JavaScript-Befehl `fetch`, nicht über `google.script.run`, da diese Funktion außerhalb der Google-Apps-Script-Umgebung nicht existiert. Jede der beiden HTML-Seiten enthält am Anfang ihres Skript-Teils eine Konstante `APPS_SCRIPT_URL`, in die nach jedem neuen Deployment des Apps-Script-Projekts die aktuelle Web-App-Adresse eingetragen werden muss.

## Schnittstelle zwischen GitHub und Apps Script

`Code.gs` beantwortet vier Arten von Anfragen.

Ein GET-Aufruf mit dem Parameter `?action=wertelisten` liefert alle Wertelisten als JSON-Objekt, in dem jeder Spaltenname im Tabellenblatt Wertelisten einem Array seiner Werte zugeordnet ist. `praxishub-eingabe.html` nutzt dies, um seine Auswahlfelder zu füllen.

Ein GET-Aufruf mit `?action=list` liefert alle Beiträge als JSON-Array. Beiträge mit der Sichtbarkeit Reduzierte Anzeige werden dabei bereits im Skript auf die Felder Titel, Kurzbeschreibung, Hauptbereich, Zielgruppe, Mitgliedseinrichtung, Name, E-Mail, Umsetzungsstand, Sichtbarkeit und ID gekürzt, bevor die Antwort das Skript verlässt. Diese Kürzung geschieht serverseitig, damit sie sich nicht durch einen Blick in die Netzwerkanfragen des Browsers umgehen lässt.

Ein GET-Aufruf mit `?action=badges` liefert die Jahresauswertung je Mitgliedseinrichtung, zusammengesetzt aus der Anzahl im laufenden Kalenderjahr eingestellter Beiträge und einem zusätzlichen Punkt je Beitrag mit einem als positiv gewerteten Kooperationsinteresse, aktuell Suche Partner:in oder Offen für Austausch. Diese Berechnung erfolgt bei jedem Aufruf neu und schreibt nichts in das Tabellenblatt Badges. Ob und wann daraus tatsächlich ein an eine Einrichtung vergebener Jahres-Badge wird, bleibt eine redaktionelle Entscheidung, die von Hand im Tabellenblatt Badges festgehalten wird.

Ein POST-Aufruf ohne Parameter nimmt einen neuen Beitrag entgegen. Der Anfragetext muss ein JSON-Objekt mit den Feldnamen aus der Kopfzeile des Tabellenblatts Beiträge sein. `praxishub-eingabe.html` setzt den Content-Type dieser Anfrage bewusst auf `text/plain`, nicht auf `application/json`, da Apps Script keine CORS-Preflight-Anfragen beantwortet, die der Browser sonst vor einer JSON-Anfrage an eine fremde Domain verschickt.

## Wichtige Entscheidungen und offene Annahmen

Mitgliedseinrichtung wird im Formular als Pflicht-Auswahlfeld geführt, dessen Werte aus der Spalte Mitgliedseinrichtung im Tabellenblatt Wertelisten stammen, entnommen aus `2026_Mitglieder.xlsx`. Diese Verpflichtung stützt sich darauf, dass ohne sie keine Badge-Auswertung je Einrichtung möglich wäre.

Zielgruppe ist das einzige Feld mit echter Mehrfachauswahl, da im Beispiel des Konzeptdokuments nur dort mehrere Werte gleichzeitig auftreten. Hauptbereich wird dagegen als Einfachauswahl geführt, da das Dokument von einer „obersten" Zuordnung spricht. Sollte sich in der Praxis zeigen, dass Beiträge regelmäßig mehreren Hauptbereichen zugleich zuzuordnen sind, ließe sich dies nachträglich auf Mehrfachauswahl umstellen.

Sichtbarkeit ist ein Pflichtfeld mit den Werten Vollständige Anzeige und Reduzierte Anzeige. Bei Reduzierte Anzeige zeigt das Whiteboard nur Titel, Kurzbeschreibung, Hauptbereich, Zielgruppe und die Kontaktangaben, alle übrigen Felder bleiben verborgen.

Die Badge-Auswertung gewichtet Anzahl und Kooperationsbereitschaft gleich, ein Beitrag zählt einen Punkt, ein positives Kooperationsinteresse einen weiteren. Eine stärkere Gewichtung der Kooperationsbereitschaft wurde erwogen, aber nicht umgesetzt; die Liste `KOOPERATION_POSITIV` in `Code.gs` ist der Ort, an dem sich das ändern ließe.

## Nach einer Codeänderung

Eine Änderung an `Code.gs` wirkt sich erst aus, nachdem im Apps-Script-Projekt über Bereitstellen, dann Bereitstellungen verwalten eine neue Version angelegt wurde. Eine Änderung an einer der beiden HTML-Dateien wirkt sich hingegen erst aus, nachdem die geänderte Datei erneut in das GitHub-Repository hochgeladen wurde. Beide Schritte sind unabhängig voneinander und ersetzen sich nicht gegenseitig.
