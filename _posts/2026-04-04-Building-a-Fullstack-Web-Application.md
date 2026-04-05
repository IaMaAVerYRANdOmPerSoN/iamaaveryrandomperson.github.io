---
layout: post
title: "Building a Fullstack Web Application"
categories: "Projects"
tags: "Fullstack, Web Development"
author: "Apostla"
---

Hey there! I'll be sharing my experience building a fullstack web application today! Long overdue, but I'm extremely busy at the moment. This project was a huge learning experience for me, and I'm excited to share my work and insights with you all.

Deployed: [ServiceMap](https://servicemap.info)  
(Repo will be public once I get some actual users and can justify the security risks of exposing the codebase, thanks for understanding!)

## Overview/About/Abstract if you're pretentious

Excerpt from the project README.md, which is a bit more formal and detailed:
> ServiceMap connects volunteers with organizations offering service opportunities. It provides a platform for users to discover and sign up for volunteer activities in their community, making it easier to give back and make a positive impact. Whether you're looking to volunteer within the school, at a local charity, or want to show your commitment to service, ServiceMap has you covered. With a user-friendly interface and a wide range of opportunities, ServiceMap is the go-to resource for anyone looking to make a difference through volunteering.
>
> **Key Features**
>
> 1. Interactive map application with a user-friendly interface for discovering and signing up for volunteer opportunities, with full GFM (GitHub Flavored Markdown) support for opportunity descriptions and details.
> 2. User authentication and authorization system with support for multiple user roles, including volunteers, organization representatives, and moderators.
> 3. Admin dashboard for managing volunteer opportunities, user accounts, and other aspects of the platform.
> 4. Integration with third-party APIs for geocoding and mapping, to provide accurate location data for volunteer opportunities.
> 5. Responsive design for optimal viewing on desktop and mobile devices.
> 6. Robust backend architecture built with Django, providing a scalable and maintainable codebase for future development and feature additions.

## Stack Choices

I was rather hasty when choosing my stack, and I ended up with a rather... unconventional one. I wanted to use Django + React, but opted for a vanilla JS/HTML/CSS frontend instead, which was more challenging but taught me key fundamentals I may have missed with the React abstraction. I tend away from strict plans, mostly building as I go, which provided various benefits and drawbacks. I had a general idea of the features I wanted, but I didn't have a clear roadmap, which led to some inefficient development and a few dead ends. However, it also allowed for more creativity and flexibility in the design process.

My stack choices were mostly based on what challenges I was encountering at the time, and what I wanted to learn. I didn't have a clear stack when I started, but as I encountered challenges, I made decisions based on what would best solve those issues and provide learning opportunities. If there weren't any particular challenges, I mostly defaulted to the first option I came across in tutorials and documentation, which was a bit haphazardous but ultimately didn't create any significant issues.

### Core components (Web frameworks, servers, databases, etc.)

- **Django**: I personally find Django as the most robust and scalable Python web framework, although it has a steep learning curve. My "Build as I Go" approach allowed me to pick up Django fundamentals organically, which was a rewarding experience. I used Django's built-in ORM for database management, which simplified data handling and migrations.

- **Vanilla JavaScript**: I opted for a vanilla JS frontend, which in hindsight, was an "interesting" choice. The lack of an opinionated framework combined with my loose planning resulted in a somewhat disorganized codebase. At the start, when the project was small, this wasn't a large issue, but without clear MVC separation and clean inheritance, the codebase quickly became a tangled mess. I eventually ended up with a 1200 line JavaScript file that handled everything from DOM manipulation to API calls, which was difficult to maintain and debug. Now that I understand core frontend concept better, I would likely benefit from using a framework like React or Vue to provide structure and organization to the frontend code.

- **Gunicorn**: I used Gunicorn as my WSGI server to serve the Django application. I didn't have a particular reason for choosing Gunicorn over other WSGI servers; it was simply the first choice I stumbled upon in Django deployment guides. It's extremely easy to configure and works well for my use case. I configured Gunicorn with 5 workers in production, following the 2 * cores + 1 rule. If I were to scale up in the future, I might consider most robust ASGI servers like Daphne or Uvicorn.

- **Nginx**: Once again, I chose Nginx simply because it was the first option I came across in deployment guides, however it appears as a much more informed choice. Nginx is a powerful, high-performance web server, serving my static files at lightning speeds. The reverse proxy configuration was straightforward, I only had to pass the headers and forward to Gunicorn through Docker's internal network. The single-file configuration also made it much easier to containerize and manage compared to older web servers like Apache, which require multiple configuration files and modules.

- **Databases**: I initially opted for SQLite for its simplicity and ease of setup, but ran into a few key issues:
  - **Containerization**: I wanted to containerize the application using Docker, but SQLite's file-based nature made it difficult to manage data persistence across containers. It was difficult to ensure each container had access to the same database file. Even if I had managed to synchronize the database file across containers, each individual Gunicorn worker with their own isolated Python environment would have had to manage their own connections to the database, which would have been a nightmare to set up and maintain.

  - **Horizontal Scaling**: If synchronizing the database file across containers would have been difficult, imagine synchronizing it across multiple different VMs/pods in a production environment. It would have been nearly impossible to ensure data consistency and integrity across multiple instances of the application. Even though my application is hosted on a single VM instance, I initially wanted to deploy to a Kubernetes cluster, which would have made scaling the application horizontally a nightmare with SQLite.

  - **Concurrency**: While my application does not require superb performance, it would only take one collision across 5*n Gunicorn workers to cause a database lock, which would have been a nightmare to debug and resolve in a production environment, whereas with production-ready databases like PostgreSQL, the database engine forks a new process for each connection, which offers protection against database locks and collisions, along with better performance and scalability.

  Eventually, due to these issues, I switched to PostgreSQL. I did encounter some challenges configuring my database (See branch `postgresMigration`), as I initially wanted to self-host a PostgreSQL instance, but ran into similar issues to SQLite with containerization and data persistence. I eventually shifted to Cloud SQL, which provided a robust and scalable database solution without the headaches of managing my own database server. I did run into a couple hiccups connecting my application to the Cloud SQL instance, going from cloud-sql-python-connector to running Cloud SQL Auth Proxy in a sidecar container, but these were minor issues and well worth the effort (I would still be struggling with proxy configuration today!).

- **Docker**: This one should be obvious enough. The vast majority of modern applications all run in orchestrated containers, and while orchestration engines where beyond the scope of this project, containerization itself provided a ton of benefits, such as consistent development and production environments, easier dependency management, and simplified deployment. I used Docker Compose for both development and production, which allowed me to manage multiple services (Django app, PostgreSQL database, Nginx server) in a single configuration file. Docker was pretty intuitive for me, and I was able to containerize my application without too much trouble, although I did run into some issues with database persistence and proxy configuration, as mentioned earlier. Overall, Docker was a great choice for this project and provided a solid foundation for future development and scaling.

  > Do: Use Docker for *cloud native* applications that do not require hardware access or "bare metal" features, and that can be easily containerized without complex configuration or performance issues.
  >
  > Don't: Use Docker for applications that require direct access to hardware resources, such as GPU-accelerated applications, or that have complex configuration requirements that may not be easily managed within a containerized environment. I made this mistake with my robotics projects, which require direct access to hardware resources and low-level system features that are not easily abstracted within a container. The hassle of mounting volumes, managing permissions, and giving containers access to the host's hardware resources end up becoming an "environment" of their own, which kind of defeats the purpose of containerization in the first place. For web applications and other cloud-native applications, Docker is a great choice, but for applications that require direct hardware access or complex configuration, it may be better to run them directly on the host system or use a different virtualization solution.

- **Cloudflared**: I initially hosted the testing server on my own Raspberry Pi, which posed serious security risks for me and my family. I used the cloudflared daemon (`sudo apt update && sudo apt install cloudflared`) to create a secure Cloudflare Argo Tunnel (`3cb96c7e-c1f8-4999-8de2-5b1f590f3e08.cfargotunnel.com`) to expose my local server to the internet without opening any ports on my router. I also got the added benefit of Cloudflare's full feature set of security and performance optimizations, such as the WAF, DDoS protection, and caching, all for free! At this point I did not have a registered domain, so I was randomly assigned a .trycloudflare.com subdomain. I continued to use the tunnel since it was already configured optimally in prod, although I could have easily enabled HTTP/HTTPS traffic on my VM instance + a simple CNAME record to point to my domain. (Which is better long term, but I was lazy, and the tunnel was working fine, so I just left it as is).

### Libraries, APIs, and other extensions

I made a rather controversial decision not using Django REST Framework. I had experimented with DRF in the past, and found it to be mostly a collection of abstractions and complex features that didn't really provide any value for anything short of enterprise-level applications with 50+ endpoints and complex relationships. For my simple application with <15 (I think?) endpoints, DRF would have been way overkill and added unnecessary complexity to the project. I built my API endpoints using plain Django views. Onto the real stuff!

#### Libraries and Extensions

- **Django-Allauth**: While my understanding of allauth and authentication in general was somewhat hazy, I knew that implementing a secure and robust auth pipeline from scratch would make me ~~Commit Suicide~~ pull my hair out, so I opted to use django-allauth to handle account creation, login, logout, third-party authentication, etc. I only dealt with minor challenges configuring allauth, mostly related to groups and permissions.

    Signals were a bit tricky to wrap my head around at first and were the biggest source of bugs in the auth pipeline, but was pretty set-and-forget once I got them working. Configuration was a breeze and I watched a few tutorials before I began implementation, which made the process much smoother. I also customized the allauth templates to match the design of my application, which was slightly annoying given the convoluted inheritance and template structure of allauth, but it was certainly better than building an auth system from scratch.

- **Django-Anymail**: I used the console email backend initially, which was fine for testing but obviously the manual copy-and-paste wasn't going to cut it for production. I followed the integration tutorial for my email provider, Mailtrap (What an awesome sandbox! Wow!), which led me to Django-Anymail, a powerful, easy to configure email backend that supports a wide range of email providers. Anymail was a breeze to set up since it hooks into the existing Django email system.

    I was able to avoid the complexities of the mail ecosystem, skipping SMTP and port configuration, since Anymail hooks into Mailtrap's API, thereby bypassing the SMTP relay system entirely. I also took advantage of Anymail's powerful templating system to create custom email templates for account activation and password reset emails, which added a nice touch of polish to the user experience.

- **Django-Ratelimit**: Because my application uses the Mapbox API, which has significant costs associated with persistent geocoding and inverse geocoding requests (as well as navigation, for the future), I wanted to implement some form of rate limiting, so ~~My dad doesn't kill me~~ I don't go broke. I used Django-Ratelimit to address these challenges, which provides a simple, yet flexible decorator-based approach to rate limiting.

    I implemented rate limiting on the geocoding and inverse geocoding views, as well as the opportunity creation view, which is the most resource-intensive endpoint in my application (in terms of human time, not compute time). I did not apply any rate limiting to the account views, since Django-Allauth already protects those endpoints against DoS-style attacks. The implementation was extremely straightforward, I simply added the following decorator to the relevant (class-based)views:

    ```python
    @method_decorator(ratelimit(key='ip', rate='...', block=True), name='dispatch')
    ```

- **Django-Blacklist**: When I was implementing blacklisting functionality for human administrators, I needed to prevent blacklisted users from authenticating and accessing the application entirely. I quickly discovered (to my surprise) that Django did not have any built-in blacklisting middleware, which was a bit concerning given how common of a feature this is for web applications.

    I found the Django-Blacklist micro-library, which provided a simple middleware-based approach to blacklisting users based on their IP address. Django-Blacklist was challenging to implement at first, due to the lack of effective documentation and examples, but as a micro-library, it was fairly easy to understand and customize once I got the hang of it. I built my blacklisting views and frontend components into the rest of the application, which allowed administrators to easily blacklist users.

- **Jsonschema**: I used the jsonschema library to build a custom model field that allows passing a schema as a file path or dictionary, which validates the JSON data against the schema before saving it to the database. This was (obviously) used for validating the structure of the opportunity recurrence timestamps, which are stored as JSON directly in the database.

    I intend to package this into a micro-library or add it to django-jsonschema (currently only supports form validation) in the future, hence the type annotations and robust error handling. The implementation wasn't too difficult, but it did deepen my understanding of Django's model field system and how to integrate custom validation logic into the model layer, which was a rewarding learning experience.

- **Loguru**: Big oopsie. As much as I love loguru for its simplicity and powerful features, it would have been a nightmare to integrate into a Django application, which uses the Logging module from the standard library. Setup and configuration was easy, and I was able to get loguru running smoothly, but now I have two separate logging systems running in the same application, which I know is going to cost me hours on a stupidly simple bug down the line. I would have been much better off using the built-in logging module, wouldn't require any additional configuration to integrate with Django's logging system, and would have provided a more consistent logging experience across the entire application. Oh well, live and learn I guess!

- **Markdown-It-Py**, and **Linkify-It-Py**: I wanted the opportunity descriptions to support Markdown formatting, so I used the markdown-it-py library to parse and render markdown content. The implementation was straightforward, I created a renderer.py file in my main app, which renders markdown with a LRU cache and sanitizes the output with ammonia to prevent XSS attacks. The function is integrated into all views that receive markdown content (HTML is stored directly in the database), which allows for consistent rendering of Markdown across the entire application.

    I borrowed GitHub's markdown CSS to stylize the rendered markdown content, which was much easier than building my own markdown styles from scratch, and provides a familiar and visually appealing design for users who are accustomed to GitHub's markdown rendering.

- **Nh3 (Ammonia)**: I used ammonia to sanitize user-generated content, particularly the rendered markdown content, to prevent XSS attacks and ensure the security of the application. I simply called `nh3.clean()` on any user-generated content before storing it in the database, which mitigated most XSS vectors.

- **FullCalendar.js**, **Mapbox GL JS** and **Mapbox Search JS**: I used FullCalendar.js to render a dynamic, interactive calendar on the frontend, which displays the opportunities and their recurrence patterns. The implementation was quite difficult since I made my own custom data format for storing the opportunity recurrence patterns, which required a custom parser to convert the data into a format that FullCalendar could understand.

    Mapbox GL JS was used to render interactive maps for the opportunity locations, which provided a much more visually appealing and immersive map experience compared to free map tiles. I also used Mapbox Search JS, and Mapbox's geocoding and inverse geocoding APIs to convert user-inputted addresses into geographic coordinates for map rendering and storage in the database, and to convert geographic coordinates back into human-readable addresses for display on the frontend.

#### API integrations

- **Mapbox API**: While I started out with the open-source leaflet library for map rendering, I was displeased with the aesthetic of free map tiles, and wanted to use Mapbox's 3D vector tiles and Mapbox GL JS library (with shaders, atmosphere, curvature, and all that awesome jazz!) to create a more visually appealing and immersive map experience. Mapbox provides additional APIs to build a complete geospatial experience, such as the geocoding API, which I used to convert user-inputted addresses into geographic coordinates for map rendering and storage in the database, and the inverse geocoding API, which... well does the inverse, converting geographic coordinates back into human-readable addresses. I also used Mapbox Search JS for interactive search functionality, which was much easier to implement than building my own search system from scratch. I'm very pleased with Mapbox's free tier, intending to use more of their APIs and SDKs in the future, such as the navigation SDK for turn-by-turn directions and the isochrone API for calculating travel times and distances.

- **Mailtrap API**: I did some background research on email providers, eventually settling on Mailtrap for its easy integration, my main priority at the time. I believe I made the correct decision, since Mailtrap's API integrated seamlessly with Django-Anymail, which simplified the process of sending emails from my application. While they lack in analytics and reporting features compared to more mature providers like SendGrid and Postmark, those features are not a priority for a high school project, and Mailtrap's focus on providing a robust sandbox environment for testing email functionality was perfect for my needs. Their documentation and integration guides were also top-notch, which made the implementation process much smoother and less frustrating than I anticipated. The free tier offers a generous 5000 emails per month, although some features that I would like to use are locked behind the paid plans.

I think my choices were mostly informed, albeit a bit haphazardous at times. Without a clear plan for the stack at the start, but as I encountered challenges, I made decisions based on what would best solve those issues and provide learning opportunities. I certainly believe I would benefit from more careful planning and consideration of the stack choices, but overall I'm pleased with the technologies I ended up using and the learning experience they provided.

## Proseline (Thanks, Wikipedia!)

I began with an EV charging station [tutorial](https://www.youtube.com/watch?v=E2bhoCOMlsA) from BugBytes with Django and Leaflet, which provided a solid foundation for building a map-based application with Django. I adapted the tutorial's code to fit my needs, which involved significant changes to the models and views. I added APIs for creating new service opportunities and frontend components for rendering those opportunities on the map. I completely redid the frontend, starting the 1200 line JavaScript file. I removed the CSV data and continued migrating the codebase away from the tutorial's structure, while implementing the auth pipeline with django-allauth, and extending core functionality with administrator tools and better UI/UX.

I started migrating to Mapbox GL JS early on, which required major refactors to the frontend code, since Mapbox GL JS uses a different API than Leaflet. I also had to configure my API keys, set up my Mapbox account, and create custom map styles. I extended my ServiceOpportunity model to include fields for addresses and markdown descriptions. I also added a live markdown preview to the opportunity creation form, using markdown-it-js on the frontend and markdown-it-py on the backend to render the markdown content. I patched a bug in the permission assignment path, and updated the license from GPL to AGPL, which posed a challenge since I had to verify all my dependencies were compatible with the AGPL license, and update the license headers in some of my source files.

Then the real "this is becoming a real product" phase kicked in.

Around mid-February, I added proper base templates, navigation, and organization approval flows. Shortly after, I started cleaning up observability and geocoding behavior, while also doing the endless frontend "one more tiny polish" loop that somehow takes 4 hours every time.

By March, a lot of work shifted into account and notification flows:

- email verification templates
- provider/social login UI
- support replies and opportunity notification emails
- account management email polish

I also did a full rebrand from **HelpMap** to **ServiceMap** (twice, because git is fun like that), plus backup scripts, Docker permission fixes, and some auth hardening work.

Late March was mostly infra ~~development~~ panic. I moved database integration toward Cloud SQL, added the Cloud SQL proxy path, and refactored Docker/environment configuration, so deployments were less "cross fingers and pray" and more reproducible. There was also a merge from the `postgresMigration` branch into `runningDev`, which was basically the point where this stopped feeling like a prototype and started feeling like an actual application stack.

After that came quality-of-life and cleanup commits:

- account template/style refactors
- one very honest `2am hotfix help me` commit
- docs cleanup in README/CONTRIBUTING
- CLI commands for quickly creating/deleting opportunities

So yeah, this project evolved in a pretty chaotic-but-productive arc:

1. tutorial scaffold
2. aggressive customization
3. feature explosion
4. infra panic
5. stabilization and tooling

Not the most elegant linear roadmap, but with my "rebase spam" approach to commits, at least I can make the repo *look* like it was a smooth process, even if I was screaming internally the entire time.

But yeah. From a simple map-based tutorial to a production-ready application with user authentication, markdown support, email notifications, and a robust backend architecture, I think this project has come a long way. I've built something real! An actually meaningful product!

## What I Learned

- "Build as you go" is great for learning, terrible for keeping files short.
- Authentication is easy to underestimate and hard to retrofit.
- Containerization decisions should be made early, especially for database strategy.
- A uh... "distributed" logging strategy is not optional once bugs have cascaded between services.
- Good docs are a force multiplier, especially when future-you is sleep-deprived.

## If I Rebuilt ServiceMap Tomorrow

- I would use a frontend framework from day one for better state/component structure.
- I would standardize logging on one system only.
- I would define clearer feature milestones before implementation sprints.
- I would design my deployment/data architecture before touching production settings.
- I would fix my sleep schedule and use better commit messages.

## Closing Thoughts

ServiceMap started as "Woah. You have zero service hours" and ended as "okay wait, this is actually a production application."

It is not perfect. It has rough edges. Some commits are questionable. But it works, people can use it, and I learned way more than I would have from a perfectly planned toy app. There's probably a few critical security issues waiting to give me an all-nighter, and there isn't a single test in the codebase, but it's live, it's useful, and it's mine!

If you're considering a project that feels a bit too ambitious, do it anyway. Just maybe keep your JavaScript files under 1200 lines unlike me. And write some tests! And please don't stay up fixing prod with zero users at stake, that's just embarrassing. But other than that, go wild! Build something real, learn a ton, and don't be afraid to make mistakes along the way. That's how you grow as a developer (and a person).

[Deployed here!](https://servicemap.info)

Peace, Apostla :3
