# What are CAPTCHAs?

CAPTCHAs (*Completely Automated Public Turing tests to tell Computers and Humans Apart*) are challenge–response tests used on websites to distinguish real users from automated bots. They usually require the user to complete a task that is easy for humans but difficult for computer systems — for example, reading distorted text or identifying objects in images. The main goal is to prevent malicious software from abusing online services (e.g. submitting spam through forms, scraping data, or fraudulently creating accounts).

## Why are CAPTCHAs necessary?

The fundamental purpose of a CAPTCHA is to block automated abuse of online systems while allowing legitimate human users through. Without CAPTCHAs or similar human verification mechanisms, malicious bots could:

* Flood online forms, comments, and emails with spam or harmful links.
* Buy up tickets or products by automating purchases faster than real customers.
* Perform credential-stuffing or brute-force attacks on login pages.
* Mass-create accounts (for spam, fake reviews, or fraud) on platforms with free registration.

CAPTCHAs create friction for these automated scripts. When users are required to interpret a distorted image or solve a puzzle, this can significantly slow down or deter bots, as they would need human-like perception to continue. CAPTCHAs therefore help ensure that only real people perform critical actions such as signing in or submitting a form.

Importantly, CAPTCHAs are a blunt instrument and not a universal solution. They create friction for all users, and motivated attackers will always find ways around them. Nevertheless, CAPTCHAs remain an effective first line of defence to prevent opportunistic abuse by making automated attacks costly or impractical.

## Traditional CAPTCHA methods

Early or “classic” CAPTCHAs typically presented explicit puzzles that users had to solve. These traditional CAPTCHAs can be grouped into several common types:

* **Distorted text CAPTCHAs:** The original standard CAPTCHA type showed an image containing text that was distorted, noisy, or warped, asking users to type in the displayed characters.

     The distortion exploited weaknesses in machine vision: humans could still recognise the letters despite interference, while computers (at least historically) struggled with segmentation and recognition. For example, the word “klappsack” might appear with irregular letters and noisy backgrounds that confuse OCR software.

     Early CAPTCHAs, such as those from Yahoo and Carnegie Mellon University, used this approach. Over time, they had to become more difficult (wavier text, busier backgrounds) as OCR algorithms improved. Around 2012, humans solved text CAPTCHAs with 90–95% accuracy in about 10 seconds, but success rates dropped as the puzzles became harder.

