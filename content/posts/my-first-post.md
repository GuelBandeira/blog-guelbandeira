---
title: "How to create a free blog?"
date: 2025-11-25
draft: false
series: "Definitive Guide to Hosting a Blog"
tags: ["Hugo", "Github Pages"]
---
## What tool did I use?

This blog runs at no cost, with no database and on a free host. To not lie to you, the only cost I had was the domain name, for $10, but I already had that domain for my portfolio, and you can host this blog without using your custom domain. For the blog itself this runs without any money from my pockets, and the tool I used is called [Hugo](https://gohugo.io/), a Go framework designed to build static websites. All you need to do is follow the documentation on their website, which is very easy to understand. It’s so straightforward that I found it unnecessary to extend this section further, you can replicate everything I did by simply following their instructions.

I chose the theme [Poison](https://themes.gohugo.io/themes/poison/), which I found the most pleasant, but you can choose the one that best suits you. One thing to keep in mind is that some themes, like [Hugo Blox](https://themes.gohugo.io/themes/blox-tailwind/), are built in modular blocks for you to assemble. These themes require extra attention because you will need to read their specific documentation.

---

## Which host did I pick?

This website is static, which means it doesn’t require any type of database, therefore, choosing a host becomes even simpler, since many platforms offer free hosting for static pages. Off the top of my head, I can mention Vercel, Netlify, and the well-known GitHub Pages, which is what I’m using for this blog.

The setup you need to follow if you like to add the blog in a subdomain, like I have, is to configure the DNS name to "blog" on the website you bought the domain, and add the IP of your host, in my case I added the IPs of the Github Pages. Your DNS config should look like this:

![Cloudflare Config](/images/my-first-post/cloudfareconfig.png)

Then you will need to create a plain CNAME file in your repository that will be your subdomain, and inside the CNAME file you should add the link do your subdomain, so these should look like these:

![CNAME Config](/images/my-first-post/cnamerepo.png)

Inside the CNAME is my subdomain, like this:

```
blog.guelbandeira
```

And that's it.

--- 
## Conclusion

This is how I created a blog that runs without any costs. There are other tools you can use to achieve the same results, such as [Jekyll](https://github.com/jekyll/jekyll), which has the same purpose of generating static websites. In the end, it’s mostly about choosing the tool you find most user-friendly or the one you enjoy using the most.
