# Was sind CAPTCHAs?

CAPTCHAs (*Completely Automated Public Turing tests to tell Computers and Humans Apart*) sind Challenge-Response-Tests, die auf Webseiten verwendet werden, um echte Nutzer von automatisierten Bots zu unterscheiden. Sie verlangen in der Regel vom Nutzer eine Aufgabe, die für Menschen leicht, für Computersysteme jedoch schwierig zu lösen sind (z.B. einen verzerrten Text zu lesen oder Objekte in Bildern zu identifizieren). Das Hauptziel ist es, bösartige Software daran zu hindern, Online-Dienste zu missbrauchen (z. B. Spam in Formulare einzutragen, Daten abzugreifen oder betrügerisch Konten zu erstellen). 

## Warum sind CAPTCHAs notwendig?

Der grundlegende Zweck eines CAPTCHAs ist es, automatisierten Missbrauch von Online-Systemen zu blockieren, während legitime menschliche Benutzer durchgelassen werden. Ohne CAPTCHAs oder ähnliche Formen der Mensch-Verifikation könnten bösartige Bots:

+ Online-Formulare, Kommentare und E-Mails mit Spam oder schädlichen Links überfluten.

+ Tickets oder Produkte aufkaufen, indem sie Käufe schneller automatisieren als echte Kunden.

+ Credential-Stuffing oder Brute-Force-Angriffe auf Login-Seiten durchführen.

+ Massiv Konten erstellen (für Spam, Fake-Bewertungen oder Betrug) auf Plattformen mit kostenloser Registrierung.

CAPTCHAs schaffen Reibung für diese automatisierten Skripte. Wenn Nutzer beispielsweise ein verzerrtes Bild interpretieren oder ein Rätsel lösen müssen, kann das einen Bot erheblich verlangsamen oder abschrecken, da er menschenähnliche Wahrnehmung bräuchte, um fortzufahren. So helfen CAPTCHAs sicherzustellen, dass nur echte Personen kritische Aktionen wie eine Kontoanmeldung oder das Senden eines Formulars ausführen. 

Ganz wichtig: CAPTCHAs sind ein grobes Instrument und keine Allzwecklösung. Sie stellen eine Belastung für alle Nutzer dar, und motivierte Angreifer werden immer Wege finden, sie zu umgehen. Dennoch bleiben CAPTCHAs eine effektive erste Verteidigungslinie, um opportunistischen Missbrauch zu verhindern, indem sie automatisierte Angriffe teuer oder unpraktisch machen.

## Traditionelle CAPTCHA-Methoden

Frühe und "klassische" CAPTCHAs präsentierten in der Regel explizite Rätsel, die Nutzer lösen mussten. Diese traditionellen CAPTCHAs lassen sich in folgende gängige Muster unterteilen:

+ **Verzerrte Text-CAPTCHAs**: Die ursprüngliche Standardform eines CAPTCHAs zeigt ein Bild mit Text, der verzerrt, verrauscht oder verfremdet ist, und fordert den Benutzer auf, die abgebildeten Zeichen einzugeben. 

    Die Verzerrung nutzt Lücken in der maschinellen Bildverarbeitung aus: Menschen können die Buchstaben meist trotz Störungen erkennen, Maschinen hatten (zumindest historisch) Schwierigkeiten bei Segmentierung und Erkennung. Zum Beispiel könnte das Wort "klappsack" mit unregelmäßigen Buchstaben und Hintergrundrauschen dargestellt werden, das OCR-Software verwirrt. 
    
    Frühe CAPTCHAs wie die von Yahoo und der Carnegie Mellon University nutzten diese Methode. Mit der Zeit mussten sie schwieriger gestaltet werden (z. B. welligerer Text, unruhige Hintergründe), da sich OCR-Algorithmen verbesserten. Um 2012 lösten Menschen Text-CAPTCHAs mit ca. 90–95 % Genauigkeit in ca. 10 Sekunden, aber die Erfolgsquote sank, je schwieriger die Rätsel wurden.