* **Image-based CAPTCHAs:** Instead of letters, some CAPTCHAs use images. Users must identify pictures that match a criterion, such as “Select all images with traffic lights” or “Click on all the cats.”

     The idea is that object recognition is harder for AI than text recognition. Google’s reCAPTCHA v2 (introduced in 2014) popularised the image grid with street signs, cars, and shopfronts, using Google’s vast image databases.

     An earlier example was [Microsoft’s Asirra](https://www.microsoft.com/en-us/research/publication/asirra-a-captcha-that-exploits-interest-aligned-manual-image-categorization/) (2007), which displayed 12 pet photos and asked users to select all the cats. Asirra reported 99.6% human accuracy within 30 seconds and was perceived as more pleasant than deciphering text. Although discontinued in 2014, its approach lives on in modern image CAPTCHAs.

* **Simple question or maths CAPTCHAs:** Some websites use simple questions (e.g. “What colour is the sky?”) or arithmetic problems (“3 + 5 = ?”). These are easy for most humans to answer and are sometimes called MAPTCHAs (Math CAPTCHAs). Their security is weak, however, as bots with basic logic or OCR can solve them easily. They may be sufficient for low-risk or accessibility scenarios but are otherwise easy to bypass.

* **Audio CAPTCHAs:** To support visually impaired users, many CAPTCHA systems offer an audio alternative. Typically, a distorted sequence of numbers or letters is played over background noise, which the user must type in.

     In practice, audio CAPTCHAs are difficult for many people. Studies show that blind users succeed only about 45% of the time, with average completion times exceeding one minute.

     Ironically, researchers have found that software using signal processing and speech recognition can sometimes solve audio CAPTCHAs more easily than visual ones. Thus, while audio CAPTCHAs are important for accessibility, they also represent a weakness in both usability and security.

* **CAPTCHA variants and games:** There are dozens of other variants. Some use logic or quiz questions (“Answer a simple riddle”), interactive games (dragging shapes, drawing patterns), or ask users to reproduce a geometric figure.

     Experimental CAPTCHAs included short games requiring users to identify moving objects. Microsoft’s prototype PixelPlotter had users connect dots in an image.

     These are less common but demonstrate the wide range of human tasks that can serve as CAPTCHAs. The key idea is that the task should be easy for an average person but hard for automated scripts.

## Alternatives to traditional CAPTCHAs

Beyond the classic text or image puzzles, there are alternative approaches and services addressing the same bot-prevention problem. Some can be used alongside or instead of traditional CAPTCHAs:

* **Google reCAPTCHA (v2 and v3):** Google’s [reCAPTCHA](https://developers.google.com/recaptcha) is the most widely used service.

     ReCAPTCHA v2 (2014) introduced the famous “I’m not a robot” checkbox, which performs invisible background analyses (cookies, browser fingerprint, Google account activity, etc.) to estimate the likelihood of human behaviour. If confidence is low, it shows a classic image or audio puzzle. This sped up verification for most legitimate users (often just one click), hence Google’s name “No CAPTCHA reCAPTCHA.”

     Critics pointed out that its effectiveness depended heavily on Google account status — a Chrome user signed into Gmail often passed immediately, while a privacy-conscious Firefox user with tracking protection might face numerous image puzzles.

     ReCAPTCHA v3 (2018) is a fully invisible, score-based system. Website owners must interpret scores and decide actions themselves, which can be complex.

     Both versions are free up to a high volume, but Google later introduced a limit of 1 million monthly requests, with paid enterprise plans beyond that. This prompted major sites (e.g. Cloudflare) to seek alternatives due to cost and privacy concerns.

* **hCaptcha:** [hCaptcha](https://www.hcaptcha.com/) emerged around 2019–2020 as a privacy-friendly, revenue-sharing alternative to reCAPTCHA. The challenges are similar (image classification tasks), but a key difference is that hCaptcha pays website owners for users’ image labelling work. Companies needing training data pay hCaptcha, which distributes the work through CAPTCHAs and shares revenue with site operators.

     Cloudflare switched from Google to hCaptcha in 2020, partly due to Google’s pricing and to reduce tracking. Users often find hCaptcha challenges harder or less intuitive (e.g. less obvious objects), but the same rule applies: genuine users often see no challenge, while suspicious ones do.

     hCaptcha also offers an invisible mode and accessibility options (audio or logic puzzles).

* **Cloudflare Turnstile:** Introduced in 2022, [Turnstile](https://www.cloudflare.com/application-services/products/turnstile/) is a CAPTCHA-as-a-Service available even to non-Cloudflare customers. It runs background checks as soon as a user interacts with a protected page and only displays a small task (e.g. rotating an image) if needed.

     Turnstile emphasises privacy: all verification runs in the browser, and no personal data are stored on Cloudflare’s servers. This ensures GDPR compliance, which is a major issue for Google’s CAPTCHA. The challenges are quick and language-independent, such as orienting an image correctly.

     Some security researchers note that these tasks may be easier for bots, but Turnstile compensates with additional signals and updates.

* **FriendlyCaptcha:** [FriendlyCaptcha](https://friendlycaptcha.com/) takes a completely different approach: proof-of-work puzzles instead of human tasks. The user’s browser must solve a small cryptographic puzzle when visiting a page. This runs in the background, and the user only sees a short loading indicator.

     For bots, large-scale computation becomes expensive; for humans, it’s just a short delay. Users don’t need to click or type anything, making it highly accessible and user-friendly. Suspicious users receive harder tasks, but still without interaction.

     FriendlyCaptcha collects no personal data, ensuring full GDPR compliance. The downside is that security relies on attackers lacking sufficient computing power, and very weak devices may experience minor slowdowns. Nevertheless, it’s an attractive option as part of a layered defence.

* **Device fingerprinting and reputation systems:** Some services skip explicit puzzles altogether, using device/browser fingerprinting and IP reputation to detect bots.

     If a fingerprint matches a known headless browser, the request can be silently blocked or delayed. Often, this is combined with adaptive logic: known bad requests are challenged or blocked, while trusted users bypass CAPTCHAs entirely.

     Advantage: no user friction. Disadvantage: potential *false positives* (unusual but legitimate users) and the need for constant updates. Such methods are often used as part of layered systems like reCAPTCHA v3.

* **Proof-of-work challenges:** Beyond FriendlyCaptcha, proof-of-work has been proposed as a general access control method. Some websites require a small computational task (hashing) per request, implemented in JavaScript.

     Consumer websites rarely use this yet; it’s more common for API rate limiting. The aim is to make mass requests costly.
     
     The downside: slower devices suffer, and attackers with high computing power can bypass it.

* **Honeypot fields and time-based tests:** A simple alternative is a *honeypot* field — an invisible form field that humans won’t fill in but bots will. The server can then detect and block them.

     Similarly, timing can be measured: if a form is submitted within one second of loading, it was likely automated.

     These tests are simple and frictionless but only stop basic bots. Advanced attackers can easily avoid them.

## RESTCaptcha in context

RESTCaptcha combines IP reputation checks, proof-of-work challenges, timing analysis, and local device fingerprinting.

To be clear: RESTCaptcha is not a revolutionary change in CAPTCHA architecture. Rather, our goal was to implement the following principles:

* An **open-source solution** that is fully configurable and can be self-hosted by anyone (digital sovereignty).
* A **privacy-friendly, GDPR-compliant** implementation — no collection of personal data.
* **Simple integration** into existing websites, with minimal coding effort.
* A **pleasant and highly accessible user experience** — no image puzzles, distorted text, or stressful maths questions to get wrong.
