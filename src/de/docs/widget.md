Diese Referenz dokumentiert alle wichtigen öffentlichen Klassen, Eigenschaften und Methoden, sowie die verfügbaren HTML-Attribute (`data-*`) und Ereignisse der `restcaptcha.js`-Bibliothek.

## Überblick

Die RESTCaptcha-JavaScript-Bibliothek stellt den erforderlichen JavaScript-Code für die clientseitige Integration von RESTCaptcha in eigene Webseiten bereit. 

Sie hat zwei Ausprägungen:

+ `restcaptcha.js`: Die Originalversion.
+ `restcaptcha.min.js`: Minimierte (minified) Version, bei der alle unnötigen Zeichen entfernt wurden, um die Dateigröße zu verkleinern und die Ladezeiten im Browser zu verbessern.

Die Bibliothek besteht aus zwei Hauptkomponenten:

1. `HeadlessRestCaptcha`: Ein reiner JavaScript-Client ohne Benutzeroberfläche (headless). Er kommuniziert direkt mit der RESTCaptcha-API, löst die kryptografische Challenge und gibt das Ergebnis über Ereignisse oder Callbacks zurück. Diese Variante eignet sich für Integrationen in eigene UI- oder Framework-Lösungen.

2. `RestCaptcha`: Ein auf dem Headless-Client basierendes, visuelles Widget, das automatisch eine einfache Benutzeroberfläche im Browser rendert. Es kann via HTML-Attribute frei konfiguriert werden, unterstützt mehrere Sprachen und Modi (interaktiv, automatisch oder unsichtbar).

## HeadlessRestCaptcha

Ein UI-loser Client, der direkt mit der RESTCaptcha-API kommuniziert, ein Proof-of-Work löst und über Ereignisse (Callbacks) gesteuert wird.

### Konstruktor

```js
new HeadlessRestCaptcha(apiBaseUrl, siteKey, language)
```

Die Parameter bedeuten:

**`apiBaseUrl`**

:   Basis-URL des RESTCaptcha-Backends (**Pflichtfeld**)

**`siteKey`**

:   Der RESTCaptcha-Sitekey (**Pflichtfeld**)

**`language`**

:   Sprachcode (z. B. `"de"`), automatische Erkennung, wenn leer

### Eigenschaften

**`HeadlessRestCaptcha.language`**

:   Effektiver Sprachcode

### Ereignisse

**`HeadlessRestCaptcha.onStarted`**

:   Eine Funktion, die aufgerufen wird, wenn das Lösen startet

**`HeadlessRestCaptcha.onSolved`**

:   Eine Funktion, die aufgerufen wird, , wenn das CAPTCHA erfolgreich gelöst wurde

**`HeadlessRestCaptcha.onFailed`**

:   Eine Funktion, die bei Fehlschlägen aufgerufen (z. B. ungültiger Token, zu viele Versuche) wird.

**`HeadlessRestCaptcha.onError`**

:   Eine Funktion, die bei unerwarteten Fehlern aufgerufen wird.

**`HeadlessRestCaptcha.onReset`**

:   Eine Funktion, die nach einem Reset aufgerufen wird.

### Methoden

**`HeadlessRestCaptcha.solve()`**

:   Startet die Kommunikation mit dem Backend, lädt eine Challenge, löst den Proof-of-Work (SHA-256/384/512) und ruft bei Erfolg `onSolved(token, solution)` auf.

**`HeadlessRestCaptcha.reset(token)`**

:   Invalidiert einen zuvor erhaltenen Token.

**`HeadlessRestCaptcha.isHeadless()`**:

:   Überprüft, ob der aufrufende Browser höchstwahrscheinlich headless oder automatisiert läuft.

## RestCaptcha

Das sichtbare CAPTCHA-Widget, das automatisch HTML rendert und mit dem bereits integrierten Headless-Client interagiert.

### Konstruktor

```js
new RestCaptcha(widgetId = "restcaptcha-widget")
```

Der Parameter bedeutet:

**`widgetId`**

