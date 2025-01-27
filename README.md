# The Principled Dev

## C R E D O

This is a set of tenets that we devs will never compromise on.

### Preamble
<!-- As the saying goes, **"familiarity breeds contempt."** Most companies are rotten to the core, especially in the city known as a "place solely for commuting, as opposed to working," where I was (unfortunately) born. -->

The jobs available in the city I was born in are mostly supportive or secondary roles. Yet, people often get into trouble with typical and simple tasks. The cause is not that the department is underfunded; it is always the ***DON'T-CARE ATTITUDE and LACK OF COMMON SENSE***.

Alas, it was a simpler world decades ago, and those individuals undeservedly attained their positions solely by seniority rather than merit. These nouveau riche people, profited from the city's advantageous conditions amidst the Cold War (gone now), are ignorant of ***universal best practices*** and show no interest in learning them. We refer to them as 'die-gwen-yow' (大滾友), literally 'big-roll guy' (as in 'this is how we roll', akin to 'stealing a living') in Cantonese; 'chong-low' (廠佬), literally 'factory guy'; or 'bong-yow' (磅友)[[1]](https://apps.itsc.cuhk.edu.hk/hanyu/Page/Search.aspx?id=17859), roughly 'someone who pretends to help'; all of which have no particularly good translations. No tech industry exists there but THREE(!) archetypal Potemkin 'science' villages.

Can you imagine, a company I worked for briefly failed at every single point outlined below? You see, when I tried to protest privately, it was in vain; when I protested publicly, I was seen as 'backstabbing'. When something finally failed (a severe data breach in this case), because of their 'loyalty' to the firm and ignorance of layman superiors (there are similarities with the former CEO of Boeing, who is not an engineer), those 'die-gwen-yow's did not need to be held accountable.

This sort of ***problematic governance*** will only lead to once hardworking colleagues adopting a 'lying flat' ('tong-ping', 躺平) mentality, becoming fatalistic and cynical. <!-- I realised my days were numbered as the city faced a brain drain. Ultimately, I decided to leave for good, despite having spent nearly three decades growing up and studying in that degenerate place. --> 

We must have strong conviction. Freedom must be forced: Impose strict management principles so that you can increase the autonomy of colleagues. Of course, the enforcers should adhere to these principles fully as well.

To good developers, remember to ***READ THE MANAGER BEFORE THE COMPANY***!

### First things first
0. ***Always do a quick web search before making an assertion.*** One 'die-gwen-yow' even insisted that 'amd64' was a mistake since the processor is an Intel. *\*sigh\** (It is a specification. Builds for the 64-bit platform, no matter the processor is Intel and AMD, are called 'amd64'. This is because the original specification was created by AMD and released in 2000.)

> If a workman wishes to do a good job, he must first sharpen his tools.
> 
> – ancient proverb
1. Set Up Automatic Backup for Everything
    * A service (Windows) or daemon (Unix-like) is the best way to do this. It doesn't matter if the service was originally a daemon; you can always easily wrap an executable using something like `forever`.
