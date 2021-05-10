---
date: 2020-11-09T11:13:32-04:00
description: ""
featured_image: "/images/basket_feature.jpg"
title: "Market Basket Analysis"
show_reading_time: true
---





## What is market basket analysis?

Market basket analysis (also known as affinity analysis) is specific implementation of *association rule learning/mining*--an unsupervised machine learning that seeks to uncover potentially valuable relationships (associations) between the items/products consumers buy from point of sale *transaction data* . For example, a grocery store might use market basket analysis to discover that 40% of the time when their customers buy cereal, they also buy bananas. Businesses used market basket analysis for things like:

  - Recommendation Engines (think Netflix's , TikTok, & Youtube)
  - Marketing Promotions
  - Product Bundling
  - Targeted Advertisement
  - Product Placement

The mining algorithm looks through retailer transaction data to find items/products that are frequently purchased together (*itemsets*) and then generates simple if/then association rules between them.



### Association Rules

Association rules are simple *if,then* rules containing itemsets on both the left hand side (lhs) and right hand side (rhs) like the examples below.
$$
\{cereal\} \rightarrow \{milk, bananas\} \\
\{ketchup, pickles\} \rightarrow \{mustard\} \\
\{\emptyset\} \rightarrow \{eggs\}
$$
Itemsets don't need to have more than one item and itemsets on the lhs (only) can be empty. When a rule has a lhs that is the empty set, it is really just giving you information about the *independent* support, confidence, etc. for the itemset on the rhs.  

Here's a more precise definition:

>"Let D be the set of transactions represented in Table 10.1, where each transaction T in D represents a set of items contained in I. Suppose that we have a particular set of items A (e.g., beans and squash), and another set of items B (e.g., asparagus). Thenan association rule takes the form if A, then B (i.e., A ⇒ B), where the antecedent A and the consequent B are proper subsets of I, and A and B are mutually exclusive. This definition would exclude, for example, trivial rules such as if beans and squash, then beans." (Larose, 2005)

One of the biggest challenges in finding strong/interesting association rules is the size of the search space--the number of possible association rules. With only 100 products, there are $6.4 * 10^{31}$



### Rule Metrics

Every item/product a retailer sells, along with every possible combination of those products is the basis of a potential association rule. But the purpose of market basket analysis is to find valuable association rules that can be used to drive marketing and other business decisions. The three most important metrics are: *support*, *confidence*, and *lift*.  


#### Support

Support is a measure of how frequently an itemset appears in the transaction dataset. In other words, the proportion of transactions that contain the itemset.

Suppose we have a dataset that contains only the following transactions:

  - Transaction 1: bananas were purchased
  - Transaction 2: milk was purchased
  - Transaction 3: cereal was purchased
  - Transaction 4: bananas and milk were purchased
  - Transaction 5: bananas were purchased

The support for itemset {bananas} is $2/5 = 0.4$

The support for itemset {bananas, milk} is $1/5 = 0.2$


#### Confidence

Confidence is a measure of how often an association rule is true. That is, how often it is the case that when the itemset on the lhs was purchased, the itemset on the rhs was also purchased (in the same transaction).
$$
\underline{Support\{ lhs \} + Support\{ rhs \}} \\ Support\{lhs\}
$$
Confidence can also be understood as a conditional probability of the rhs itemset appearing in a transaction given that the lhs itemset appears in the transaction. $$Pr(\{rhs\}|\{lhs\})$$.


#### Lift

Lift is $$\underline{Support\{ lhs \} + Support\{ rhs \}} \\ Support\{lhs\} * Support\{rhs\}
$$
Unlike confidence, the value of lift is the same regardless of whether the rhs or lhs comes first.

# Example : Electronidex

Electronidex is a online electronics retailer selling desktops, laptops, computer accessories, and other electronics products. We have a dataset containing 30 days of online transactions from 2017. I use the `apriori()` function from the arules package to mine for association rules.


```{r load pkgs, message=FALSE, warning=FALSE}
# load standard pkgs
library(tidyverse)
library(here)
library(kableExtra)

# load pkgs for market basket analysis
library(arules)
library(arulesViz)
```

The raw Electronidex data contains rows that each represent a single online transaction from a 30 day period in 2017. The columns each contain one item/product the customer purchased in that transaction. Thus if a customer only purchased on item, the row representing that transaction will contain only one column with the name of the product they purchased. Here is a snippet of what the data looks like when opened in Excel.

{{< figure src="/images/snip_raw_electronidex_data1.png" >}}

For the mining algorithm to work, the above raw data needs to be transformed into a *sparse matrix*  (also called basket format), where each column represents one item/product the company sells and the items purchased in each transaction are represented as ones.  

```{r snip2, eval=FALSE}
transactions_matrix <- as(transactions, 'matrix')

transaction_df <- as.data.frame(transactions_matrix)

transactions_num_df <- 1 * transaction_df
```
Here's a snip of what the data in a sparse matrix would look like if it were transformed into a dataframe. Each row represents a transaction and each column represents one item/product the company sells. The item(s) purchased in each tranaction are recorded as 1s in the appropriate columns.

{{< figure src="/images/snip_trandformed_eletronidex_data1.png" >}}

However, you can avoid transforming the raw transaction data by loading it into R using the **read.transactions()** function from the arules package instead of read.csv().

```{r read.trans}
transactions <- read.transactions(
  here('Data', 'ElectronidexTransactions2017.csv'),
  format = 'basket',
  sep = ',',
  rm.duplicates = TRUE)
```

```{r}
# verify import
transactions
```



## Summarize Data

Let's get a summary of what's in the data.
```{r}
summary(transactions)
```
From the output, we can ascertain that there are 9,835 rows (transactions) and 125 items that could potentially appear in association rules. We can also see the 5 most frequently purchased items are iMac, HP Laptops, CYBERPOWER Gamer
Desktops, Apple Earpods, and Apple MacBook Airs. Lastly, we can see that most transactions are for between 1 and 4 items.

To get a visual representation of the most frequently purchased items, we can use use the itemFrequencyPlot() function from arules.

```{r}
# plot 10 most frequent items
itemFrequencyPlot(transactions,
                  topN = 10,
                  type = 'absolute',
                  cex.names = 0.8) #reduces label size
```

### Inspect Tractions

To look at the items purchased particular transactions, e.g. large transactions with 22+ items, we can use the inspect() and size() functions from the arules package.

```{r, include=FALSE}
# to change long outputs scrolling, add mat.height='100px' to code chunk
options(width = 60)
local({
  hook_output <- knitr::knit_hooks$get('output')
  knitr::knit_hooks$set(output = function(x, options) {
    if (!is.null(options$max.height)) options$attr.output <- c(
      options$attr.output,
      sprintf('style="max-height: %s;"', options$max.height)
    )
    hook_output(x, options)
  })
})
```

```{r, max.height='200px'}
inspect(transactions[size(transactions) >= 25])
```


## Generate Association Rules

To mine the data for association rules using arules, you can use the `apriori()` function, and

```{r}
# find all potentially valuable association rules
ruleset_01 <- transactions %>%
  apriori()
```
If you don't get any rules (as in the example above) that may be because the default support and confidence parameters, which are 0.1 and 0.8 respectively, are too high.

To change the default settings, add a `parameter =` argument to your `apriori()` function and specify lower values for `supp`, `conf`, and/or other parameters. Note: if you want to change 2 or more parameters embed them in a `list()`.

```{r}
ruleset_01 <- transactions %>%
  apriori(parameter = list(supp = 0.02, conf = 0.25))
```

### Summarize and Inspect Rules

Use the `summary()` function to get more information.

```{r}
summary(ruleset_01)
```
This tells us that the majority of the rules generated (46) contain 2 items, and we also get summary statistics on the rule metrics, along with coverage and count.

Here's the defintion of coverage from rdrr.io:

>"Coverage (also called cover or LHS-support) is the support of the left-hand-side of the rule, i.e., supp(X). It represents a measure of to how often the rule can be applied."

When it comes to a single rule, count is simply the number of times the itemsets in the rule appear in the dataset. The summary statistics are just telling you about the range and variation for count among the 388 rules generated.  

To look at a subset of rules generated, we can pipe the ruleset into the `head()` function and then `inspect()`. (Note: the `kbl()` and `kable_paper()` functions at the end of the pipeline are part of the kableExtra package and just used to turn the standard console output into a more readable table. They are not necessary.)

```{r}
ruleset_01 %>%
  head(n = 10) %>%
  inspect() %>%
  kbl() %>%
  kable_paper('striped', 'condensed')
```

The first rule has an empty lhs itemsets. As mentioned earlier, these rules are about transactions where just the item(s) on the rhs were purchased. To avoid rules with an empty antecedent (lhs) add  `minlen = 2` to the `parameter = list()` in the `apriori()` function.



### Sort Rules

To sort the rules by a paricular metric, e.g. confidence, we just need to add a `sort()` function into our pipeline.

```{r}
ruleset_01 %>%
  head(n = 10) %>%
  sort(by = 'confidence') %>%
  inspect() %>%
  kbl() %>%
  kable_paper('striped', 'condensed')
```


### Redundant Rules

To find out if any of the rules generated are redundant run the `is.redundant()` function on your ruleset.

```{r}
is.redundant(ruleset_01)
```



## Vizualize Rules

The `arulesViz` package enables easy creation of both static and interactive visualizations of association

By selecting a item/product from the "Select by Id" menu, users can highlight the rules that are connected to that item/product.

```{r}
plot(ruleset_01,
     method = "graph",
     engine = "htmlwidget"
     )
```

For example, if you select 'Dell Laptop', you can see that the idem only features in one rule. (use your mouse scroll wheel to read the labels on the graph nodes).


[Link to Project GitHub Repository](https://github.com/kpiatti/market-basket-analysis)
