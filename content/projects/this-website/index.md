---
title: "This Website"
date: 2023-07-23T11:28:00-05:00
draft: false
icon: "/images/projects/this-website/dksite_icon.jpg"
---

{{< figure src="/images/projects/this-website/cover.png" caption="An early version of this site's home page" >}}

# Hello World!

I figured it would be funny to make this website the first entry into my projects portfolio. The site isn't even finished as I'm writing this, so this page is pushing it one step closer to completion. Though, I suppose the site will never truly be complete as long as I keep working on things. Philosophical musings aside, allow me to give you a little background on why I built this site, how it was built, and how it is structured.

# Why?

If you've seen the cover image for this post, you probably noticed how I mention that I am a programmer and drummer. I also say I'm a "few" other things, which is snarky humor on my part. In reality, there's probably at least ten things I could include in that list because I am constantly exploring something new. Since there's thousands of interesting topics to explore, I move between projects relatively quickly, which frequently causes some projects to be nearly forgotten. Therein lies my first reason for creating this website: documentation. 

By documenting my projects here, I will create a "library" of sorts that I can look back on. Each project on this site can serve as a souvenir, allowing me to relive a bygone time. Of course, nostalgia isn't the only driving factor here; leaving breadcrumbs behind will be useful if I ever need to revisit a project.

The second reason has to do with writing. I will readily admit English was my least favorite subject in school (sorry, Mr. Garrett). Though, I still recognize the great power that lies in written communication, particularly for technical subjects. Posting to this site will be good practice for clearly expressing my thoughts and ideas. Writing also provides an excellent means for introspection; I can build comprehension, generate new ideas, and identify gaps in my knowledge while writing. 

Finally, I hope people find my work interesting. I would not be where I am today if others did not publicly share their passions, creations, and knowledge. If it weren't for people like [Martin Molin (Wintergatan)](https://www.youtube.com/channel/UCcXhhVwCT6_WqjkEniejRJQ), I may not have explored CNC machining and CAD as much as I have. Of course, many other passionate individuals like Martin have inspired me. I hope that I can be inspirational in a similar fashion. While I certainly do not expect to be the reason someone pursues a new passion, it makes me happy to think someone's life might be indirectly changed by my work.

# How?

I created this site using the [Hugo](https://gohugo.io/) framework. I was originally unsure about what I tool I wanted to use, if any. I wanted something *really* simple, lightweight, and capable. I essentially wanted a blog that could be written in Markdown. Static generation was also a must; I didn't want to host a database or webserver for this. Ideally, I didn't want to deal with JSX or Vue files, so that ruled out Next.js, Nuxt, and Gatsby. That left Hugo, which seems to be a great fit so far.

Hugo comes out of the box ready for blogging. I toyed with the idea of installing a theme (of which there are many) and going off to the races, but I quickly decided against doing so. I couldn't find any themes I liked, and making my own theme seemed like a good opportunity to brush up on basic web development. At first, it was a little confusing figuring out some of Hugo's interesting (but sensible) quirks. The biggest thing I had to figure out was how to make a splash page that uses its own layout. Fortunately, Hugo makes that relatively easy: just reference the [lookup order](https://gohugo.io/templates/lookup-order/) and pick the right folder and file names. Figuring out what the heck an archetype was also fascinating (it's essentially a template for page metadata.) Finally, I am still familiarizing myself with the templating system. I don't use server-side templating often, so I can't properly evaluate Hugo's templating system, but it feels like Golang and Jinja had a ~~weird~~ interesting baby.

As mentioned before, the site is not something I have to worry about hosting myself. Luckily, because everything is static content, I can easily host it using GitHub Pages.

Overall, I've been quite happy with Hugo. I feel like I can focus on what I care about most without having to worry about fine details like collecting posts and displaying them in a list. The lack of good tooling compared to Vue or React is a slight pain point, but it's not a deal breaker for me. I'm looking forward to learning more about what Hugo has to offer.

# Plans & Structure

Finally, as promised, I'll give some insight into how I plan to structure content on this site.

Anything listed under projects will be project related (crazy, right?) Any substantial past or ongoing activity that yields a well-defined product will likely fall under this section. Usually a project will have required substantial thought in its creation; a simple script that deletes files based on if they contain the word cheese is not a project in my book. I won't belabor the point; I think you'll get a good intuition for what I consider a project just by looking at my work.

Anything that is not a project will likely end up in the blog section. I plan on recording any miscellaneous interests there. For example, I've been playing around with Stable Diffusion a lot lately, so I might talk a little bit about what I find interesting about it in a blog post. The blog section won't be strictly technical; I expect to write about plenty of non-technical matters, like the negative effects of perfectionism or something similar. To help you filter for different topics, I'll giving posts specific topic-based tags.

Depending on the content, I may create other sections other than those two. Who knows, maybe I'll upload drumming videos or something. This part of the website will be a lot more personal and less like a portfolio.

That's all I have for now. Thank you for stopping by and spending your time with me.
