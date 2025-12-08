Eine CAPTCHA-Lösung (egal welche) kann immer nur *ein* Baustein in der Absicherungsstrategie Deiner Webseite sein und sollte mit zusätzlichen Maßnahmen kombiniert werden. 

Hier ein paar Anregungen:

+ **Content Security Policy (CSP)**: Eine Sicherheitsfunktion moderner Browser, mit der Du festlegen kannst, welche Inhalte (Skripte, Styles, Bilder, iFrames, etc.) auf Deiner Website geladen und ausgeführt werden dürfen. 

    *Ziel*: Verhinderung von Cross-Site Scripting (XSS) und Dateninjektionen.

+ **Rate-Limiting**: Ein Funktion, die dafür sorgt, dass ein Client (z. B. ein Benutzer, ein Browser oder ein Bot) nicht zu viele Anfragen in zu kurzer Zeit an Deinen Server senden kann. 

    *Ziel*: Verhinderung von DoS-Attacken. 

+ **(Dynamisches) IP-Blacklisting**: Eine Funktion, mit der Du bestimmte IP-Adressen oder IP-Bereiche blockieren kannst, um zu verhindern, dass sie auf Deinen Server zugreifen. Mit Werkeugen wie [Fail2Ban](https://github.com/fail2ban/fail2ban) kannst Du solche Listen dynamisch verwalten, in dem Du IP-Adressen erst dann blockierst, wenn sie ein bestimmtes Verhalten an den Tag legen (z.B. mehrfaches falsches Anmelden in kurzer Zeit). 

    *Ziel*: Blockieren von Clients mit verdächtigem Verhaltensmuster.

+ **TLS Hardening**: Neben der selbstverständlichen Tatsache, dass Deine Webseite unter HTTPS zu erreichen sein sollte, solltest Du auch dafür sorgen, dass nur TLS 1.2 und TLS 1.3 angeboten und keine veralteten bzw. unsicheren kryptografischen Verfahren genutzt werden. 

    *Ziel*: Sichere Verschlüssslung der Kommunikation

+ **HTTP Strict Transport Security (HSTS)**: Eine Funktion, die den Browser dazu zwingt, Deine Website ausschließlich über HTTPS aufzurufen und nicht mehr über unsichere HTTP-Verbindungen. 

    *Ziel*: Schutz vor dem berühmten Man-in-the-Middle.
