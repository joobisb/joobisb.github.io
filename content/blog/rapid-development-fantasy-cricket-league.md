---
title: "Vibe coded a fantasy cricket app"
date: 2025-06-02
draft: false
tags: ["development", "AI", "claude", "cursor", "supabase", "rapid-development", "cricket", "fantasy-sports", "vibe-coding"]
description: "A brutally honest take on building a fantasy cricket league using 'wipe coding' - rapid, iterative development with AI assistance. What worked, what didn't, and lessons learned."
---
I just wrapped up vibe coding a fantasy cricket league. Here's my take on what worked, what didn't, and what I learned along the way.

Please do check it out at [playz.one](https://playz.one) ✨

## The Project: Fantasy Cricket League

We were planning our annual cricket tournament at [KeyValue](https://keyvalue.systems), and as part of that, we ran a fantasy cricket league. Everyone could create teams by picking their favorite colleagues similar to [Dream11](https://www.dream11.com/), where points are awarded based on real-time match performance.

Previously, we managed everything using Google Sheets and Google Apps Script. I’d been wanting to vibe code something for a while, so this was perfect. Given the time constraints, I was curious to see how much I could pull off through vibe coding.

## The Tech Stack

The first step was deciding what to build with. I had been wanting to try [Lovable](https://lovable.dev/) for a while, and I wasn’t aiming to build a separate backend service. I wanted something that came with a backend out of the box, so decided to use [Supabase](https://supabase.com). Since Lovable has direct integration with Supabase, it made things easier.

Later on, added few other things as well

- PostHog
- Grafana
- Cloudflare
- Github Actions

I'll talk about these as we go along.

## Here's how it went

I brainstormed the idea a bit using ChatGPT and came up with a rough layout of how the pages and the initial setup should look. I then fed that prompt to Lovable.

Found ChatGPT 4o better than ChatGPT 4.5 for brainstorming, though its mentioned 4.5 is better for exploring ideas 🤷‍♂️.


![](/images/vibe-coding/1.png)  

&nbsp;


Lovable generated an initial version pretty quickly. It gave me a solid outline of all the pages I needed, which worked really well. I also integrated Supabase with Lovable, and the integration process was super smooth.

It was able to generate the initial API integrations I needed, which helped speed things up further.

The initial version looked like this.

![](/images/vibe-coding/2.png)
&nbsp;


This was a good starting point. But ofcourse, had to go through multiple iterations to get it to the final version.

After some iterations, Lovable credits ran out. I had Cursor subscription, so I started using it with `claude-4-sonnet`. To my surprise cursor with `claude-4-sonnet` in agent mode was working at par with Lovable. The only thing missing was the Supabase integration.

Fortunately, Supabase's MCP server was available, and I integrated it with Cursor. This made Cursor feel like it was running in “superagent” mode. It sped things up a lot.

Having MCP server was a game changer. The model could generate the code by verifying changes against the database, making development much faster with minimal "please fix this" prompts.

Vibe coding felt fun that I impulse bought a domain because, why not?

Since Cloudflare has a very generous free plan with DDoS protection, reverse-proxy, workers, etc. I migrated the domain from GoDaddy to Cloudflare.

Next, I wanted to monitor traffic and app performance. Supabase does provide monitoring, and also offers Supabase+Grafana for more detailed insights. So, I used Grafana cloud for all my monitoring needs.

I also wanted to track how users were interacting with the app, so I integrated PostHog for product analytics. A PostHog MCP server would have sped up the development, but Cursor did an okay job here too.

And that’s how I ended up with [playz.one](https://playz.one)

![](/images/vibe-coding/3.png)

## The Good

### *Claude Sonnet-4 Is a Game Changer*

Sonnet 4 really impressed me. The kind of pattern recognition and auto-fixing saved me hours of manual work. It's genuinely good at understanding context and applying fixes consistently. I did play around with `gemini-2.5-pro` as well, didn't find it as compelling as `claude-4-sonnet`. To be fair, most of my prompting was done with claude, so it better context.

Also, the agent’s ability to use command-line tools and grep makes it much more powerful.

### *Get Your Basics Right First*

This sounds obvious, but I can't stress this enough. When I focused on getting the fundamental architecture solid before going all-in, everything else fell into place much smoother.

### *Supabase (When You Understand It)*

Supabase was powerful once I wrapped my head around it. The Row level Security(RLS) and the `SECURITY DEFINER` functions were particularly useful which solved a lot of my permission headaches. This whole process also helped me appreciate how far you can stretch Postgres.

## The Painful

### *Git commit frequently*

Cursor would sometimes hallucinate and get stuck in loops, especially when context became complex. I learned to commit frequently *really* frequently, because you never know what's actually been implemented versus what Cursor thinks it implemented. Once you have a working version, make sure you commit it.

### *Working code, but not optimized*

I didn’t expect perfectly optimized code, but there were multiple instances where several APIs were being called when a single one would have sufficed. So it's important to understand the code and flow. Once you figure out the optimal approach, it's better to guide the model rather than asking it to “optimize” in one go.

### *Missing features*

Both Cursor and Lovable would benefit from a voice mode. When you're in the zone, typing out long explanations breaks the rhythm. I personally found it very productive to use ChatGPT’s voice mode and then paste the prompts into Cursor. That worked well for me, though of course, preferences may vary.


## Technical Gotchas and Workarounds

### *Postgres Backups*

I was using free plan of supabase, and it did not have backups. Hence wrote a Github action to take daily backups of the database, and stored it initially in GH as artifacts and later on moved it to a Cloudflare R2 bucket.

### *Row Level Security (RLS)* in Supabase
I had to work around RLS in several places, not because it's bad, but because when you're moving fast, the security model can become a bottleneck if not planned well.

`SECURITY DEFINER` functions help, but you need to be cautious of security path hijacking. Running with elevated privileges is powerful, but dangerous if misused.

### *Domain and Cloudflare Setup*
I used Cloudflare workers to reverse proxy a subdomain for PostHog analytics. Even subdomains get blanket blocked if they detect tracking/analytics, so be aware of that.

A CNAME record would’ve worked, but PostHog required rewriting host headers which is only available on Cloudlare's enterprise plan. So, the alternative was to use Cloudflare Workers, which come with a 100k requests/day free quota, more than enough for my use case.


## Final Thoughts

I think we are at a stage where we can't go back to a world without AI-assisted development.
But at the same time, I think it's very crucial to get your fundamentals right to make the most out of AI. Those who understand this would be the ones who can leverage AI to its full potential.

Would I do it again? Absolutely. But next time, I'm starting with a better plan and keeping that git commit habit from day one 🙂.

---

*So where is our future heading? Maybe as Jensen Huang says, You won't lose your job to AI, you'll lose your job to somebody who uses AI.* 
