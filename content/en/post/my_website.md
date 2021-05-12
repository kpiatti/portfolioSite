---
date: 2021-05-10T11:00:59-04:00
description: ""
featured_image: "/images/my_website_feature.jpg"
title: "Portfolio Website"
show_reading_time: true
---
{{< figure src="/images/website_logos.png" >}}

After the slightly humiliating experience of my 14 year-old neighbor showing me the website *she* built, I decided enough is enough, it's time I show her I'm better than she is and *I can build my own website too*.

I researched several of the most popular DIY website creation tools (e.g. WordPress, Squarespace) and ultimately settled on using *HUGO* (an open source static site generator) to build my site and Netlify to for web hosting services.

Most of the decorative images on my site are downloaded from [Unsplash](https://unsplash.com/). I used [ezgif](https://ezgif.com/) to downsize, modify, and convert the images into .webp files to optimize the rendering speed of my site pages.

Below is a list of the primary tools and resources I used to create this site.

### *What you'll need:*

- [Hugo](https://gohugo.io/) & the [Ananke](https://themes.gohugo.io/gohugo-theme-ananke/) Gohugo Theme
- [Netlify](https://www.netlify.com/) Account & custom domain. (Optional)
- GitHub [Account](https://github.com/)
- GitHub [Desktop Application](https://desktop.github.com/) (Optional)
- Text editor (like Github's [Atom](https://atom.io/) editor)

*Helpful Resources:*
- Video [tutorial](https://www.youtube.com/watch?v=G7umPCU-8xc) on how to install and setup HUGO on Windows—don't skip this unless you know how to install programs you access and use through the command prompt.
- Video [tutorial](https://www.youtube.com/watch?v=mEZ1Hj5yQ-8) on building a data science portfolio using Hugo and Github Pages.
- Jupyter [documentation](https://jupyterlab.readthedocs.io/en/stable/user/export.html) showing how to export notebooks (.ipynb) as markdown files (.md)

You will interact with Hugo in the command prompt. But don't worry, you'll really only use it to run two commands: ```hugo server``` to display your website locally while you're working on it and ```hugo``` to add the changes to your public site. Otherwise you will be working in your text editor (e.g. Atom) and whatever tool you're using to commit changes to your site's GitHub repo (e.g. GitHub Desktop). Below is a screenshot of my setup right now. On the left is my site on my local server (localhost:1313) and on the right is my Atom text editor where I am working on the project page for my website.

{{< figure src="/images/hugo_atom_sidebyside.png" >}}

All in all, I'd say the process of creating this site has been relatively painless and often quite enjoyable. If you found this helpful, have any feedback, or just want to say hi, send me a message ([Contact Me](/resume_contact)).

[Link to GitHub Repository](https://github.com/kpiatti/portfolioSite)
