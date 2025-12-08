Die Integration von RESTCaptcha in eine Webseite ist recht einfach. In dieser Dokumentation gehen wir von folgender Umgebung aus:

+ Der RESTCaptcha-Server ist installiert und unter der URL `https://captcha.beispiel.de` erreichbar.
+ Deine Webseite hat ein Backend, das Formularanfragen via `POST` entgegennehmen kann

Wir werden jetzt folgendes machen:

1. Das **RESTCaptcha-JS-Skript** in Deine Webseite **einfügen**.
2. Ein **leeres Container-Element** in Deine Webseite **einfügen** und dieses konfigurieren.
3. Im **Backend** Deiner Webseite die Challenge-Lösung entgegennehmen und mit Hilfe des **RESTCaptcha-Servers überprüfen** lassen.

## Skript einfügen

Füge das Widget-Skript in den `<head>`-Bereich oder ans Ende von `<body>` Deiner Webseite ein:

``` html
<script src="https://captcha.beispiel.de/restcaptcha.min.js"></script>
```

Das Skript sucht automatisch nach HTML-Elementen mit `id="restcaptcha-widget"` (außer im Headless-Modus, siehe unten) und initialisiert das RESTCaptcha-Widget.

## Container-Element einfügen

Füge auf Deiner Formularseite ein leeres HTML-Element ein, das vom RESTCaptcha-JS-Skript zum Rendern des RESTCaptcha-Widgets genutzt wird. Dieses Element musst Du mit passenden Attributen anreichern. Die wichtigsten wären:

+ `id` (Hallo, ich bin ein RESTCaptcha-Widget, der Standardwert ist `restcaptcha-widget`), 
+ `data-api-baseurl` (Wo befindet sich mein RESTCaptcha-Server?) 
+ `data-sitekey` (Authentifizierung).
+ `data-widget-mode` (Modus, der Standardwert ist `interactive`)

Das sieht dann in etwa so aus:

``` html
<div 
  id="restcaptcha-widget"
  data-api-baseurl="https://captcha.beispiel.de/v1/"
  data-sitekey="<Mein-Site-Schlüssel>"
  data-widget-mode="interactive">
  ...
</div>
``` 

Sobald die Challenge interaktiv oder automatisch gelöst wurde, fügt das RESTCaptcha-JS-Skript zwei zusätzliche aber unsichtbare `<input>`-Felder in Deine Webseite ein: Ein Feld für die Challenge-Lösung und ein anderes für ein kryptografisches Token zur Absicherung der Lösung.

Wenn der Benutzer die Daten des Web-Formulars an Deinen Server überträgt, werden dies Felder automatisch mitgesendet.

Es folgt ein minimales aber komplettes Beispiel für eine Webseite mit integriertem RESTCaptcha-Widget:

``` html
<!DOCTYPE html>
<head>
  <script async defer src="https://captcha.beispiel.de/restcaptcha.min.js"></script>
</head>
<body>
  <form id="login-form" method="POST" action="submit">
    <label for="username">Username</label>
    <input id="username" name="username" required>
    <label for="password">Password</label>
    <input id="password" type="password" name="password" required>
    <div
      id="restcaptcha-widget"
      data-api-baseurl="https://captcha.beispiel.de/v1/"
      data-sitekey="DEIN_SITE_KEY"
      data-widget-mode="interactive"
      data-callback-solved="onCaptchaSolved"
      data-callback-reset="onCaptchaReset">
    </div>
    <button id="loginButton" type="submit" disabled>Login</button>
  </form>
  <script>
    function onCaptchaSolved(token, solution) {
      // Wird aufgerufen, sobald der Nutzer die Aufgabe gelöst hat
      document.getElementById('loginButton').disabled = false;
    }
    function onCaptchaReset() {
      // Wird aufgerufen, wenn der Captcha-Status zurückgesetzt wird
      document.getElementById('loginButton').disabled = true;
    }
  </script>
</body>
</html>
```

