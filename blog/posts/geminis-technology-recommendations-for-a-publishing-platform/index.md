<figure><img loading="lazy" decoding="async" src="Gemini_Generated_Image_hnmg7ohnmg7ohnmg.jpeg" alt="AI-generated image of a robot writing a blog article"><figcaption>AI-generated image of a robot writing a blog article</figcaption></figure>

Going through my recent technical conversations with AI about various architecture and technology choices, I thought I would post this one first. It’s a shorter conversation, but is still typical of the discussions I’ve had. In this case, it’s about the technology and architecture choices that would work well for a publishing platform.

Given my interest in blogging and the fact that I currently have four more or less active blogs all running on WordPress, I’ve often wondered what the best technology stack would be to create a custom replacement for WordPress. I’ve thought about this and made several failed attempts at actually implementing one for years. WordPress is incredibly sticky though and works just well enough that I haven’t fully gone through the effort of creating a full replacement. That said, I still like to think about it and that’s what this conversation is mainly about.

However, in this conversation, my curiosity gets the better of me and I go off on a tangent about massive publications like BBC that process enormous amounts of requests and what it would choose for that situation as well. It’s an interesting insight.

I did restrict it, though, to the four main languages I’m familiar with: TypeScript/Node.js, PHP, .NET and Swift (using the Vapor web framework). Even if it were to actually choose something else if I gave it free rein (which I’ve done in other conversations), I originally began this one with the idea that I might actually implement it. Needless to say, that never happened.