+ **Bildbasierte CAPTCHAs**: Anstelle von Buchstaben verwenden einige CAPTCHAs Bilder. Nutzer müssen dabei Bilder identifizieren, die ein bestimmtes Kriterium erfüllen – etwa "Wählen Sie alle Bilder mit Ampeln aus" oder "Klicken Sie auf alle Katzenfotos". 

    Die Idee dahinter ist, dass Objekterkennung für KI schwieriger ist als Texterkennung. Googles reCAPTCHA v2 (2014 eingeführt) machte das Bildraster mit Straßenschildern, Autos, Schaufenstern usw. populär und nutzte Googles riesige Bilddatenbanken. 
    
    Ein früheres Beispiel war [Microsofts Asirra](https://www.microsoft.com/en-us/research/publication/asirra-a-captcha-that-exploits-interest-aligned-manual-image-categorization/) (2007), das 12 Fotos von Haustieren zeigte und den Nutzer aufforderte, alle Katzen auszuwählen. Asirra berichtete von 99,6 % menschlicher Genauigkeit in unter 30 Sekunden und wurde subjektiv als angenehmer empfunden als das Entziffern von Buchstaben. Obwohl Asirra 2014 eingestellt wurde, lebt der Ansatz in modernen bildbasierten CAPTCHAs weiter.

+ **Einfache Fragen oder Mathematik-CAPTCHAs**: Manche Seiten nutzen einfache Fragen (z. B. "Welche Farbe hat der Himmel?") oder Rechenaufgaben ("3 + 5 = ?") als CAPTCHAs. Diese sind für die meisten Menschen leicht und einfach zu beantworten. Sie werden manchmal MAPTCHAs (für Math CAPTCHAs) genannt. Ihre Sicherheit ist jedoch schwach, da Bots mit kleinem Wissensschatz oder OCR schnell angepasst werden können. Für Szenarien mit geringem Risiko oder Barrierefreiheit sind sie ausreichend, ansonsten leicht zu umgehen.

+ **Audio-CAPTCHAs**: Zur Unterstützung sehbehinderter Nutzer bieten viele CAPTCHA-Systeme eine Audio-Alternative. Dabei wird typischerweise eine verzerrte Zahlen- oder Buchstabenfolge über Hintergrundgeräusche abgespielt, die der Nutzer eintippen muss. 

    In der Praxis sind Audio-CAPTCHAs für viele sehr schwierig. Studien zeigen, dass blinde Nutzer bei Audio-CAPTCHAs nur ca. 45 % Erfolgsquote hatten, mit durchschnittlich über einer Minute Bearbeitungszeit. 
    
    Ironischerweise konnten Forscher viele Audio-CAPTCHAs softwareseitig (mittels Signalverarbeitung und Spracherkennung) leichter knacken als visuelle Varianten. Daher sind Audio-CAPTCHAs zwar wichtig für Barrierefreiheit, stellen aber eine Schwachstelle in Sicherheit und Nutzbarkeit dar.

+ **CAPTCHA-Varianten und Spiele**: Es gibt Dutzende weitere Varianten. Manche nutzen Logikrätsel oder Quizfragen ("Beantworte ein einfaches Rätsel"), interaktive Spiele (Teile bewegen, Muster nachzeichnen) oder fordern Nutzer auf, eine geometrische Figur zu zeichnen. 

    Ein experimentelles CAPTCHA ließ Nutzer kurze Spiele spielen, bei denen bewegte Objekte identifiziert werden mussten. Microsofts Forschungsprototyp PixelPlotter ließ Nutzer Punkte in einem Bild verbinden. 
    
    Diese Varianten sind weniger verbreitet, zeigen aber die Bandbreite menschlicher Aufgaben, die als CAPTCHA genutzt werden. Wichtig ist, dass die Aufgabe für durchschnittliche Menschen einfach, für automatisierte Skripte jedoch schwer ist.

## Alternativen zu traditionellen CAPTCHAs

Über die klassischen (Text- oder Bildrätsel) hinaus gibt es alternative Ansätze und Dienste, die dasselbe Problem der Bot-Abwehr adressieren. Manche können zusätzlich oder statt der klassischen CAPTCHAs eingesetzt werden:

+ **Google reCAPTCHA (v2 und v3):** Googles [reCAPTCHA](https://developers.google.com/recaptcha) ist der am weitesten verbreitete Dienst. 

    ReCAPTCHA v2 (2014) führte das berühmte "Ich bin kein Roboter"-Kontrollkästchen ein, das beim Anklicken unsichtbare Analysen des Nutzers durchführt (Cookies, Browser-Fingerprint, Google-Kontoaktivität etc.), um die Wahrscheinlichkeit für menschliches Verhalten einzuschätzen. Bei geringer Sicherheit zeigt es dann klassische Bild- oder Audio-Rätsel. Dies beschleunigte die Verifikation für die meisten legitimen Nutzer (oft nur ein Klick), weshalb Google es auch "No CAPTCHA reCAPTCHA" nannte. Kritik kam jedoch auf, da das Verfahren stark vom Google-Login-Status abhängt. Ein Chrome-Nutzer, der bei Gmail angemeldet ist, kommt meist sofort durch, während ein datenschutzbewusster Firefox-Nutzer mit Tracking-Schutz viele Bildrätsel lösen muss. 

    reCAPTCHA v3 (2018) wiederum ist ein vollständig unsichtbares, score-basiertes System. Website-Betreiber müssen die Scores selbst interpretieren und Aktionen festlegen, was komplex sein kann. 
    
    Beide Versionen sind bis zu einem hohen Volumen kostenlos, doch Google setzte ein Limit von 1 Mio. Aufrufen pro Monat durch (darüber kostenpflichtige Enterprise-Tarife). Das führte dazu, dass große Webseiten (z. B. Cloudflare) aus Kostengründen und wegen Datenschutzbedenken Alternativen suchten.

+ **hCaptcha:** [hCaptcha](https://www.hcaptcha.com/) wurde ab 2019–2020 populär als Ersatz für reCAPTCHA und positioniert sich als datenschutzfreundlicher und für Seitenbetreiber lohnend. Die Herausforderungen ähneln reCAPTCHA v2 (Bildklassifikationsaufgaben), aber ein wesentlicher Unterschied ist, dass hCaptcha Website-Betreiber für die Bildannotation durch Nutzer bezahlt. Unternehmen, die Trainingsdaten benötigen, bezahlen hCaptcha. Die Arbeit wird in Form von CAPTCHAs an Nutzer weitergegeben, wobei Seitenbetreiber am Umsatz beteiligt werden. 

    Cloudflare wechselte 2020 von Google zu hCaptcha, als Googles Preise sich änderten, auch um Googles Tracking einzuschränken. In der Nutzererfahrung gelten hCaptcha-Aufgaben teils als schwieriger oder ungewohnter (z. B. weniger offensichtliche Objekte). Aber auch hier gilt das Prinzip: Wenn man sicher menschlich ist, sieht man keine Herausforderung; wenn nicht, muss man eine lösen. 
    
    hCaptcha bietet auch einen unsichtbaren Modus und Barrierefreiheitsoptionen (Audio oder Logikrätsel).

+ **Cloudflare Turnstile:** Cloudflare stellte 2022 [Turnstile](https://www.cloudflare.com/application-services/products/turnstile/) vor, ein CAPTCHA-as-a-Service, das auch von Nicht-Cloudflare-Kunden genutzt werden kann. Standardmäßig unsichtbar, laufen Hintergrundprüfungen, sobald ein Nutzer mit einer geschützten Seite interagiert. Nur wenn nötig, wird eine kleine Aufgabe angezeigt (z. B. Bild drehen). 

    Turnstile legt großen Wert auf Datenschutz: Es verarbeitet alle Verifikationen im Browser und speichert keine personenbezogenen Daten auf Cloudflare-Servern. Damit erfüllt es strenge DSGVO-Anforderungen, die Googles CAPTCHA problematisch machen. Die Aufgaben sind schnell und sprachunabhängig, z. B. ein Bild aufrichten. 
    
    Manche Sicherheitsforscher bemerken, dass diese Aufgaben für Bots leichter zu lösen sein könnten, aber Turnstile versucht dies mit zusätzlichen Signalen und Updates auszugleichen. 

+ **FriendlyCaptcha:** [FriendlyCaptcha](https://friendlycaptcha.com/) verfolgt einen ganz anderen Ansatz: Proof-of-Work-Puzzles statt menschlicher Aufgaben. Der Browser des Nutzers muss beim Besuch einer Seite ein kryptografisches Puzzle lösen. Das läuft im Hintergrund, der Nutzer sieht nur eine kurze Ladeanzeige. 

    Für Bots ist dies im großen Stil teuer, für Menschen bedeutet es nur Sekunden Verzögerung. Nutzer müssen nichts klicken oder tippen, was es extrem barrierefrei und nutzerfreundlich macht. Verdächtige Nutzer bekommen schwerere Aufgaben, aber ebenfalls ohne Interaktion. 

    FriendlyCaptcha erhebt keine personenbezogenen Daten, was strengen Datenschutzauflagen entspricht. Die Schwäche: Die Sicherheit basiert darauf, dass Angreifer nicht genug Rechenleistung einsetzen können. Auf sehr schwachen Geräten kann es zudem Leistung kosten. Dennoch ist es als Teil einer mehrschichtigen Verteidigung attraktiv.

+ **Device Fingerprinting & Reputationssysteme:** Manche Dienste verzichten ganz auf explizite Rätsel und nutzen Geräte-/Browser-Fingerprinting und IP-Reputation, um Bots zu erkennen. 

    Wenn ein Fingerprint einem bekannten Headless-Browser ähnelt, wird die Anfrage still blockiert oder verlangsamt. Oft wird dies kombiniert: Bekannte schlechte Anfragen werden blockiert oder mit schwerem CAPTCHA belegt, bekannte gute Nutzer erhalten kein CAPTCHA oder nur einfache Prüfungen. 
    
    Vorteil: Keine Nutzerbelastung. Nachteil: Mögliche *False Positives* (ungewöhnliche, aber legitime Nutzer) und ständige Anpassung nötig. 
    
    Solche Verfahren sind meist Teil mehrschichtiger Systeme wie reCAPTCHA v3.

+ **Proof-of-Work-Challenges:** Über FriendlyCaptcha hinaus wird Proof-of-Work als Zugangskontrolle diskutiert. Manche Webseiten erfordern eine kleine Rechenaufgabe (Hash-Berechnung) pro Anfrage. Dies kann per JavaScript geschehen. 

    Verbraucher-Webseiten setzen dies noch selten ein, häufiger wird es für API-Drosselung genutzt. Ziel ist es, massenhafte Anfragen teuer zu machen. Nachteil: langsame Geräte leiden, und Angreifer mit viel Rechenleistung können es aushebeln.

+ **Honeypot-Felder und Zeit-Tests:** Eine einfache Alternative ist ein Honeypot-Feld (Honigtopf). Das ist ein unsichtbares Formularfeld, das Menschen nicht ausfüllen, Bots jedoch schon. So kann der Server Bots erkennen und blockieren. 

    Ähnlich kann die Zeit zwischen Laden und Absenden gemessen werden. Wenn ein Formular beispielsweise innerhalb einer Sekunde abgeschickt wird, war es vermutlich ein Bot. 
    
    Diese Tests sind einfach und belasten Nutzer nicht, stoppen aber nur einfache Bots. Fortgeschrittene Angreifer sind in der Lage, diese zu umgehen.

## Abgrenzung zu RESTCaptcha

RESTCaptcha implementiert eine Kombination aus IP-Reputationsüberprüfungen, Proof-of-Work-Challenge, Analyse von Zeitmetriken und lokalem Device Fingerprinting. 

Um es vorweg zu nehmen: RESTCaptcha ist keine revolutionäre Neuerung ins Sachen CAPTCHA-Architektur. Vielmehr ging es uns darum, folgende Anforderungen umzusetzen:

+ Eine **Open-Source-Lösung**, die komplett konfigurierbar ist und von jedermann selbst gehostet werden kann (Stichwort: Digitale Souveränität)

+ Eine **datenschutzfreundliche, DSGVO-kompatible** Implementation. Also keine Speicherung von personenbezogenen Daten.

+ Eine sehr **einfache Integration** in bestehende Webseiten, ohne größere Programmierarbeit.

+ Eine Nutzererfahrung, die **angenehm und möglichst barrierefrei** ist. Also keine Bilderrätsel, verzerrten Texte oder mathematischen Rechenaufgaben, die man vor lauter Stress auch noch falsch löst.