Was passiert hier?

1. Das RESTCaptcha-Widget wird automatisch gerendert.

2. Wird die Challenge gestarted (durch Klicken des Nutzers), dauert es etwas, bis die Lösung gefunden ist.

3. Nach erfolgreicher Lösung werden die folgenden zwei Felder ins Formular eingefügt:

    + `captcha-token`
    + `captcha-solution`

4. Mit diesen beiden Werten verifizierst Du serverseitig die Lösung (siehe nächster Abschnitt).

## Serverseitige Verifikation

Die Verifikation auf Serverseite geschieht nach folgendem Prinzip:

1. Das Formular wird an Dein Backend gesendet. Es enthält u.a. die Felder `captcha-token` und `captcha-solution`.
2. Dein Backend ruft die RESTCaptcha-API auf, authentifiziert bzw. autorisiert sich mit `DEIN_SITE_KEY` und einem zusätzlichen `DEIN_SITE_SECRET` und lässt Token und Lösung prüfen.
3. Nur nach erfolgreicher Verifikation verarbeitest Du die eigentlichen Formulardaten weiter.

Es folgt ein minimales Beispiel in PHP:

```php
<?php
// POST-Werte aus dem Formular
$token    = $_POST['captcha-token']    ?? '';
$solution = $_POST['captcha-solution'] ?? '';

// Konfiguration
$siteKey    = 'DEIN_SITE_KEY';
$siteSecret = 'DEIN_SITE_SECRET';

// Optional (fürs Logging)
$callerIp = $_SERVER['REMOTE_ADDR'] ?? '';

// Request-Daten vorbereiten
$payload = [
  'siteSecret' => $siteSecret,
  'solution'   => $solution,
  'token'      => $token,
  'callerIp'   => $callerIp
];

// cURL initialisieren
$ch = curl_init('https://captcha.beispiel.de/v1/verify?siteKey=' . urlencode($siteKey));
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST           => true,
    CURLOPT_HTTPHEADER     => ['Content-Type: application/json'],
    CURLOPT_POSTFIELDS     => json_encode($payload),
    CURLOPT_TIMEOUT        => 10,
]);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
$error    = curl_error($ch);
curl_close($ch);

// Fehlerbehandlung
if ($error) {
    die("Verbindungsfehler bei der CAPTCHA-Verifikation: $error");
}

// Antwort auslesen
$data = json_decode($response, true);

// Erfolg, wenn HTTP 200 und status == "success"
$isVerified = ($httpCode === 200 && isset($data['status']) && $data['status'] === 'success');

if ($isVerified) {
  // OK > Nutzeraktion erlauben
} else {
  // Fehlgeschlagen > Formular mit Fehlermeldung anzeigen
}
?>
```

!!! warning "Wichtig!"

    Das **siteSecret** gehört nur auf Deinen Server. Niemals im Browser exponieren!

## Weitere Hilfen

Für die serverseitige Integration von RESTCaptcha stehen optional auch [Client-Bibliotheken](./libraries.md) für [PHP](https://www.php.net/) und [.NET](https://dotnet.microsoft.com/) zur Verfügung:

+ [openpotato/restcaptcha-client.net](https://github.com/openpotato/restcaptcha-client.net): Offizielle RESTCaptcha API .NET-Client-Bibliothek.
+ [openpotato/restcaptcha-client.php](https://github.com/openpotato/restcaptcha-client.php): Offizielle RESTCaptcha API PHP-Client-Bibliothek.

## Beispiele

Die Quellcodes für unsere beiden Live-Demos ([PHP-basiert](https://php-demo.restcaptcha.openpotato.org/) und [ASP\.NET-basiert](https://dotnet-demo.restcaptcha.openpotato.org/) ) sind bestimmt auch ganz interessant. Sie befinden sich im [GitHub-Repository von RESTCaptcha](https://github.com/openpotato/restcaptcha).
