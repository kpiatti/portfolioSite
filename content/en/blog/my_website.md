---
date: 2021-05-24
description: ""
featured_image: ""
title: "How I Built THIS Website"
omit_header_text: true
show_reading_time: false
draft:
---
{{< figure src="/images/website_logos.png" >}}

Several people have asked for the resources I used to create this website. I'm writing this so I can just send a link when responding to any future requests.

The three main criteria I used when choosing what tools to use:
1. Quick & easy enough to learn that I could have a basic site up and running within a day or two and add/update content without too much effort.
2. The site will look modern, polished, and professional.
3. Free or low cost (Squarespace is probably quicker & easier but costs about $15/mo).

While researching my options, I saw lots of positive comments about and found plenty of tutorials/resources for *HUGO* (an open source static site generator). After looking over some of the available themes and reading/watching a couple tutorials, I downloaded [Hugo](https://gohugo.io/) and installed it on my computer following [this](https://www.youtube.com/watch?v=G7umPCU-8xc) video tutorial (unless you already know how to install programs that you access and use via the command prompt/PowerShell, I wouldn't skip the tutorial). In addition to Hugo, you'll also need the following:

- [Hugo Theme](https://themes.gohugo.io/) (I used the [Ananke](https://themes.gohugo.io/gohugo-theme-ananke/) theme)
- [GitHub Account](https://github.com/)
- a text editor designed for coding (I like Github's [Atom](https://atom.io/) editor)
- a way to stage, commit, and push changes to your site's Github repo (you can do all that from the GitHub pane inside Atom, but I also like [GitHub Desktop Application](https://desktop.github.com/)...I'm not shilling for GitHub, their stuff's just good.)
- you can publish your site from your GitHub account using GitHub pages (as you will see if you follow the portfolio tutorial I link to below). However, if you want to publish your site on a custom domain (e.g. www.yourname.com), you will need a [Netlify](https://www.netlify.com/) account & custom domain.

&nbsp;
#### Step-by-Step Tutorial
I didn't follow his process exactly, but I used [this](https://www.youtube.com/watch?v=mEZ1Hj5yQ-8) step-by-step tutorial by the Data Professor demonstrating how to create a data science portfolio using Hugo.

&nbsp;
#### Code Chunks
I can't remember if this is covered in the Data Professor's tutorial, but adding code chunks to project pages on your site is effortless. All Hugo pages in your site are generated from markdown files, so all you have to do is export your code as a markdown file ([this](https://jupyterlab.readthedocs.io/en/stable/user/export.html) link shows how to do that in Jupyter).

&nbsp;
#### Images
I got most of the decorative images on my site from [Unsplash](https://unsplash.com/) and modified them on [ezgif.com](https://ezgif.com/) (large images take a noticeably long time to render. You can dramatically decrease render time by downsizing, cropping, optimizing and/or converting images  to .webp files).

&nbsp;
#### Using Hugo
You will interact with Hugo in the command prompt. But don't worry, I still really only use it to run two commands: the ```hugo server``` command to display my website locally (localhost:1313) while I'm working on it (so I can see what changes will look like before I commit and push them to GitHub), and the ```hugo``` command to change to files in the public directory of your site's GitHub repo (what you see on localhost:1313 won't be reflected on your public website until you run ```hugo```, commit, and push the changes to your repo). Most of the time you'll be working in your text editor (e.g. Atom) and whatever tool you're using to commit changes to your repo (e.g. GitHub Desktop). Below is a screenshot of my setup as I work on this post. On the left is the site on my local server and on the right is my Atom text editor where I am writing this line.

&nbsp;

{{< figure src="/images/hugo_atom_sidebyside.png" >}}

&nbsp;
#### Final Thoughts
As long as you are happy with the defaults in your chosen template, creating your website is as easy as advertised. If you want to customize aspects of your site, you will have probably have to figure out how to create a custom shortcode or modify the CSS files in the theme directory, neither of which are trivial.

If you found this helpful, have any feedback, or just want to say hi, send me a message ([Contact Me](/resume_contact)).

&nbsp;

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [Link to GitHub Repository](https://github.com/kpiatti/portfolioSite)
