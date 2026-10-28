EchoCache: Stop paying twice for the same AI question

The Problem
If you've ever built an AI-powered app, you know the pain: every single prompt sent to an LLM costs money and makes your users wait 2 to 10 seconds for a response. Worse yet, users naturally ask the same core question in a dozen different ways ("How do I reset my password?" vs. "Forgot password, what do I do?" vs. "Reset my login").
Traditional caches look at these as totally different strings, so your app makes three separate, paid, slow API calls for the exact same underlying intent.



Who is Facing It?
This hits student teams, hackathon participants, and early-stage indie developers the most. When you're running on a tight budget or a free tier, a sudden traffic spike or a loop bug during 
a demo can completely exhaust your API credits and slow your app to a crawl within hours.


The Idea
What if our backend had a memory? Instead of treating natural language like rigid code, what if our system could understand that two differently worded questions mean the exact same thing—and instantly serve the cached answer for free in milliseconds?


The Solution
EchoCache is a high-performance backend service engineered with Spring Boot, Redis, and PostgreSQL that acts as an intelligent optimization layer for LLM applications.
Here is how it saves your wallet and your latency:
1.The Exact Match: It checks Redis first for identical strings ($\sim$5 ms, $0 cost).
2.The Semantic Layer: If that misses, it runs a semantic vector similarity check. If a past question matches the same intent, it pulls the answer instantly ($\sim$50 ms, $0 cost).
3.The Safety Net: Genuine cache misses call the LLM and save the pair. Plus, built-in Bucket4j rate-limiting protects against spam (returning 429 Retry-After), and automatic provider fallbacks keep your app alive if the primary LLM goes down.
4.Total Visibility: It tracks exact telemetry—showing developers their hit rates, latency drops, and literal money saved in real-time.
