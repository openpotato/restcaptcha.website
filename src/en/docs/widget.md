This reference documents all important public classes, properties, and methods, as well as the available HTML attributes (`data-*`) and events of the `restcaptcha.js` library.

## Overview

The RESTCaptcha JavaScript library provides the code required for client-side integration of RESTCaptcha into your own websites.

It comes in two variants:

* `restcaptcha.js`: The original, readable version.
* `restcaptcha.min.js`: A minified version with all unnecessary characters removed to reduce file size and improve browser load times.

The library consists of two main components:

1. `HeadlessRestCaptcha`: A pure JavaScript client without a user interface (headless). It communicates directly with the RESTCaptcha API, solves the cryptographic challenge, and returns the result via events or callbacks. This variant is suitable for integrating with your own UIs or framework solutions.

2. `RestCaptcha`: A visual widget built on top of the headless client that automatically renders a simple user interface in the browser. It can be freely configured via HTML attributes, and supports multiple languages and modes (interactive, automatic, or invisible).

## HeadlessRestCaptcha

A UI-less client that communicates directly with the RESTCaptcha API, solves a proof-of-work, and is driven via events (callbacks).

### Constructor

```js
new HeadlessRestCaptcha(apiBaseUrl, siteKey, language)
```

Parameters:

**`apiBaseUrl`**

:   Base URL of the RESTCaptcha backend (**required**)

**`siteKey`**

:   The RESTCaptcha site key (**required**)

**`language`**

:   Language code (e.g. `"en"`). Auto-detected if empty.

### Properties

**`HeadlessRestCaptcha.language`**

:   Effective language code.

### Events

**`HeadlessRestCaptcha.onStarted`**

:   Function invoked when solving starts.

**`HeadlessRestCaptcha.onSolved`**

:   Function invoked when the CAPTCHA has been solved successfully.

**`HeadlessRestCaptcha.onFailed`**

:   Function invoked on failures (e.g. invalid token, too many attempts).

**`HeadlessRestCaptcha.onError`**

:   Function invoked on unexpected errors.

**`HeadlessRestCaptcha.onReset`**

:   Function invoked after a reset.

### Methods

**`HeadlessRestCaptcha.solve()`**

:   Starts backend communication, retrieves a challenge, solves the proof-of-work (SHA-256/384/512), and on success calls `onSolved(token, solution)`.

**`HeadlessRestCaptcha.reset(token)`**

:   Invalidates a previously obtained token.

**`HeadlessRestCaptcha.isHeadless()`**

:   Checks whether the calling browser is most likely headless or automated.

## RestCaptcha

The visible CAPTCHA widget that automatically renders HTML and interacts with the integrated headless client.

### Constructor

```js
new RestCaptcha(widgetId = "restcaptcha-widget")
```

Parameter:

**`widgetId`**

:   The ID of the widget in the browser DOM (**required**)

## RestCaptchaProblemDetails

