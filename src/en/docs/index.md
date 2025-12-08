RESTCaptcha is an easy-to-integrate, privacy-friendly CAPTCHA solution that does not require users to solve image puzzles or maths problems.

Instead, RESTCaptcha uses a proof-of-work mechanism to verify human interaction with your form.

Features:

+ Stateless challenge with an HMAC-signed nonce (a one-time-use string)
+ Client-side proof-of-work puzzle (SHA-256, SHA-384, or SHA-512 hash below a defined threshold)
+ CDN-ready `restcaptcha.min.js` script with a configurable API endpoint
+ Optional IP reputation check via [AbuseDBIP](https://www.abuseipdb.com/) and/or [Spamhaus](https://www.spamhaus.org/) as an additional protective measure
+ Easy to integrate with [Node.js](https://nodejs.org/), [PHP](https://www.php.net/), [ASP.NET Core](https://dotnet.microsoft.com/apps/aspnet), or any other server technology
+ Fully customisable
+ Supports four different modes (interactive, automatic, invisible, headless)
+ Multilingual (currently: English, German, French, Italian, Portuguese, Spanish)
+ Build with [.NET](https://dotnet.microsoft.com/) and [JavaScript](https://developer.mozilla.org/docs/Web/JavaScript).

## Data Privacy

RESTCaptcha does not process or store any personal data:

+ No cookies or server-side tracking technologies are used.
+ The browser fingerprint check is performed entirely on the client side.
+ When generating the challenge, the reputation of the client's IP address is optionally checked.
+ During verification, only the data necessary for the technical process are processed: the anonymous challenge solution and (depending on configuration) the client’s IP address for logging purposes.
+ RESTCaptcha can be fully operated on your own servers, ensuring that all data remain under your own control.
+ The source code is open source and can be reviewed or audited at any time.

## How it works

If the term *CAPTCHA* doesn’t mean much to you, have a look at the chapter [What are CAPTCHAs?](captcha.md) first.

In simple terms, RESTCaptcha works as follows:

1. You integrate the RESTCaptcha widget into your website — typically into a registration or contact form.
2. The user visiting your website receives a token from the RESTCaptcha server (this happens transparently in the background). Their web browser must then solve a small computational task by clicking a checkbox (optionally, this can also happen without any click).
3. Once the user has filled in the form, it is submitted to your server (the one hosting your website). The token and the puzzle solution are sent along with the form data.
4. Before processing the submitted data, your server must verify the token and the puzzle solution by making an API request to the RESTCaptcha server. If verification succeeds, everything is fine; otherwise, your server should assume the request came from a bot.

## Live demos

The following live demos are available:

<div class="grid cards" markdown>

-   :material-language-php:{ .lg .middle } &nbsp;
    __PHP-Demo__

    ---

    We have implemented a small website using PHP and Bootstrap 5 to demonstrate the use of RESTCaptcha.

    [:octicons-arrow-right-24: Visit the site](https://php-demo.restcaptcha.openpotato.org/)

-   :material-dot-net:{ .lg .middle } &nbsp;
    __ASP\.NET-Demo__

    ---

    We have built the same website using ASP.NET Core and Bootstrap 5.

    [:octicons-arrow-right-24: Visit the site](https://dotnet-demo.restcaptcha.openpotato.org/)

</div>

The source code for both demos is available in the [RESTCaptcha GitHub repository](https://github.com/openpotato/restcaptcha).
