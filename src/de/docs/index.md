RESTCaptcha ist eine einfach zu integrierende, datenschutzfreundliche CAPTCHA-Lösung, die vom Nutzer keine Bildrätsel oder Mathematikaufgaben abverlangt.

Stattdessen verwendet RESTCaptcha einen Proof-of-Work-Mechanismus, um die menschliche Interaktion mit Deinem Formular zu überprüfen.

Funktionen:

+ Zustandslose Herausforderung (challenge) mit HMAC-signiertem Nonce (Zeichenfolge zur einmaligen Verwendung)
+ Clientseitiges Proof-of-Work-Rätsel (SHA-256-, SHA-384- oder SHA-512-Hash unterhalb eines Schwellenwerts)
+ CDN-fähiges `restcaptcha.min.js`-Skript mit konfigurierbarem API-Endpunkt
+ Optionale Überprüfung der IP-Reputation via [AbuseDBIP](https://www.abuseipdb.com/) und/oder [Spamhaus](https://www.spamhaus.org/) als zusätzliche Schutzmaßnahme
+ Einfach integrierbar mit [Node.js](https://nodejs.org/), [PHP](https://www.php.net/), [ASP.NET Core](https://dotnet.microsoft.com/apps/aspnet) oder jeder anderen Servertechnologie
+ Beliebig anpassbar
+ Unterstützt vier verschiedene Modi (interaktiv, automatisch, unsichtbar, headless)
+ Multilingual (zurzeit: Englisch, Deutsch, Französisch, Italienisch, Portugiesisch, Spanisch)
+ Entwickelt mit [.NET](https://dotnet.microsoft.com/) und [JavaScript](https://developer.mozilla.org/docs/Web/JavaScript).

## Datenschutz

RESTCaptcha verarbeitet oder speichert keine personenbezogenen Daten:

+ Es werden keine Cookies oder serverseitige Tracking-Technologien eingesetzt.
+ Die Fingerprint-Überprüfung des Web-Browsers läuft vollständig clientseitig ab.
+ Beim Generieren der Challenge wird optional die Reputation der IP-Adresse des Clients überprüft. 
+ Bei der Verifizierung werden nur die für den technischen Ablauf notwendigen Daten verarbeitet: Die anonyme Challenge-Lösung und (je nach Konfiguration) die IP-Adresse des Clients für Logging-Zwecke.
+ RESTCaptcha kann vollständig auf eigenen Servern betrieben werden, sodass alle Daten unter eigener Kontrolle bleiben.
+ Der Quellcode ist offen (Open Source) und kann jederzeit überprüft oder auditiert werden.

## Funktionsweise

Wer beim Begriff *CAPTCHA* nur Bahnhof versteht, der sollte sich zunächst das Kapitel [Was sind CAPTCHAs?](captcha.md) durchlesen.

RESTCaptcha funktioniert ganz grob wie folgt:

1. Du integrierst das RESTCaptcha-Widget in Deine Webseite — klassischerweise ein Registrierungs- oder Kontaktformular.

2. Der Benutzer Deiner Webseite bekommt vom RESTCaptcha-Server ein Token zugesendet (das geschieht transparent im Hintergrund) und muss seinen Webbrowser eine Aufgabe lösen lassen, in dem er auf ein Kontrollfeld klickt (geht optional auch ohne Klicken).

3. Wenn der Benutzer Deiner Webseite das Formular ausgefüllt hat, sendet er die Daten an Deinen Server (also jenen, der Deine Webseite bereitstellt). Dabei wird auch der Token und die Lösung der Aufgabe mitgesendet.

4. Dein Server muss jetzt, bevor er die Formulardaten weiterverarbeitet, den Token und die Lösung der Aufgabe durch einen API-Aufruf beim RESTCaptcha-Server verifizieren lassen. Ist die Verifizierung erfolgreich, ist alles ok, andernfalls muss Dein Server davon ausgehen, dass es sich um einen Bot handelt.

## Live-Demos

Die folgenden Live-Demos stehen bereit:

<div class="grid cards" markdown>

-   :material-language-php:{ .lg .middle } &nbsp;
    __PHP-Demo__

    ---

    Wir haben eine kleine Webseite mit PHP und Bootstrap 5 implementiert, die den Einsatz von RESTCaptcha demonstriert.


    [:octicons-arrow-right-24: Zur Webseite](https://php-demo.restcaptcha.openpotato.org/) 

-   :material-dot-net:{ .lg .middle } &nbsp;
    __ASP\.NET-Demo__

    ---

    Wir haben die gleiche Webseite mit ASP\.NET Core und Bootstrap 5 implementiert.


    [:octicons-arrow-right-24: Zur Webseite](https://dotnet-demo.restcaptcha.openpotato.org/) 

</div>

Die Quellcodes für beide Demos befinden sich im [GitHub-Repository von RESTCaptcha](https://github.com/openpotato/restcaptcha).