You can see my thoughts on the conversation at the end of the post. Also, you can view the original conversation on Gemini [here](https://share.gemini.google/i3IBUqFVkZsq).

* * *

**Prompt:** *I’ve been thinking about what it would take to create a publishing platform for a website that needs the following features: posts with categories, subcategories and tags, users with several roles (reader, admin, contributor, editor, author), a CMS for editing and publishing contents, comments, media, etc., a frontend that is publicly accessible, a frontend cache, the ability for registered users (reader role) to leave comments, RSS feeds, an API for apps to access published content, ability to create, send and manage newsletters, support for multiple languages, the ability to automatically post to social media when new content goes live, support for multiple podcasts, support for video content, and the ability to add several other features. I would like to use TypeScript, PHP, .NET or Swift+Vapor as a technology stack. What would you recommend? Also, what would you recommend in terms of other parts of the tech stack, for example databases?*

Building a platform with this level of complexity—encompassing strict RBAC, multi-tenant-like media handling, multi-language support, and heavy API/caching requirements—demands an architecture that prioritizes long-term maintainability and performance.

Given the requirement for a robust, long-term solution, here is a breakdown of how the candidate technologies stack up for this specific feature set, along with recommendations for the wider architecture.

### The Backend Application Layer

**1\. TypeScript (Node.js with Fastify or NestJS)**

-   **The Verdict:** The strongest pragmatic choice for this specific feature set.
-   **Why:** A platform doing social media auto-posting, RSS generation, newsletter dispatching, and media handling benefits massively from the Node ecosystem. You won’t have to write integrations for AWS SES (newsletters) or Twitter/LinkedIn APIs from scratch.
-   **Architecture Fit:** Using Fastify gives you excellent throughput for the API and frontend rendering. Having TypeScript on the backend also allows you to share types and validation logic directly with your frontend, speeding up iteration.

**2\. Swift + Vapor**

-   **The Verdict:** The best choice for raw performance, memory safety, and a low-dependency footprint.
-   **Why:** If you want a highly performant, tightly controlled architecture with minimal overhead, Vapor is outstanding. It is particularly strong for building custom, lightning-fast APIs and handling heavy concurrent read requests.
-   **The Trade-off:** The Swift server-side ecosystem is smaller. You will likely find yourself writing custom wrappers for third-party APIs (like social media auto-posting or newsletter service integrations) that would be off-the-shelf packages in TypeScript or PHP.

**3\. PHP**

-   **The Verdict:** Highly capable, but depends on your approach.
-   **Why:** If you utilize a heavy framework like Laravel, this entire feature list is practically its blueprint. However, if you prefer building custom architectures or using vanilla PHP for long-term stability, managing the sheer volume of features (especially asynchronous tasks like bulk newsletter sending and video processing) can become complex compared to event-driven environments.

**4\. .NET**

-   **The Verdict:** An enterprise powerhouse, but potentially heavier than necessary.
-   **Why:** It has phenomenal out-of-the-box support for complex user roles, authentication, and localization. However, it can feel bloated if your goal is a streamlined, low-dependency application.

**Recommendation:** If you want to move quickly and lean on existing integrations for the auxiliary features (social, newsletters), go with **TypeScript**. If you prioritize a lean, compiled, and highly performant core and don’t mind writing custom integrations, go with **Swift + Vapor**.

### The Wider Tech Stack Recommendations

To support the heavy media, caching, and background processing requirements, you need a decoupled architecture.

**1\. Primary Database: PostgreSQL** PostgreSQL is the undeniable choice here.

-   **Hierarchical Data:** It natively handles the recursive queries needed for categories and subcategories.
-   **Relational Integrity:** Perfect for managing complex many-to-many relationships (users to roles, posts to tags).
-   **Multi-language:** Postgres’s `JSONB` column type is incredibly efficient for storing localized string variations (e.g., storing a post’s title and metadata in multiple languages) without needing overly complex table joins.

**2\. Caching Layer: Redis & SQLite**

-   **Redis:** Essential for managing rate limiting on the API, session storage, and queue management for asynchronous tasks (like sending a 10,000-subscriber newsletter).
-   **Read Cache:** For the publicly accessible frontend, you need a lightning-fast read cache. Pushing compiled, published markdown/HTML into a highly optimized, file-based SQLite database acting purely as a read-cache for the frontend is a brilliant, low-latency pattern that drastically reduces the load on your primary Postgres database.

**3\. Object Storage: S3-Compatible Storage (AWS S3, Cloudflare R2, MinIO)**

-   Do not store podcasts, videos, or high-res images on the local server block. Offload all media to an object storage bucket placed behind a CDN. Cloudflare R2 is highly recommended here due to its zero-egress fee structure, which is vital when serving bandwidth-heavy podcast and video files.

**4\. Background Workers / Queues**

-   Features like “automatically posting to social media,” “sending newsletters,” and “processing video content” cannot run on the main thread. You will need a dedicated background worker process. In TypeScript, this would be BullMQ; in Vapor, you would utilize SwiftNIO’s event loops or a dedicated queuing package to handle these jobs asynchronously.

Given the heavy media requirements for podcasts and video, are you planning to encode and transcode those media files yourself on the server, or will you offload that processing to a specialized third-party service?

* * *

**Prompt:** *This is just a thought experiment. I feel like I really want to create something like this, but can’t justify doing it for myself since I have no real use for it.*

That “itch” is incredibly real. There is a specific kind of developer joy in designing a sprawling, beautifully engineered machine just to watch the gears turn perfectly. It’s the digital equivalent of building a highly detailed model railway in the basement—you don’t build it because you need a tiny train to commute to work; you build it for the sheer mastery of the craft.

When you don’t have a commercial “use case” or a client breathing down your neck, the justification changes. It stops being about “product-market fit” and becomes about **technical play**.

If you look at it through that lens, building a platform like this is actually the ultimate sandbox. It lets you:

-   **Test extreme architectures:** You can try out patterns that a risk-averse company would never let you deploy—like using a compiled, zero-dependency Swift/Vapor binary as a lightning-fast API, or using a flat-file markdown system with an SQLite database acting as an ultra-high-speed, read-only cache.
-   **Build a reusable blueprint:** If you design it with strict modularity, you aren’t just building a single website. You’re building a personal, battle-tested framework. The next time you (or a friend, or a client) need a fast blog, a portfolio, a newsletter manager, or a podcast feed, you can spin up a lightweight, highly optimized instance of your own engine in minutes rather than reaching for a bloated, dependency-heavy third-party tool.

Sometimes, the best justification for building something is simply that you want to see if you can make it run faster, cleaner, and more elegantly than anything else currently out there.

If you did let yourself build even just a small, highly optimized slice of this system purely for the intellectual fun of it, which specific architectural piece would you want to tackle first?

* * *

**Prompt:** *Back to the original question. How would your recommendation change if the project were for a major news organization like BBC?*

For an organization at the scale of the BBC, the architectural requirements shift dramatically. You are no longer just optimizing for CPU cycles or memory footprints on a single server; you are optimizing for **global high availability, extreme read-to-write ratios, talent pool hiring, and disaster recovery.**

A breaking news event can instantly drive millions of concurrent users to the platform. If the system fails for even five minutes, it is national news.

Here is how the recommendations and the wider architecture would change for an enterprise news organization of this scale.

1\. The Technology Stack Shift
------------------------------

At BBC scale, a single monolithic backend is replaced by a **decoupled, microservices, or federated API architecture**.

```
                           [ Edge CDN (Fastly / Cloudflare) ]
                                          |
                        [ BFF / API Gateway (Node.js / GraphQL) ]
                                          |
         +--------------------------------+-------------------------------+
         |                                |                               |
[ Editorial CMS ]                [ Content Delivery Service ]    [ Media & Transcoding ]
 (.NET / Java / Go)                   (Go / Node.js)               (AWS Elemental / S3)
```

### The Backend Tier: TypeScript & Go/Java/ .NET

-   **The Winner for Delivery:** **TypeScript (Node.js)** remains highly recommended, but specifically for the **BFF (Backend-for-Frontend)** or API Gateway layer. The BBC actually uses Node.js heavily for its web rendering tier because of its asynchronous I/O performance and the massive talent pool available in the UK and globally.
-   **The Winner for Core Services & Workflows:** For the heavy-lifting core services (like user permission management, editorial workflows, and localization engines), **.NET**, **Java**, or **Go** become the standards. They offer robust typing, enterprise-grade stability, and seamless integration with legacy newsroom systems (like ENPS or AP ENPS).
-   **The Swift + Vapor Verdict:** **Ruled out.** While technically brilliant, a major news organization cannot risk building core infrastructure on a niche server-side technology. The talent pool of server-side Swift developers is too small to sustain round-the-clock global engineering teams.

2\. The Database & Storage Strategy
-----------------------------------

The simple PostgreSQL database and SQLite cache model must scale out to handle massive write distribution and global read replication.

-   **Primary Database:** A globally distributed database cluster. Instead of standard PostgreSQL, you would use **Amazon Aurora PostgreSQL** (with multi-region read replicas) or a distributed SQL database like **Google Cloud Spanner** or **CockroachDB**. This ensures that if an entire AWS region goes dark, the CMS and public site remain online.
-   **Content Store (NoSQL):** Article body content and historical archives (which can span decades and millions of posts) are often stored in document stores like **MongoDB** or **DynamoDB** because of their horizontal scaling capabilities and flexible schemas for different content types (live blogs, photo galleries, long-form articles).

3\. Caching: The Real Savior (Edge Caching)
-------------------------------------------

At this scale, **99.9% of the public-facing read traffic must never hit your origin servers.**

-   **The Edge CDN:** You would use enterprise-tier CDNs like **Fastly** or **Cloudflare** with instant purging capabilities.
-   **Surrogate Keys / Cache Tagging:** When an editor updates a breaking news article, the CMS sends a purge request to the CDN using a unique surrogate key (e.g., `article-12345`). Within milliseconds, that specific page is invalidated globally across all edge nodes, while the rest of the site remains cached.
-   **Stale-While-Revalidate:** The CDN is configured to serve stale content to users for a few seconds while fetching the freshly updated article in the background. This prevents “cache stampedes” (where thousands of simultaneous requests hit the database at the exact moment a cache expires).

4\. Media, Video, and Podcasts
------------------------------

At BBC scale, you cannot host or transcode video files yourself on a standard server block.

-   **Transcoding Pipelines:** Raw video and audio uploaded by journalists must be pushed to a dedicated pipeline like **AWS Elemental MediaConvert**. It automatically packages the video into multiple bitrates and formats (HLS/DASH) for seamless playback on everything from low-bandwidth mobile devices in rural areas to 4K smart TVs.
-   **Multi-CDN Strategy:** To deliver video and podcast files globally without buffering during high-traffic events, you distribute the bandwidth load across multiple CDN providers (e.g., Akamai, Cloudflare, and Fastly) dynamically based on the user’s geographic performance.

5\. Security & RBAC (Role-Based Access Control)
-----------------------------------------------

Security changes from a simple database table to a compliance-heavy identity provider.

-   **Identity & Access Management (IAM):** Rather than handling passwords in-house, the CMS integrates with enterprise identity providers (like **Okta** or **Ping Identity**) via SAML/OIDC.
-   **Audit Trails:** For a major news outlet, an unauthorized post can cause geopolitical or financial chaos. The system requires cryptographic audit logs tracking exactly who wrote, edited, approved, and published every single character.

* * *

**Prompt:** *And how would the architecture change for a small publication with only about 1000-3000 views per day?*

For a small publication pulling in 1,000 to 3,000 views per day, the architectural priorities flip completely. At this scale, you are no longer architecting for global redundancy or massive concurrent spikes. Instead, your primary design goals are:

1.  **Extreme cost efficiency** (ideally keeping hosting under $10–$20/month, or even entirely free).
2.  **Minimal maintenance overhead** (no complex Kubernetes clusters, microservices, or multi-node databases to monitor).
3.  **Simplicity and longevity** (the system should be able to run for years without touching it).

Here is how the architecture collapses into a highly optimized, elegant, and low-cost setup.

1\. The Technology Stack Shift
------------------------------

At 3,000 views a day, a single user is visiting the site roughly once every 30 seconds on average. Any modern CPU can handle this load in its sleep.

### The Backend Tier: Swift + Vapor or PHP (Vanilla/Lightweight)

-   **The Winner for Performance & Fun:** **Swift + Vapor**. Because Vapor compiles to a single, highly efficient native binary, it uses an incredibly small memory footprint (often under 30–50MB of RAM at idle). You can easily run the entire application, database, and cache on the cheapest $4–$5/month VPS (like a shared-CPU Hetzner or DigitalOcean droplet) with resources to spare.
-   **The Winner for Simplicity:** **Vanilla PHP** or a micro-framework. It is virtually zero-maintenance, runs on any cheap hosting provider, and has zero compile steps.
-   **TypeScript & .NET Verdict:** Node.js/TypeScript is still fine, but its memory footprint is higher than Swift. .NET is massive overkill and will require a larger, more expensive VPS just to keep the runtime comfortable.

2\. The Storage & Database Strategy
-----------------------------------

You do not need dedicated, managed database instances.

-   **The Database: SQLite**
    -   **Why:** For 3,000 views a day, **SQLite is the absolute king**. Modern SQLite handles concurrent reads incredibly well (especially in WAL mode), requires zero server setup, and exists as a single file on disk. This makes backups as simple as copying the database file to an S3 bucket once a day.
    -   You do not need PostgreSQL, MySQL, or Mongo. SQLite can easily handle millions of rows and a few thousand requests a day without breaking a sweat.

3\. Caching & Frontend: Static Generation (SSG) or Edge-Cached SSR
------------------------------------------------------------------

The absolute best architecture for a small publication is to **decouple the CMS from the frontend entirely.**

```
[ Writer / Admin ] ---> [ CMS (Vapor or PHP + SQLite) ] 
                               |
                   (Generates static HTML / RSS)
                               |
                               v
               [ Cloudflare Pages / Vercel (Free) ] <---> [ Reader ]
```

### Option A: Static Site Generation (SSG) — *Highly Recommended*

-   Your CMS is just a private, password-protected backend.
-   When you publish a post, the backend compiles the website into static HTML, CSS, and RSS files and pushes them to a free static hosting service like **Cloudflare Pages**, **GitHub Pages**, or **Vercel**.
-   **The Result:** Your public site is virtually un-hackable, loads instantly worldwide, has **zero server hosting costs**, and can handle a sudden spike of 1,000,000 users without you paying a single extra penny.

### Option B: Server-Side Rendering (SSR) with SQLite Cache

-   If you want dynamic features like reader comments, you run your Swift/Vapor or PHP app on a cheap $4 VPS.
-   **The Cache:** You use a simple in-memory cache (like Vapor’s built-in memory cache) or build a custom, ultra-fast read-cache using a secondary, memory-mapped SQLite database.
-   **The CDN:** You put a free **Cloudflare** proxy in front of it. Cloudflare will cache your static pages and assets at the edge, meaning 90%+ of your 3,000 daily views never even hit your $4 VPS.

4\. Media, Newsletters, and Podcasts
------------------------------------

At this scale, you want to offload specialized tasks to free or low-cost tiers.

-   **Media Storage:** Use **Cloudflare R2** or **Backblaze B2** for hosting images, podcast MP3s, and video. Both have incredibly generous free tiers (R2 gives you 10GB of storage and millions of class A/B operations per month for $0).
-   **Newsletters:** Writing a custom bulk-email engine is painful and risks your server’s IP getting blacklisted. Instead, use your backend to collect subscriber emails in your SQLite database, and use a service like **Mailgun** or **Brevo** (which has a free tier of up to 300 emails/day) to send the newsletters via a simple API call.
-   **Social Auto-Posting:** Instead of writing complex API integrations for every social media platform, your CMS can simply output a clean, standard **RSS feed**. You can then use a free automation tool (like Zapier, Make, or a simple custom cron job script) to watch that feed and post to Mastodon, Bluesky, or LinkedIn when a new item appears.

* * *

My Thoughts on the Conversation
-------------------------------

Most of its advice made a lot of sense to me, even if I might implement some of it differently. Not much was new to me in terms of technologies or ideas which, I suppose, shouldn’t be that surprising. I’ve never worked on a platform that will see a sudden spike of 1,000,000 users, but I’ve certainly worked on enough platforms to be familiar with concepts such as edge caching that make such spikes possible without the system crashing.

I really do enjoy using AI for conversations like this one. I find it very inspirational and it motivates me to actually sit down and program something, even if I never actually finish it. In this case, I didn’t really do anything after having this discussion, but I nonetheless enjoy the thought experiment.