2. Set Up Version Control (I know this sounds ridiculous, but I've recently encountered some firms that don't use it in 2024—goodness me!) [[2]](https://www.reddit.com/r/csharp/comments/1ey6ggq/how_big_a_red_flag_is_it_for_a_company_not_to_use/) [[3]](https://www.reddit.com/r/csharp/comments/1ey6ggq/comment/ljblju9/)
    * Use version control for EVERYTHING, including **DATABASE CHANGES, CONFIGURATION CHANGES, AND NETWORK CHANGES**.
    * At a minimum, create one working branch for each working unit, one for each version, and one master branch.
    * Teach colleagues on how Git branches work, what the commands do, what `.gitignore` does, the file size limit (if there is any), and the remedies when a colleague uses it incorrectly.
    * Use [monorepos](https://en.wikipedia.org/wiki/Monorepo) to ease code reuse.
3. Set Up an Issue Tracking System
    * Never use a spreadsheet for this. (Spreadsheets may have privacy and access control issues; the file will become extremely large; there is a lack of data control; and there are limited analytical tools available...)

### Comments and documentation
4. Line comment style: At least four spaces in front, format: `    // 20xx.xx.xx - Your name - What has been changed`
5. **API testing**: NEVER use ~~curl~~ or ~~Postman~~ (obsolete tools), for [many obvious reasons, security and "Don't Repeat Yourself" just being two of them](https://peciel.com/blog/2024/10/01/why-you-should-never-ever-use-postman). ALWAYS use an **OpenAPI generator** from function comments. For the interface, [Redoc](https://github.com/Redocly/redoc) is good for operations personnel, while [Scalar](https://github.com/scalar/scalar) is good for developers (it provides code snippets for making the sample request in multiple libraries in various languages).

### Error handling
6. **[RFC 9457](https://datatracker.ietf.org/doc/rfc9457/)**: Use the standard, troubleshoot-able format for all HTTP responses. The more verbose the better, but without the compromise of security. Here is an actix_web (Rust) example:
```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, Error};
use serde::{Deserialize, Serialize};
use uuid::Uuid;

#[derive(Serialize, Deserialize)]
struct InputData {
    name: String,
    age: u32, // Age is a required field
}

#[derive(Serialize)]
struct ResponseData {
    message: String,
    received_data: InputData,
}

#[derive(Serialize)]
struct ProblemDetails {
    #[serde(rename = "type")]
    problem_type: String,
    title: String,
    status: u16,
    detail: String,
    instance: String,
    trace_id: Option<String>, // Optional field for tracing
    balance: Option<u32>,      // Additional information (if applicable)
    cost: Option<u32>,         // Additional information (if applicable)
}

// Custom error handler
async fn custom_error_handler(err: Error) -> HttpResponse {
    // Log the error or handle it according to your needs
    println!("Error occurred: {:?}", err);

    // Return a 400 Bad Request with a custom problem details response
    let problem_details = ProblemDetails {
        problem_type: "https://example.com/errors/missing-field".to_string(),
        title: "Missing required field.".to_string(),
        status: 400,
        detail: "One or more required fields are missing.".to_string(),
        instance: "/post_data".to_string(),
        trace_id: Some(Uuid::new_v4().to_string()), // Generate a new trace id for the error
        balance: None,
        cost: None,
    };

    HttpResponse::BadRequest()
        .header("Content-Type", "application/problem+json")
        .json(problem_details)
}

// Handler for the POST request
async fn post_data(item: web::Json<InputData>) -> impl Responder {
    let cost: u32 = 50; // Example cost
    let current_balance: u32 = 30; // Example current balance
    let trace_id = Uuid::new_v4().to_string(); // Generate a unique trace ID

    // Validate input data (age validation since it's required)
    if item.age < 0 || item.age > 120 {
        // Return error response if age is invalid
        let problem_details = ProblemDetails {
            problem_type: "https://example.com/errors/invalid-age".to_string(),
            title: "Invalid age provided.".to_string(),
            status: 400,
            detail: "Age must be between 0 and 120.".to_string(),
            instance: "/post_data".to_string(),
            trace_id: Some(trace_id),
            balance: None,
            cost: None,
        };
        return HttpResponse::BadRequest()
            .header("Content-Type", "application/problem+json")
            .header("Access-Control-Allow-Origin", "*")
            .header("Content-Language", "en")
            .json(problem_details);
    }

    // Check available balance against cost
    if current_balance < cost {
        // Return error response for insufficient credits
        let problem_details = ProblemDetails {
            problem_type: "https://example.com/errors/out-of-credit".to_string(),
            title: "You do not have enough credit.".to_string(),
            status: 403,
            detail: format!(
                "Your current balance is {}, but that costs {}.",
                current_balance, cost
            ),
            instance: "/account/12345".to_string(),
            trace_id: Some(trace_id),
            balance: Some(current_balance),
            cost: Some(cost),
        };
        return HttpResponse::Forbidden()
            .header("Content-Type", "application/problem+json")
            .header("Access-Control-Allow-Origin", "*")
            .header("Content-Language", "en")
            .json(problem_details);
    }

    // Create the success response data
    let response = ResponseData {
        message: "Data received successfully".to_string(),
        received_data: item.into_inner(),
    };

    // Return the success response with a 200 OK status
    HttpResponse::Ok()
        .header("Content-Type", "application/json")
        .json(response)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/post_data", web::post().to(post_data))
            // Add custom error handling here if needed
            .default_service(web::route().to(custom_error_handler)) 
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```
7. Public-facing error messages: `Debug` and `Release` builds are so much more than different optimisation levels and environment variables. The public-facing error message must be understandable by someone completely non-technical. Always include telephone number, email, (and office hours) of the technical support team. Ideally the customer can click on a button to email the internal error (`traceId` field in **RFC 9457 HTTP response**) to technical support.

8. Prevent duplicate requests (very common: NOT only caused by double clicks, but also refreshes, return to previous pages, and browser behaviour etc.): It should be handled both on the front end and back end. While you should always disable a button immediately after clicking it, the server should handle duplicate requests as well.

### Time format
9. **[RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6)**: Use `YYYY-MM-DD` to avoid confusion between month and day.

### Standard routines
10. **Push notifications (cf. `push_notifications.vsdx`):** Explain what \*push\* notifications mean in the first place, as opposed to \*pull\* notifications. I have found that most people are confused by this distinction. Push notifications are ephemeral and should not be stored on servers, except for those that need to be resent due to an error. Instead, you should synchronize new notifications with old ones in local mobile storage. ALWAYS use a wrapper to encapsulate methods so that you can test them (both manually and automatically) before CI/CD.
11. **[RFC 7009](https://datatracker.ietf.org/doc/rfc7009/)**: OAuth 2.0 Token Revocation.
12. **[RFC 7519](https://datatracker.ietf.org/doc/rfc7519/)**: use JWT for everything confidential, like a log in session. These information must be encrypted beforehand. DON'T store them in `window.localStorage`, store it in Cookies with proper timeout.
13. Never use gibberish like Lorem Ipsum as placeholders. Better use something like 'This section is under construction.'
14. Build your own build tools, such that it is adapted to the use case.
15. Eat your own dog food.
16. For mobile applications, use a single code base (e.g., Flutter, React Native) when possible (Don't Repeat Yourself; also saves the effort of writing and maintaining code)
17. **Atomic microservices**: Break tasks to microservices that do one thing only. Most of them should be reusable. Use an orchestration suite (there are lightweight ones like [Airflow](https://airflow.apache.org/) so it is unreasonable to complain about the extra work) for arrangement and error handling.
18. **ISO 27001**: Use containerisation to isolate working sensitive data, such as server-side workflows using user sessions.
19. **ISO 27001**: Remove data that are no longer been used. (We ensure this using metadata and an internal system.)

### Dependencies
20. Use battle-tested libraries, even if they are only slightly more efficient or consume less storage. Don’t rely on random projects by some Russian developer on GitHub that have only a dozen stars (in my case, even the last commit was made 12 years ago) for the supposed benefit of a millisecond performance increase. (Alas, this is a true story.) Always fork the library and keep it updated yourself.

### Testing and deployment
21. Spend most of your time writing unit tests, that can be used as a yardstick for a new release, especially as automatic tests before continuous integration/deployment (CI/CD).

22. **Mobile-first approach** for Web page/application: When previewing webpage changes, always check the **mobile** site first.

23. EXPLAIN that the new Microsoft Edge browser is essentially Chromium, the base of Google Chrome. There is little point repeating unit tests if you have done them on Chrome.

23. SQL Injection attacks: security should be offensive, not just defensive. There are many platform-agnostic tools available like [Sqlmap](https://en.wikipedia.org/wiki/Sqlmap).

<!-- Red flags -->
<!-- Secure payment -->

## Yet... Technicalities are NOT the Most Important!
It is effortless to avoid being rude, yet the city I was born in disappointed me... [[4]](https://www.reddit.com/r/HongKong/comments/1fo0b0r/comment/lomc4zk/)[[5]](https://geoexpat.com/forum/26/thread190305.html)[[6]](https://www.youtube.com/watch?v=CKr0soIf4jA)[[7]](https://www.quora.com/Are-Hong-Kong-people-rude)[[8]](https://www.quora.com/Why-are-people-so-rude-and-grumpy-in-Hong-Kong-like-they-hate-you-without-even-knowing-you)[[9]](https://www.quora.com/What-is-your-opinion-on-people-from-Hong-Kong-Why-do-you-feel-this-way)[[10]](https://www.quora.com/Are-the-people-of-Hong-Kong-rude-I-often-have-a-poor-experience-when-I-visit-Hong-Kong-Do-others-have-the-same-experience)[[11]](https://news.ycombinator.com/item?id=15772797) And yes, ***the rudeness is contagious*** since us human beings learn by imitating our peers.

Let's see what international netizens have to say about the city. ***I don't agree entirely***, but as the saying goes, 'many a true word is spoken in jest':
> "The sternest look (that would put Severus Snape to shame)."
>
> "It's just generally not a happy place. Don't take it personally."
>
> "That was my experience too. Everyone looks sooo unhappy – even by the standards of a big city. And I'm from London!"
>
> "The frustrations of the people are definitely a factor but people have been acting this way long before five years ago."
>
> "_ _ is a s\*\*t\*\*\*e place. Do not go there especially if you don't speak their language or if you have small kids. The locals are rude and try to cheat whenever they can. They have no integrity, morale, compassion; they do not know how to smile; there is no such thing as courtesy; they hate their jobs and their lives."
>
> "One topic was always brought up no matter who I spoke to,  _ _ are rude – downright rude."
>
> "I don't hate _ _ anymore, I just pity. Something needs to be done to save the poor people in _ _. And someone needs to do it".
>
> "The rudeness has nothing to do with the cost of living and business, as talent and hard work mean nothing in front of systematic class consolidation."
>
> "We can't even get along with ourselves."
> 
> "FILTH" (acronym for "Failed in London, try _ _")

There are very few expats left in that city. Alas, how can that wannabe business hub become a real one, when most people have adopted that attitude?

And how can people from that city be efficient leaders? The servant leadership movement asserts that leaders have a duty to focus ***primarily*** on their subordinates' well-being. When a colleague has low job satisfaction, how can she/he be productive?

Just be proper (also a 'note to self'):
1. ALWAYS introduce yourself, shares your GitHub account and ask the colleauge to follow you, follows him/her back.
2. ALWAYS greet your colleagues, at least say "morning'" and 'see ya'.
3. ALWAYS talk slowly (and softly). NEVER talk fast. Fast talkers, especially at the workplace, not only lead to misunderstanding, but also gives an impression of being an idiot. The Irish poet Peter MacMillan spoke slow and lucid English on a Japanese television show, which made a lasting impression on me. A famous commentator and English pedagogue from the city I was born in also observed that the American missionaries who taught him English spoke slowly, enabling him to understand most of what was said, which greatly boosted his confidence in speaking English.
4. ALWAYS smile and give eye contact.
5. ALWAYS remember the names of your colleagues and strive to spell their names correctly.
6. Personal hygiene: Keep the workplace clean and dust-free. This leaves a good impression. Don't cough in front of people.
7. NEVER speak a word of profanity, including a minced oath.
8. NEVER speak from the throat.
9. Appreciate your colleagues.

## 'Zo kwe zo'?
There are even some peculiar companies which go to great lengths to talk about the company ethos. One firm states that 'they believe everybody has something that they were good at', which essentially translates to "cheer up, you're not useless". I find this motto very patronising. It appears their interest lies in employing low-skill labour without much concern for the quality of work produced.

The condescending waffle reminds me of the phrase 'zo kwe zo,' meaning 'all people are people' in the Sango language, which is the motto of the Central African Republic. This country is often referred to as a failed state and has the lowest GDP per capita (PPP) as reported by both the World Bank and the CIA. This approach is the opposite of meritocracy, yet ironically, this company hails from Singapore, a nation that prides itself on its meritocratic principles.
