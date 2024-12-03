# The Principled Dev

## C R E D O

What makes Peciel awesome? It's the tenets we'll never compromise on.

### Preamble
As the saying goes, "familiarity breeds contempt." Most companies are rotten to the core, especially in the city known as a "place solely for commuting, as opposed to working," where I was (unfortunately) born.

The jobs available in the city I was born in are mostly supportive or secondary roles. Yet, people often get into trouble with typical and simple tasks. The cause is not that the department is underfunded; it is always the intelligence of the PEOPLE.

Alas, it was a simpler world decades ago, and those individuals undeservedly attained their positions solely by seniority rather than merit. These nouveau riche profited from the aftermath of the Cold War. We refer to them as "die-gwen-yow" (大滾友), literally "big-roll guy" (as in "this is how we roll"), or "chong-low" (廠佬), literally "factory guy" in Cantonese, both of which have no particularly good translations.

We must have strong conviction. Freedom must be forced: Impose strict management principles so that you can increase the autonomy of colleagues. Of course, the enforcers should adhere to these principles fully as well.

### First things first
1. Set Up Automatic Backup for Everything
    * A service (Windows) or daemon (Unix-like) is the best way to do this. It doesn't matter if the service was originally a daemon; you can always easily wrap an executable using something like `forever`.
2. Set Up Version Control (I know this sounds ridiculous, but I've recently encountered some firms that don't use it in 2024—goodness me!)
    * Use version control for EVERYTHING, including **DATABASE CHANGES, CONFIGURATION CHANGES, AND NETWORK CHANGES**.
    * At a minimum, create one working branch for each working unit, one for each version, and one master branch.
    * EDUCATE colleagues on how branches work, what the commands do, what `.gitignore` does, the file size limit (if there is any), and the remedies when a colleague uses it incorrectly.
    * Use [monorepos](https://en.wikipedia.org/wiki/Monorepo) to ease code reuse
3. Set Up an Issue Tracking System
    * Never use a spreadsheet for this. (Spreadsheets may have privacy and access control issues; the file will become extremely large; there is a lack of data control; and there are limited analytical tools available...)

### Comments and documentation
4. Line comment style: At least four spaces in front, format: `    // 20xx.xx.xx - Your name - What has been changed`
5. **API testing**: NEVER use ~~curl~~ or ~~Postman~~ (obsolete tools), for [many obvious reasons, security and "Don't Repeat Yourself" just being two of them](https://peciel.com/blog/2024/10/01/why-you-should-never-ever-use-postman). ALWAYS use an **OpenAPI generator** from function comments. For the interface, [Redoc](https://github.com/Redocly/redoc) is good for operations personnel, while [Scalar](https://github.com/scalar/scalar) is good for developers (it provides code snippets for making the sample request in multiple libraries in various languages).

### Error handling
6. **[RFC 9457](https://datatracker.ietf.org/doc/rfc9457/)**: Use the standard, troubleshoot-able format for all HTTP responses. The more verbose the better, but without the compromise of security:
```python3
from flask import make_response

res = make_response({
  "type": "https://example.com/errors/out-of-credit",
  "title": "You do not have enough credit.",
  "status": 403,
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345",
  "traceId": "acbd0001-7b8e-4e81-9d6b-3e34ce7c9a9e", # optional field, but highly recommended!! Internal Error ID goes here!!
  "balance": 30, # additional information
  "cost": 50 # additional information
})    # here you could use make_response(render_template(...)) too

res.headers['Access-Control-Allow-Origin'] = '*'
res.headers['Content-Type'] = 'application/problem+json'
res.headers['Content-Language'] = 'en'
return res
```
7. Public-facing error messages: Always include telephone number, email, (and office hours) of the technical support team. Ideally the customer can click on a button to email the internal error (`traceId` field in **RFC 9457 HTTP response**) to technical support.

8. Prevent duplicate requests (very common: NOT only caused by double clicks, but also refreshes, return to previous pages, and browser behaviour etc.): It should be handled both on the front end and back end. While you should always disable a button immediately after clicking it, the server should handle duplicate requests as well.

### Time format
9. **[RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6)**: Use `YYYY-MM-DD`.

### Standard routines
10. **Push notifications (cf. `push_notifications.vsdx`):** Explain what \*push\* notifications mean in the first place, as opposed to \*pull\* notifications. I have found that most people are confused by this distinction. Push notifications are ephemeral and should not be stored on servers, except for those that need to be resent due to an error. Instead, you should synchronize new notifications with old ones in local mobile storage. ALWAYS use a wrapper to encapsulate methods so that you can test them (both manually and automatically) before CI/CD.
11. **[RFC 7009](https://datatracker.ietf.org/doc/rfc7009/)**: OAuth 2.0 Token Revocation.
12. **[RFC 7519](https://datatracker.ietf.org/doc/rfc7519/)**: use JWT for everything confidential, like a log in session. These information must be encrypted beforehand. DON'T store them in `window.localStorage`, store it in Cookies with proper timeout.
13. Build your own build tools, such that it is adapted to the use case.
14. Eat your own dog food.
15. For mobile applications, use a single code base (e.g., Flutter, React Native) when possible (Don't Repeat Yourself; also saves the effort of writing and maintaining code)
16. **Atomic microservices**: Break tasks to microservices that do one thing only. Most of them should be reusable. Use an orchestration suite (there are lightweight ones like [Airflow](https://airflow.apache.org/) for arrangement and error handling.
17. **ISO 27001**: Use containerisation to isolate working sensitive data.
18. **ISO 27001**: Remove data that are no longer been used. (We ensure this using metadata and an internal system.)

### Dependencies

19. Use battle-tested libraries, even if they are only slightly more efficient or consume less storage. Don’t rely on random projects by some Russian developer on GitHub that have only a dozen stars (in my case, even the last commit was made 12 years ago) for the supposed benefit of a millisecond performance increase. (Alas, this is a true story.) Always fork the library and keep it updated yourself.

### Testing and deployment

20. Spend most of your time writing unit tests, that can be used as a yardstick for a new release, especially as automatic tests before continuous integration/deployment (CI/CD).

21. **Mobile-first approach** for Web page/application: When previewing webpage changes, always check the **mobile** site first.

22. EXPLAIN that the new Microsoft Edge browser is essentially Chromium, the base of Google Chrome. There is little point conducting unit tests with the browser.

## Yet... Technicalities are NOT the Most Important!
Just be proper:
1. ALWAYS introduce yourself, shares your GitHub account and ask the colleauge to follow you, follows him/her back.
2. ALWAYS talk slowly (and softly). NEVER talk fast. Fast talkers, especially at the workplace, not only lead to misunderstanding, but also gives an impression of being an idiot. The Irish poet talked very slowly on the Japanese show has left a deep impression on me. A famous commentator and English pedagogue from the city I was born in also noted that the missionaries who taught him English talked slowly, such that he understood most of what he was speeking, which gave him much confidence.
3. ALWAYS smile and give eye contact.
4. ALWAYS remember the names of your colleagues.
5. Keep the workplace clean and dust-free. This leaves a good impression.
6. NEVER speak a word of profanity, including a minced oath.

<!-- Red flags -->
<!-- Secure payment -->
