Integrating RESTCaptcha into a website is quite straightforward. This guide assumes the following environment:

+ The RESTCaptcha server is installed and accessible at `https://captcha.example.com`.
+ Your website has a backend capable of handling form submissions via `POST`.

We’ll now go through the following steps:

1. **Insert the RESTCaptcha JS script** into your website.
2. **Add and configure an empty container element** to host the widget.
3. **Receive and verify** the challenge solution on your **backend** using the RESTCaptcha server.

## Adding the script

Add the widget script to either the `<head>` section or the end of the `<body>` of your website:

``` html
<script src="https://captcha.example.com/restcaptcha.min.js"></script>
```

The script automatically searches for HTML elements with `id="restcaptcha-widget"` (except in headless mode; see below) and initialises the RESTCaptcha widget.

## Adding the container element

Insert an empty HTML element on your form page. This element will be used by the RESTCaptcha JS script to render the widget. You need to include the following key attributes:

* `id` — identifies the RESTCaptcha widget (default: `restcaptcha-widget`)
* `data-api-baseurl` — specifies the location of your RESTCaptcha server
* `data-sitekey` — defines your authentication key
* `data-widget-mode` — sets the display mode (default: `interactive`)

Example:

``` html
<div 
  id="restcaptcha-widget"
  data-api-baseurl="https://captcha.example.com/v1/"
  data-sitekey="<MY_SITE_KEY>"
  data-widget-mode="interactive">
  ...
</div>
```

Once the challenge is solved (either interactively or automatically), the RESTCaptcha script inserts two hidden `<input>` fields into your form:
one containing the challenge solution, and another containing a cryptographically signed token.

When the user submits the form, these fields are automatically included in the submission.

Here’s a minimal but complete example of a web page with an integrated RESTCaptcha widget:

``` html
<!DOCTYPE html>
<head>
  <script async defer src="https://captcha.example.com/restcaptcha.min.js"></script>
</head>
<body>
  <form id="login-form" method="POST" action="submit">
    <label for="username">Username</label>
    <input id="username" name="username" required>
    <label for="password">Password</label>
    <input id="password" type="password" name="password" required>
    <div
      id="restcaptcha-widget"
      data-api-baseurl="https://captcha.example.com/v1/"
      data-sitekey="YOUR_SITE_KEY"
      data-widget-mode="interactive"
      data-callback-solved="onCaptchaSolved"
      data-callback-reset="onCaptchaReset">
    </div>
    <button id="loginButton" type="submit" disabled>Login</button>
  </form>
  <script>
    function onCaptchaSolved(token, solution) {
      // Called when the user has successfully solved the challenge
      document.getElementById('loginButton').disabled = false;
    }
    function onCaptchaReset() {
      // Called when the CAPTCHA status is reset
      document.getElementById('loginButton').disabled = true;
    }
  </script>
</body>
</html>
```

What happens here?

1. The RESTCaptcha widget is automatically rendered.

2. When the challenge starts (e.g. after a user click), it takes a few moments to compute the solution.

3. Once solved, two additional hidden form fields are added:

   + `captcha-token`
   + `captcha-solution`

4. These two values are then verified on the server side (see next section).

## Server-side verification

The verification process on the server follows this general pattern:

1. The form is submitted to your backend and includes the fields `captcha-token` and `captcha-solution`.
2. Your backend calls the RESTCaptcha API, authenticating with your `YOUR_SITE_KEY` and `YOUR_SITE_SECRET` to verify the token and solution.
3. Only if verification succeeds should the backend proceed with processing the form data.

Here’s a minimal PHP example:

``` php
<?php
// POST values from the form
$token    = $_POST['captcha-token']    ?? '';
$solution = $_POST['captcha-solution'] ?? '';

// Configuration
$siteKey    = 'YOUR_SITE_KEY';
$siteSecret = 'YOUR_SITE_SECRET';

// Optional (for logging)
$callerIp = $_SERVER['REMOTE_ADDR'] ?? '';

// Prepare request payload
$payload = [
  'siteSecret' => $siteSecret,
  'solution'   => $solution,
  'token'      => $token,
  'callerIp'   => $callerIp
];

// Initialise cURL request
$ch = curl_init('https://captcha.example.com/v1/verify?siteKey=' . urlencode($siteKey));
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

// Handle errors
if ($error) {
    die("Connection error during CAPTCHA verification: $error");
}

// Parse response
$data = json_decode($response, true);

// Success if HTTP 200 and status == "success"
$isVerified = ($httpCode === 200 && isset($data['status']) && $data['status'] === 'success');

if ($isVerified) {
  // OK → proceed with form processing
} else {
  // Verification failed → show error message
}
?>
```

!!! warning "Important"

    The **siteSecret** must only ever be stored on your server. Never expose it in the browser or client-side code!

## Additional resources

For server-side integration, official [client libraries](./libraries.md) are also available for [PHP](https://www.php.net/) and [.NET](https://dotnet.microsoft.com/):

+ [openpotato/restcaptcha-client.net](https://github.com/openpotato/restcaptcha-client.net): Official RESTCaptcha API .NET client library.
+ [openpotato/restcaptcha-client.php](https://github.com/openpotato/restcaptcha-client.php): Official RESTCaptcha API PHP client library.

## Examples

The source code for both live demos ([PHP-based](https://php-demo.restcaptcha.openpotato.org/) and [ASP.NET-based](https://dotnet-demo.restcaptcha.openpotato.org/)) can also serve as a helpful reference You can find them in the [RESTCaptcha GitHub repository](https://github.com/openpotato/restcaptcha).
