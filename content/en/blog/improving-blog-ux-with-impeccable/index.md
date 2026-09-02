---
title: "How I Used Impeccable to Review My Own Blog's UX"
date: 2026-09-02T08:00:00-04:00
draft: false
translationKey: "improving-ux-with-impeccable"
tags: ["ux", "design", "ai", "hugo", "accessibility"]
description: "How I ran an AI-guided design review session on this blog using Impeccable, and what it found and fixed — from hidden bugs to accessibility."
---

I'm not a designer. I built this blog on my own, with help from LLMs to structure the Hugo setup, pick a theme, and write the templates, but I never stopped to look at the result with a critical UX eye. I knew some things were off — some even visible at a glance, like tags that looked clickable but weren't — but I didn't have a process to find the rest.

That's when I decided to try **Impeccable**, a Claude Code skill built for UI design review and refinement. The idea is simple: instead of asking "make the design better" (too vague for any AI to do well), Impeccable has specific commands — one to diagnose problems, others to fix specific categories (typography, spacing, accessibility, and so on) — plus a way to actually verify a fix worked before calling it done.

I ran a full session through it, command by command, fixing and validating one item at a time before moving to the next. This post is a summary of how it went.

## The diagnosis: `/impeccable critique`

I started without pointing at anything specific, and Impeccable picked the homepage as the target — makes sense, it's the front door of the site. The `critique` command does two things at once: an actual design "read" (like a design director evaluating hierarchy, clarity, accessibility) and an automated scanner looking for known problem patterns in the code. The two run independently of each other, with no cross-influence, and only afterward do I see them combined.

The result came back with a quality score (16 out of 28 applicable points — "acceptable," neither good nor terrible) and a list of problems, ranked from most to least severe:

- The homepage bio section was completely empty (I haven't written that content yet — saved for later, once the material is ready).
- There was no path from the homepage to the blog posts other than a link hidden in the nav menu.
- Post tags (like `#Groq`, `#OpenAI`) looked like clickable pills, but were just decorative text with no link at all.
- A silly CSS bug in the header, which was supposed to stay pinned to the top while scrolling but didn't.
- Accessibility labels in English mixed into an otherwise Portuguese page, and no description configured for when someone shares the homepage link on social media.

## Fixing what the critique found

I went one at a time, validating each fix before moving to the next:

**Tags became real links.** Each tag now points to an actual page listing every post under that topic — including a page I didn't even know Hugo was already generating automatically.

**The homepage got a preview of the latest posts.** While working on this, I found a much bigger hidden bug: the site's UI translations (strings like "share," "related posts," etc.) were completely broken across the entire site — a file had ended up in the wrong folder months earlier, and nobody had noticed because the site simply showed blank text in its place, with no error at all.

**The header became genuinely sticky.** What looked like just a contradictory CSS class turned into a deeper investigation: even after removing the wrong class, the header still scrolled away with the page. The real cause was structural — the parent element wasn't leaving enough room for the "stick to the top" effect to actually work. I only found this by testing for real in a browser, scrolling the page, instead of trusting the code alone.

**Accessibility and sharing.** Labels correctly translated into both languages, plus a default description for when the homepage gets shared on WhatsApp or LinkedIn.

## `/impeccable typeset`: the wrong font for the job

Next I asked specifically for a typography review, because I felt the reading experience on posts wasn't great. The reason: the entire blog — including the running body text of articles — used a monospace font (the kind you'd see in a terminal or code editor). That fits the site's `{amaral stack}` brand identity, but it's known to be worse for reading long paragraphs, since every letter takes up the same width and the eye has a harder time recognizing word shapes.

The fix kept the monospace font where it makes sense — logo, nav, dates, code — and brought in a dedicated reading font (IBM Plex Sans) just for the body of posts. I also gave article headings real visual weight (before, they were the same weight as regular text, just bigger) and narrowed the reading column, which had been too wide for comfort.

## `/impeccable polish`: the general quality pass

This command runs a broad sweep, testing the whole site the way a real visitor would. It found two problems I hadn't caught:

- The tags at the top of each individual post (different from the ones in the list view) had a typo in the link-building code, producing broken URLs on every published post.
- Almost no clickable element on the site had a visible focus indicator for keyboard navigation — only 2 buttons, out of more than 20 elements, had that care. That matters for accessibility: without visible focus, someone navigating with a keyboard alone can't tell where they are on the page.

## `/impeccable layout`: spacing and hierarchy

Finally, a structure and spacing review. It found two layout problems that only showed up at intermediate screen widths (like a tablet): tags on a post with many labels would wrap to a second line misaligned, leaving one tag stranded on its own; and long titles would break mid-word in the middle of a compound term (like "GLM-5.1" splitting into "GLM-" on one line and "5.1" on the next). It also fixed the "related posts" card so it wouldn't leave an odd empty gap when there's only one related post to show.

## What's still open

The homepage bio is still intentionally empty — I haven't written that copy yet. And the tag listing page (the one the chips now correctly point to) still uses the theme's bare, unstyled default look, without the same visual care as the rest of the site. Both are saved for a future round.

## Was it worth it?

What stood out most was how many of the real problems weren't "ugly" — they were invisible until someone actually tested them: the header that didn't stick, the broken translations, the tag link with an extra character in the middle of the URL. None of those show up just by calmly reading the code; only after running the site in a real browser and actually trying to use it. Having a structured process for that — diagnose, fix by category, actually verify before moving on — is what made the difference for someone like me, who doesn't have trained design instincts but can recognize a problem once it's pointed out.
