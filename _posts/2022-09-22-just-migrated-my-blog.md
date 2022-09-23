---
title: Just Migrated my Blog 
date: 2022-09-22 08:00:00 +0300 
categories: [Blog] 
tags: [blog, jekyll, github, markdown, github-pages]
image:
  path: /assets/2022-09/pages.jpeg
  width: 1000
  height: 400
  alt: github pages logo

--- 

I just migrated my blog to GitHub Pages. Well, I actually created a new blog using VuePress and migrated that to GitHub Pages. [/r/technicallythetruth](https://www.reddit.com/r/technicallythetruth/). I will talk about why I did that, but before that a little primer on what GitHub Pages is and why I chose it. 

## What is GitHub Pages?
GitHub is the most famous repository hosting service. It allows you to share projects with other developers and work on them together or separately. Gitlab is a similar service to GitHub, but perhaps is better suited if you want a self-hosted environment. Both are industry standards and are used by millions of developers.

Lucky for us both GitHub and Gitlab offer a free hosting service for static websites. This means that you can host your website on GitHub or Gitlab for free. A very common misconception is that GitHub Pages can be used to host dynamic websites. This is not true. GitHub Pages is a static storage the same way GitHub is a static storage of your code. 

A static website is a website that is generated once and then served to the user. This means that the website is not generated on the fly for each user. This is in contrast to a dynamic website which is generated on the fly for each user, based on the user’s request and most probably by using some form of a data store.

## What is Markdown?
Markdown is a lightweight markup language with plain text formatting syntax. It is entirely designed to be readable as-is, without looking like it has been marked up with tags or formatting instructions. It is designed so that it can be converted to HTML and many other formats with ease. It was created by John Gruber and Aaron Swartz in 2004 and released in 2006. Markdown is often used to format readme files, for writing messages in online discussion forums, and to create rich text using a plain text editor.

## What is SSG (Static Site Generation)?
Static site generation is the process of creating a static website from a set of files. The files are usually written in a markup language like Markdown. The static site generator then takes these files and generates a static website from them. The static website is then served to the user. We can use SSGs to create blogs, documentation sites, landing pages, and many other types of websites that donot require dynamic content and can be generated once and then served to the user.

Jekyll is a popular static site generator. It is written in Ruby and is used by GitHub, which itself is built on Ruby, out of the box. This means that you can write the contents of you website easily using Markdown and then use Jekyll to generate a static website from it. Jekyll is very easy to use and is very popular among developers. 

## Why did I choose GitHub Pages?
There are many reasons why I chose GitHub Pages. First of all, I have been using GitHub for a long time and I am very familiar with it. It is also very popular and is used by millions of developers. 

"That's not good enough reason for me" I hear you say stranger, well, look at it this way. GitHub pages is much more easier to setup than GitLab. Granted you can have very advanced configuration at your finger tips on GitLab, but for a simple blog, GitHub Pages is much easier to setup. A fork and an edit is all you need to get started.

## Why did I move over from VuePress?
VuePress is a similar static site generator to Jekyll. It is written in JavaScript and is used by Vue.js, which itself is built on JavaScript, out of the box. I built the first version of this blog using VuePress. I was very happy with it and I still am. However, I wanted to try out Jekyll and see how it compares to VuePress. A lighthouse comparison on a localhost network showed that Jekyll was a bit faster than VuePress, but more importantly, It had much better points on SEO and Accessibility.

## Conclusion
I hope you enjoyed this article. If you have any questions, please feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/elijahma/). I will be happy to answer them. 

## References
- [https://pages.github.com/](https://pages.github.com/)
- [https://en.wikipedia.org/wiki/GitHub](https://en.wikipedia.org/wiki/GitHub)
- [https://en.wikipedia.org/wiki/GitLab](https://en.wikipedia.org/wiki/GitLab)
- [https://en.wikipedia.org/wiki/Markdown](https://en.wikipedia.org/wiki/Markdown)
- [https://en.wikipedia.org/wiki/Static_web_page](https://en.wikipedia.org/wiki/Static_web_page)
- [https://en.wikipedia.org/wiki/Static_site_generator](https://en.wikipedia.org/wiki/Static_site_generator)