:   Die ID des Widgets im Browser-DOM (**Pflichtfeld**)

## RestCaptchaProblemDetails

Implementiert den Standard [RFC 9457 Problem Details](https://datatracker.ietf.org/doc/html/rfc9457) für strukturierte API-Fehler. Erweitert die JavaScript-`Error`-Klasse.

### Konstruktor

```js
new RestCaptchaProblemDetails({
  type = 'about:blank',
  title = 'Unknown error',
  status = 0,
  detail = null,
  instance = null,
  traceId = null,
  errors = null
} = {})
```

Der Paremeter ist ein anonymes Objekt mit den passenden Eigschaften (siehe nächsten Abschnitt "Eigenschaften").

### Eigenschaften

**`RestCaptchaProblemDetails.type`**

:   URI, die den Fehlertyp beschreibt

**`RestCaptchaProblemDetails.title`**

:   Kurze, lesbare Zusammenfassung

**`RestCaptchaProblemDetails.status`**

:   HTTP-Statuscode

**`RestCaptchaProblemDetails.detail`**

:   Detaillierte Fehlerbeschreibung

**`RestCaptchaProblemDetails.instance`**

:   URI dieser Fehlersituation

**`RestCaptchaProblemDetails.traceId`**

:   Serverseitige Diagnose-ID

**`RestCaptchaProblemDetails.errors`**

:   Feldbezogene Validierungsfehler (optional)

### Instanzmethoden

**`RestCaptchaProblemDetails.toString()`**

:   Gibt den Fehler als `[Status] Titel: Detail` zurück.

### Statische Methoden

**`RestCaptchaProblemDetails.fromResponse(response, json)`**

:   Erstellt eine `RestCaptchaProblemDetails`-Instanz aus einer `fetch()`-Response und dem zugehörigen JSON-Body.

## Widget `data-*` Attribute

Diese HTML-Attribute steuern das Verhalten und das Design des Widgets.

### Grundkonfiguration

**`data-api-baseurl`**

:    Basis-URL der RESTCaptcha-API (**Pflichtfeld**)

**`data-sitekey`**

:    Öffentlicher Site-Schlüssel (**Pflichtfeld**)

**`data-widget-language`**

:    Sprachcode (`de`, `en`, `es`, `fr`, `it`, `pt`). Standardmäßig wird versucht, die Sprache automatisch vom Browser zu ermitteln, mit Fallback zu `en`.

**`data-widget-mode`**

:    Modus. Mögliche Werte sind `interactive` (Standard), `auto` oder `invisible`.

### Callback-Attribute


**`data-callback-started` > `() => void`**

:    Callback, nachdem die Challenge auf dem Client gestartet wurde.

**`data-callback-solved`  > `(token, solution) => void`**

:    Callback, nachdem eine erfolgreiche Verifikation gemeldet wurde.

**`data-callback-failed` > `(message) => void`**

:    Callback, nachdem eine ungültige Verifikation gemeldet wurde.

**`data-callback-error` > `(error) => void`**

:    Callback, nachdem ein API-Fehler gemeldet wurde.

**`data-callback-reset` > `(token) => void`**

:    Callback, nachdem der Reset-Link gedrückt wurde.

Beispiel:

```html
<script>
  function onCaptchaSolved(token, solution) {
    console.log("Gelöst:", token, solution);
  }
</script>

<div id="restcaptcha-widget"
     data-api-baseurl="https://captcha.beispiel.de/v1/"
     data-sitekey="<Mein-Site-Schlüssel>"
     data-widget-language="de"
     data-callback-solved="onCaptchaSolved">
</div>
```

### CSS-Klassen-Attribute

Das Widget erlaubt vollständige Kontrolle über CSS-Klassen durch `data-widget-css-*` Attribute.

Einige Attribute sind spezifisch für bestimmte Visualisierungen (z.B. `data-widget-css-interactive`), andere wiederum wiederum werden bei allen Visualisierungen eingestzt (z.B. `data-widget-css-footer`). In Kombination mit den `id`-Werten der HTML-Knoten ist jedoch maximale Flexibiltät möglich.

Beispiel für [Bootstrap 5](https://getbootstrap.com/):

``` html
<div 
  id="restcaptcha-widget" 
  data-api-baseurl="https://captcha.beispiel.de/v1/"
  data-sitekey="<Mein-Site-Schlüssel>"
  data-widget-language="de"
  data-widget-mode="interactive" 
  data-widget-css-interactive="alert alert-light"
  data-widget-css-interactive-body="form-check"
  data-widget-css-interactive-checkbox="form-check-input"
  data-widget-css-interactive-checklabel="form-check-label"
  data-widget-css-solving="alert alert-light"
  data-widget-css-solving-body="d-flex align-items-center"
  data-widget-css-solving-animation="spinner-border spinner-border-sm me-3"
  data-widget-css-solved="alert alert-success"
  data-widget-css-solved-body="d-flex align-items-center"
  data-widget-css-solved-icon="me-2"
  data-widget-css-solved-text="fw-bold"
  data-widget-css-failed="alert alert-warning"
  data-widget-css-failed-body="d-flex align-items-center"
  data-widget-css-failed-icon="me-2"
  data-widget-css-error="alert alert-danger"
  data-widget-css-error-body="d-flex align-items-center"
  data-widget-css-error-icon="me-2"
  data-widget-css-footer="d-flex justify-content-end align-items-center mt-3 mb-1 gap-2"
  data-widget-css-links="d-flex gap-2"
  data-widget-css-link-external="link-secondary link-offset-1"
  data-widget-css-link-reset="link-offset-1"
  >
</div>
```

Es folgen alle fünf möglichen Visualisierungen von RESTCaptcha in abstrahierter Form.

UI zur Visualisierung des interaktiven Modus:

``` html
<div id="${widgetId}-interactive" class="${data-widget-css-interactive}" role="alert">
    <div id="${widgetId}-interactive-body" class="${data-widget-css-interactiveBody}">
        <input id="${widgetId}-interactive-checkbox" type="checkbox" class="${data-widget-css-interactiveCheckBox}">
        <label id="${widgetId}-interactive-checklabel" for="${widgetId}-interactive-checkbox" class="${data-widget-css-interactiveCheckLabel}">
            // Text
        </label>
    </div>  
    <div id="${widgetId}-footer" class="${data-widget-css-footer}">
        <div id="${widgetId}-logo" class="${data-widget-css-logo}" role="img" aria-hidden="true">
            // RESTCaptcha Logo
        </div>
        <small id="${widgetId}-links" class="${data-widget-css-links}">
            // External Links
        </small>
    </div>
</div>
```

UI zur Visualisierung des Lösungsprozesses:

``` html
<div id="${widgetId}-solving" class="${data-widget-css-solving}" role="alert">
    <div id="${widgetId}-solving-body" class="${data-widget-css-solvingBody}">
        <div id="${widgetId}-solving-animation" class="${data-widget-css-solvingAnimation}" role="status" aria-hidden="true"></div>
        <div id="${widgetId}-solving-text" class="${data-widget-css-solvingText}">
            // Text
        </div>
    </div>
    <div id="${widgetId}-footer" class="${data-widget-css-footer}">
        <div id="${widgetId}-logo" class="${data-widget-css-logo}" role="img" aria-hidden="true">
            // RESTCaptcha Logo
        </div>
        <small id="${widgetId}-links" class="${data-widget-css-links}">
            // External Links
        </small>
    </div>
</div>
```

UI zur Visualisierung der erfolgreich validierten Challenge-Lösung:

``` html
<div id="${widgetId}-success" class="${data-widget-css-solved}" role="alert">
    <div id="${widgetId}-success-body" class="${data-widget-css-solvedBody}">
        <div id="${widgetId}-success-icon" class="${data-widget-css-solvedIcon}">
            // Icon
        </div>
        <div id="${widgetId}-success-text" class="${data-widget-css-solvedText}">
            // Text
        </div>
    </div>
    <div id="${widgetId}-footer" class="${data-widget-css-footer}">
        <div id="${widgetId}-logo" class="${data-widget-css-logo}" role="img" aria-hidden="true">
            // RESTCaptcha Logo
        </div>
        <small id="${widgetId}-links" class="${data-widget-css-links}">
            // External Links
        </small>
    </div>
</div>
```


UI zur Visualisierung der nicht erfolgreich gelösten Challenge-Lösung:

``` html
<div id="${widgetId}-failed" class="${data-widget-css-failed}" role="alert">
    <div id="${widgetId}-failed-body" class="${data-widget-css-failedBody}">
        <div id="${widgetId}-failed-icon" class="${data-widget-css-failedIcon}">
            // Icon
        </div>
        <div id="${widgetId}-failed-text" class="${data-widget-css-failedText}">
            // Error message
        </div>
    </div>
    <div id="${widgetId}-footer" class="${data-widget-css-footer}">
        <div id="${widgetId}-logo" class="${data-widget-css-logo}" role="img" aria-hidden="true">
            // RESTCaptcha Logo
        </div>
        <small id="${widgetId}-links" class="${data-widget-css-links}">
            // External Links
        </small>
    </div>
</div>
```

UI zur Visualisierung von API-Fehlern:

``` html
<div id="${widgetId}-error" class="${data-widget-css-error}" role="alert">
    <div id="${widgetId}-error-body" class="${data-widget-css-errorBody}">
        <div id="${widgetId}-error-icon" class="${data-widget-css-errorIcon}">
            // Icon
        </div>
        <div id="restcaptcha-error-text" class="${data-widget-css-errorText}">
            // Error message
        </div>
    </div>
    <div id="${widgetId}-footer" class="${data-widget-css-footer}">
        <div id="${widgetId}-logo" class="${data-widget-css-logo}" role="img" aria-hidden="true">
            // RESTCaptcha Logo
        </div>
        <small id="${widgetId}-links" class="${data-widget-css-links}">
            // External Links
        </small>
    </div>
</div>
```

### Externe Links

Bis zu 8 zusätzliche Links können angezeigt werden, z. B. für Verweise auf Datenschutz oder Impressum.

Beispiel:

```html
<div id="restcaptcha-widget"
  data-widget-external-link1-href="https://www.beispiel.de/restcaptcha"
  data-widget-external-link1-text="Über RESTCaptcha"
  data-widget-external-link2-href="https://www.beispiel.de/datenschutz"
  data-widget-external-link2-text="Datenschutz">
</div>
```

## Events & Callbacks

Es gibt zwei Ansätze, um Ereignisse zu behandeln: Programmatisch (Headless) oder deklarativ (via HTML-Attribute)

Programmatisch:

```js
const h = new HeadlessRestCaptcha(api, siteKey, 'de');

h.onStarted = () => console.log("Gestartet");
h.onSolved = (token, solution) => console.log("Gelöst:", token, solution);
h.onFailed = msg => alert(msg);
h.onError = e => console.error(e);
h.onReset = () => console.log("Zurückgesetzt");

h.solve();
```

Deklarativ:

```html
<script>
  function captchaStarted() { console.log("Gestartet"); }
  function captchaSolved(token, solution) { console.log("Gelöst:", token, solution); }
  function captchaFailed(msg) { alert(msg); }
</script>

<div id="restcaptcha-widget"
  data-api-baseurl="https://captcha.beispiel.de/v1/"
  data-sitekey="<Mein-Site-Schlüssel>"
  data-callback-started="captchaStarted"
  data-callback-solved="captchaSolved"
  data-callback-failed="captchaFailed">
  ...
</div>
```

## Automatische Initialisierung

Beim Laden des Dokuments (`DOMContentLoaded`) sucht das Skript automatisch nach einem Element mit der ID `restcaptcha-widget` und initialisiert es mit:

```js
new RestCaptcha("restcaptcha-widget");
```

Fehlt das Element, passiert nichts weiter.