---
date: 2020-11-10T11:00:59-04:00
description: ""
featured_image: "/images/product_sales_feature.jpg"
title: "The Impact of Customer Reviews on Product Sales"
show_reading_time: true
draft: TRUE
---


## Project Summary

Maybe it's a lack of self control, need of a reminder that other people are still out there, or the outcome of some nefarious [dark patterns](https://www.vox.com/recode/22351108/dark-patterns-ui-web-design-privacy) but I have definitely spent more time then I can justify reading product reviews. A few have made me laugh out loud and sometimes they seem useful. But, in my own case at least, I'm not confident product reviews make a real difference in my purchasing behavior.

The **analysis** portion of this project was to examine 3 years of data on transactions and product reviews from and electronics retailer to figure out whether there *is* a relationship between a product's sales volume and it's reviews. And if there is, what that relationship tell us.

The **modeling** part of this project was to see if we could build and train a model to accurately predict a product's sales volume from data collected on it's customer reviews.

Spoiler Alert: I did, in fact, uncover a statistically significant/meaningful relationship between products' sales volume and the type and number of reviews about each product.

Perhaps unsurprisingly, I did find a strong positive relationship between a product's three, four, and five ★ reviews and the product's sales volume. There are interesting (and few sucpicious) details that I go over in my analysis, but the broadly speaking my findings can be summarized like this: *the __more__ 3, 4, and 5 star reviews a product has, the __higher__ it's sales volume is likely to be*

Interestingly, I also found a significantly weaker but still *positive* relationship between a product's one and two star reviews and sales volume. Roughly speaking, an increase in the number of one and two star reviews correspond to a small increase in a product's sales.

Assuming the data is valid (in a live scenario I'd do some additional checking), I can think of a couple reasons why there an increase in a product's sales volume might be associated with a increase in the number of (presumably) bad 1 and 2 star reviews:
  - the 1 and 2 star reviews are *driving/causing* customers to purchase the product. Perhaps demonstrating the truth  of "any press is good press"
  - However, I think it's more likely that the higher a product's sales volume is, the more people there are who own it, and thus the more likely it is a product will receive reviews of any type.  

## Analysis Details

{{< figure src="/images/r_logo_pic.png" >}}

I did all the data wrangling, exploratory analysis, and modeling for this project using R inside the R Studio IDE and I captured everything in R Markdown files (.rmd). One of the many advantages of using R Markdown files for data analysis is that all of your work can be effortlessly *knit* into many different formats for easy sharing, including slide decks, PDFs, Word documents, and (my personal favorite) interactive html pages (requires having the knitr package—part of the tidyverse—installed in your environment). To see my work for this project in an interactive html page, click the Rpubs button below.

[{{< figure src="/images/rpubs_button.png" >}}](https://rpubs.com/kpiatti/767809)


[Link to  My GitHub Repository for this Project](https://github.com/kpiatti/product-type-sales-volume)
