<figure><img loading="lazy" decoding="async" src="a-robot-trapped-in-a-maze.jpg" alt="AI-generated image of a robot lost in a maze"><figcaption>AI-generated image of a robot lost in a maze</figcaption></figure>

I recently stumbled upon an article at [Ars Technica](https://arstechnica.com/ai/2025/03/cloudflare-turns-ai-against-itself-with-endless-maze-of-irrelevant-facts/) about Cloudflare turning AI against itself. I thought it was a very interesting strategy in the battle to try to protect creative content against AI training models.

> On Wednesday, web infrastructure provider Cloudflare announced a new feature called “[AI Labyrinth](https://blog.cloudflare.com/ai-labyrinth/)” that aims to combat unauthorized AI data scraping by serving fake AI-generated content to bots. The tool will attempt to thwart AI companies that crawl websites without permission to collect training data for large language models that power AI assistants like ChatGPT.
> 
> \[…\]
> 
> Instead of simply blocking bots, Cloudflare’s new system lures them into a “maze” of realistic-looking but irrelevant pages, wasting the crawler’s computing resources. The approach is a notable shift from the standard block-and-defend strategy used by most website protection services. Cloudflare says blocking bots sometimes backfires because it alerts the crawler’s operators that they’ve been detected.
> 
> “When we detect unauthorized crawling, rather than blocking the request, we will link to a series of AI-generated pages that are convincing enough to entice a crawler to traverse them,” writes Cloudflare. “But while real looking, this content is not actually the content of the site we are protecting, so the crawler wastes time and resources.”
> 
> The company says the content served to bots is deliberately irrelevant to the website being crawled, but it is carefully sourced or generated using real scientific facts—such as neutral information about biology, physics, or mathematics—to avoid spreading misinformation (whether this approach effectively prevents misinformation, however, remains unproven). Cloudflare creates this content using its [Workers AI](https://developers.cloudflare.com/workers-ai/) service, a commercial platform that runs AI tasks.
> 
> [Ars Technica](https://arstechnica.com/ai/2025/03/cloudflare-turns-ai-against-itself-with-endless-maze-of-irrelevant-facts/)

The article itself is based on a blog post by Cloudflare which you can find [here](https://blog.cloudflare.com/ai-labyrinth/).

Of course, not everyone cares about whether their websites are scraped by AI but there are plenty of people who do. Large media companies like the New York Times are even going so far as to [sue companies like OpenAI](https://arstechnica.com/tech-policy/2025/04/judge-doesnt-buy-openai-argument-nyts-own-reporting-weakens-copyright-suit/) for using their intellectual property without payment or permission.

I think it’s an interesting approach to solving a problem that is unsurprisingly getting more and more relevant as more models are trained on real data found on the internet. That said, I don’t bother opting into such things for my blogs because I figure that if I publish it openly on the internet, it’s fair game. I understand why someone would be upset if it managed to get through any paywalls though.

Here are the links to the posts:

-   The original blog post: [https://blog.cloudflare.com/ai-labyrinth/](https://blog.cloudflare.com/ai-labyrinth/)
-   The article written about it at Ars Technica: [https://arstechnica.com/ai/2025/03/cloudflare-turns-ai-against-itself-with-endless-maze-of-irrelevant-facts/](https://arstechnica.com/ai/2025/03/cloudflare-turns-ai-against-itself-with-endless-maze-of-irrelevant-facts/)