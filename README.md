# The Principled Dev

## Preamble
As the saying goes, **'familiarity breeds contempt'**. Most companies are **rotten to the core**, especially in the city (known as a 'place of solely for commuting, opposed to working') where I was (unfortunately) born in.

The jobs available in the city I was born in are mostly of supportive/secondary roles. Yet people always get into trouble with typical and simple tasks. The cause is not that the department is underfunded, it is always (the intelligence of) PEOPLE (Prof Davison, whom I know personally know, agrees with me).

Alas, it was a simpler world decades ago, and those people undeservedly got their position only by seniority and not by merit. These nouveau-riche only profited from the aftermath of the Cold War. We call them 'die-gwen-yow' (大滾友), literally 'big-roll [as in 'this is how we roll']-guy') or 'chong-low' (廠佬), literally 'factory-guy' in Cantonese, both having no particularly good translations.

Impose strict management principles so that you can increase autonomy of colleagues. The enforcer should follow these principles fully as well.

## C R E D O

What makes Peciel awesome? It's the tenets we'll never compromise on.

### First things first
1. Set up Automatic Backup for everything
    * Service(Windows)/daemon(Unix-like) is the best way to do it. It's not a big deal if the service is originally a daemon or not, as you can always easily wrap an executable using something like `forever`.
2. Set up Version Control (I know this sounds ridiculous, but I've just recently come across some firms that don't use in 2024, goodness me)
    * Use Version Control for EVERYTHING, including **DATABASE, CONFIGURATION AND NETWORK CHANGES**
    * At least create one working branch for one working unit, one for each version, and one master branch
    * EDUCATE how branches work, what the commands do, what `.gitignore` does, the file size limit (if there is any), and the remedies when a colleague used it incorrectly
3. Set up Issue Tracking system. Never use a spreadsheet for it. (spreadsheets may have privacy and access control issues; the file will become extremely large; lack of data control; lack of suitable analytical tools...)

### Comments and documentation
4. Comment style: At least four spaces in front, format: `    // 20xx.xx.xx - Name - What has been changed`
5. **API Testing**: NEVER use ~~curl~~ or ~~Postman~~ (obsolete tools), for [many obvious reasons, security and "Don't Repeat Yourself" just being two of them](https://peciel.com/blog/2024/10/01/why-you-should-never-ever-use-postman). ALWAYS use an OpenAPI generator from function comments.

6. Push notifications: Explain what \*push\* notifications mean in the first place, as opposed to \*pull\* notifications. I found most people confused by it. Push notifications are ephemeral and should not be stored in servers, except the ones that need to be resent as a result of an error. Instead you should synchronise new notifications with old ones in local mobile storage. ALWAYS use a wrapper to wrap methods so that you can test them (both manually and automatically before CI/CD).

### Error handling
7. **RFC 9457**: Use the standard, troubleshoot-able format for all HTTP responses. The more verbose the better, but without the compromise of security:
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
8. Public-facing error messages: If you follow Always include telephone number, email, (and office hours) of the technical support team. Ideally the customer can click on a button to email the internal error (`traceId` field in RFC 9457 HTTP response) to technical support.

9. Use battle-tested libraries, even when they are a tad more efficient, consumes less storage, etc. Don't use some random project by some Russian developer on GitHub with a dozen of stars, for the sake of a millisecond performance increase. (Alas, this is a true story.) Always fork the library and keep it up to date.

10. Spend most of your time writing unit tests, that can be used as a yardstick for a new release, especially as automatic tests before continuous integration/deployment (CI/CD).

11. **Mobile-first approach** for Web page/application: When previewing webpage changes, always check the **mobile** site first.

### Testing and Deployment

12. EXPLAIN that the new Microsoft Edge browser is essentially Chromium, the base of Google Chrome. There is little point conducting unit tests with the browser.

13. **RFC 7009**: OAuth 2.0 Token Revocation.
14. **RFC 7519**: use JWT for everything confidential, like a log in session. DON'T store it in `window.localStorage`, store it in a Cookie with proper timeout.
15. Build your own build tools, such that it is adapted to the use case.
16. Eat your own dog food.
17. For mobile applications, use a single code base (e.g., Flutter, React Native) when possible (Don't Repeat Yourself; also saves the effort of writing and maintaining code)
18. **Atomic microservices**: Break tasks to microservices that do one thing only. Most of them should be reusable. Use an orchestration suite (there are lightweight ones like Airflow) for arrangement and error handling.
19. **ISO 27001**: Use containerisation to isolate working sensitive data.
20. **ISO 27001**: Remove data that are no longer been used. (how to ensure it?)

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