Implements the [RFC 9457 Problem Details](https://datatracker.ietf.org/doc/html/rfc9457) standard for structured API errors. Extends JavaScript’s `Error` class.

### Constructor

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

The parameter is an anonymous object with the corresponding properties (see the next section, “Properties”).

### Properties

**`RestCaptchaProblemDetails.type`**

:   URI describing the error type.

**`RestCaptchaProblemDetails.title`**

:   Short, human-readable summary.

**`RestCaptchaProblemDetails.status`**

:   HTTP status code.

**`RestCaptchaProblemDetails.detail`**

:   Detailed error description.

**`RestCaptchaProblemDetails.instance`**

:   URI identifying this error occurrence.

**`RestCaptchaProblemDetails.traceId`**

:   Server-side diagnostic ID.

**`RestCaptchaProblemDetails.errors`**

:   Field-level validation errors (optional).

### Instance methods

**`RestCaptchaProblemDetails.toString()`**

:   Returns the error as `[Status] Title: Detail`.

### Static methods

**`RestCaptchaProblemDetails.fromResponse(response, json)`**

:   Creates a `RestCaptchaProblemDetails` instance from a `fetch()` response and its JSON body.

## Widget `data-*` attributes

These HTML attributes control the widget’s behaviour and appearance.

### Basic configuration

**`data-api-baseurl`**

:    Base URL of the RESTCaptcha API (**required**)

**`data-sitekey`**

:    Public site key (**required**)

**`data-widget-language`**

:    Language code (`de`, `en`, `es`, `fr`, `it`, `pt`). By default, the script attempts to detect the language from the browser, falling back to `en`.

**`data-widget-mode`**

:    Mode. Possible values are `interactive` (default), `auto`, or `invisible`.

### Callback attributes

**`data-callback-started` > `() => void`**

:    Callback after the challenge has started on the client.

**`data-callback-solved`  > `(token, solution) => void`**

:    Callback after a successful verification has been reported.

**`data-callback-failed` > `(message) => void`**

:    Callback after an invalid verification has been reported.

**`data-callback-error` > `(error) => void`**

:    Callback after an API error has been reported.

**`data-callback-reset` > `(token) => void`**

:    Callback after the reset link has been clicked.

Example:

```html
<script>
  function onCaptchaSolved(token, solution) {
    console.log("Solved:", token, solution);
  }
</script>

<div id="restcaptcha-widget"
     data-api-baseurl="https://captcha.example.com/v1/"
     data-sitekey="<My-Site-Key>"
     data-widget-language="en"
     data-callback-solved="onCaptchaSolved">
</div>
```

### CSS class attributes

The widget allows full control over CSS classes via `data-widget-css-*` attributes.

Some attributes are specific to certain visual states (e.g. `data-widget-css-interactive`), while others are used across all states (e.g. `data-widget-css-footer`). In combination with the HTML nodes’ `id` values, this provides maximum flexibility.

Example for [Bootstrap 5](https://getbootstrap.com/):

```html
<div 
  id="restcaptcha-widget" 
  data-api-baseurl="https://captcha.example.com/v1/"
  data-sitekey="<My-Site-Key>"
  data-widget-language="en"
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

Below are all five possible RESTCaptcha visual states in abstract form.

UI for the interactive mode:

```html
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

UI for the solving state:

```html
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

UI for a successfully validated challenge:

```html
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

UI for a failed challenge:

```html
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

UI for API errors:

```html
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

### External links

You can display up to eight additional links, e.g. to privacy or legal information.

Example:

```html
<div id="restcaptcha-widget"
  data-widget-external-link1-href="https://www.example.com/restcaptcha"
  data-widget-external-link1-text="About RESTCaptcha"
  data-widget-external-link2-href="https://www.example.com/privacy"
  data-widget-external-link2-text="Privacy">
</div>
```

## Events & callbacks

There are two approaches to handling events: programmatic (headless) or declarative (via HTML attributes).

Programmatic:

```js
const h = new HeadlessRestCaptcha(api, siteKey, 'de');

h.onStarted = () => console.log("Started");
h.onSolved = (token, solution) => console.log("Solved:", token, solution);
h.onFailed = msg => alert(msg);
h.onError = e => console.error(e);
h.onReset = () => console.log("Reset");

h.solve();
```

Declarative:

```html
<script>
  function captchaStarted() { console.log("Started"); }
  function captchaSolved(token, solution) { console.log("Solved:", token, solution); }
  function captchaFailed(msg) { alert(msg); }
</script>

<div id="restcaptcha-widget"
  data-api-baseurl="https://captcha.example.com/v1/"
  data-sitekey="<My-Site-Key>"
  data-callback-started="captchaStarted"
  data-callback-solved="captchaSolved"
  data-callback-failed="captchaFailed">
  ...
</div>
```

## Automatic initialisation

On document load (`DOMContentLoaded`), the script automatically looks for an element with the ID `restcaptcha-widget` and initialises it with:

```js
new RestCaptcha("restcaptcha-widget");
```

If the element is missing, nothing further happens.
