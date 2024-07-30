---
title: "Measuring Willingness to Pay for Climate Change Abatement"
author: "Nikant Yadav"
date: "2024-06-24"
output:
  html_document: default
  pdf_document: default
---



![](/images/postimages/p4_1.jpg)

## **Abatement Costs**

As businesses shift towards pursuing environmental, social, and governance (ESG) means, abatement costs play a large role in discouraging companies from leniency on their environmental greenhouse gas emissions.

Specifically, abatement costs are there as **"fines"** for companies that either fail to innovate in creating greener production cycles or fail to account for potential problems and end up damaging the environment.

The most common scenario in which abatement costs are applied is for pollution and oil spills, whether accidental or intentional.

<br>

In this project, as an illustrative example, we will examine climate change mitigation. Addressing climate change often involves short-term expenses, such as the reforestation of degraded forests.

Consequently, governments may seek to understand the extent to which their citizens are willing to financially contribute towards reducing carbon emissions as a strategy for mitigating climate change.

------------------------------------------------------------------------

<br>

# **1. Summarizing the Data**

We will be using data collected from an **internet survey sponsored by the German government.**

Two common ways of obtaining information about willingness to pay (WTP) are:

-   *dichotomous choice (DC)*: presenting individuals with an amount, to which they respond with either ‘yes/willing to pay’ or ‘no/not willing to pay’ (sometimes a ‘no response’ option is also offered)

-   *a two-way payment ladder (TWPL)*: asking individuals to state the minimum and maximum amount they are willing to pay (selecting from a pre-specified list of amounts).

The German government sponsored a nationwide online survey that investigated the effect of question format (DC or TWPL) on WTP responses.

<br>


```{r}
WTP <- read_excel("excel-project-11.xlsx", sheet = "Data")
```


<div style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;">
  <p style="font-weight: bold; margin-bottom: 5px;">Code Explanation:</p>
  <p>This code imports the dataset into the project for further analysis.</p>
</div>

<br>

<br>

### **Reverse-coding variables**

Attitudes were evaluated utilizing a **Likert** scale ranging from 1 to 5, with 1 representing strong disagreement and 5 representing strong agreement.

The phrasing of the questions varied, resulting in an answer of 'strongly agree' indicating high climate change skepticism for one question and low skepticism for another. To consolidate these questions into a single index, it is necessary to recode, specifically reverse-code, certain variables.

We will reverse the following variables:

-   **`cog_2`**

-   **`cog_5`**

-   **`scepticism_6`**

-   **`scepticism_7`**

<br>

```{r}
WTP <- WTP %>%
  mutate_at(c("cog_2", "cog_5", 
    "scepticism_6", "scepticism_7"),
    funs(recode(., "1" = 5, "2" = 4, "3" = 3, 
      "4" = 2, "5" = 1)))
```


<div style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;">
  <p style="font-weight: bold; margin-bottom: 5px;">Code Explanation:</p>
  <p>This R code uses the dplyr package to transform (mutate) specific columns (cog_2, cog_5, scepticism_6, scepticism_7) in the dataframe WTP. It recodes values "1" to 5, "2" to 4, "3" to 3, "4" to 2, and "5" to 1 in these columns, effectively swapping their numeric values in a descending order.</p>
</div>


<br>

<br>

### **Creating some new variables**

For the variables **`WTP_plmin`** and **`WTP_plmax`**, create new variables with the values replaced with the actual euro values.

<br>

```{r}
wtp_euro_levels <- c(48, 72, 84, 108, 156, 192, 252, 324, 
  432, 540, 720, 960, 1200, 1440)

category_amount <- data.frame(original = 1:14, 
  new = wtp_euro_levels)

WTP <- merge(WTP, category_amount, 
  by.x = "WTP_plmin", by.y = "original", 
  all.x = TRUE) %>%
  rename(., "WTP_plmin_euro" = "new")

WTP <- merge(WTP, category_amount, 
  by.x = "WTP_plmax", by.y = "original", all.x = TRUE) %>%
  rename(., "WTP_plmax_euro" = "new")
```


<div style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;">
  <p style="font-weight: bold; margin-bottom: 5px;">Code Explanation:</p>
  <p>The code defines a vector <b>wtp_euro_levels</b> containing specified euro amounts. It then creates a dataframe <b>category_amount</b> with columns <b>original</b> (ranging from 1 to 14) and new (mapped to <b>wtp_euro_levels</b>).

The dataframe <b>WTP</b> is merged twice with <b>category_amount</b> based on columns <b>WTP_plmin</b> and <b>WTP_plmax.</b> After each merge, the merged column (new from category_amount) is renamed to <b>WTP_plmin_euro</b> and <b>WTP_plmax_euro</b>, respectively.

These operations effectively map specific euro amounts to corresponding values in WTP based on <b>WTP_plmin</b> and <b>WTP_plmax</b> columns.</p>
</div>



<br>

<br>

### **Creating Indices**

We create three new indices:

-   **`climate` :** Belief that climate change is a real phenomenon

-   **`gov_intervention` :** Preferences for government intervention to solve problems in society

-   **`pro_environment` :** Feeling of personal responsibility to act pro-environmentally

creating indices

```{r}
WTP <- WTP %>%
  
  rowwise() %>%
  mutate(., climate = rowMeans(cbind(
    scepticism_2, scepticism_6, scepticism_7))) %>%
  mutate(., gov_intervention = rowMeans(cbind(
    cog_1, cog_2, cog_3, cog_4, cog_5, cog_6))) %>%
  mutate(., pro_environment  = rowMeans(cbind(
    PN_1, PN_2, PN_3, PN_4, PN_6, PN_7))) %>%
  ungroup()
```

<br>

<br>

### **Assessing Consistency/Reliability**

There are two prevalent methods to evaluate reliability: one involves examining the correlation between items within the index, and the other employs a summary measure known as **Cronbach’s alpha.**

> Cronbach’s alpha is a way to summarize the correlations between many variables, and ranges from 0 to 1, with 0 meaning that all of the items are independent of one another, and 1 meaning that all of the items are perfectly correlated with each other. While higher values of this measure indicate that the items are closely related and therefore measure the same concept, with values that are very close to 1 (or 1), we could be concerned that our index contains redundant items 

<br>

Cronbach's alpha is computed by correlating the score for each scale item with the total score for each observation (usually individual survey respondents or test takers), and then comparing that to the variance for all individual item scores:

$$
\alpha = (\frac{k}{k - 1})(1 - \frac{\sum_{i=1}^{k} \sigma_{y_{i}}^{2}}{\sigma_{x}^{2}})
$$

where:

-   ***k*** refers to number of scale items

-   $\sigma_{y_{i}}^2$ refers to the variance associated with item i

-   $\sigma_{x}^2$ refers to the variance associated with the observed total scores

<br>

<br>

#### **Calculating Correlation Coefficients:**

-   For questions on climate change

```{r}
cor(cbind(WTP$scepticism_2, WTP$scepticism_6, 
  WTP$scepticism_7))
```
<pre><code class="hljs">##                    exaggeration not.human.activity no.evidence
## exaggeration          1.0000000          0.3904296   0.4167478
## not.human.activity    0.3904296          1.0000000   0.4624211
## no.evidence           0.4167478          0.4624211   1.0000000</code></pre>

###### Correlation table for survey items on climate change scepticism: Climate change is exaggerated (exaggeration), Human activity is not the main cause of climate change (not.human.activity), No evidence of global warming (no.evidence)

<br>

-   For questions on government behavior :

```{r}
cor(cbind(WTP$cog_1, WTP$cog_2, WTP$cog_3, 
  WTP$cog_4, WTP$cog_5, WTP$cog_6))
```
<pre><code class="hljs">##                          too.much not.pass.laws minimal.intervention
## too.much                1.0000000     0.2509464           0.32358783
## not.pass.laws           0.2509464     1.0000000           0.11761093
## minimal.intervention    0.3235878     0.1176109           1.00000000
## not.dictate             0.6823385     0.2771883           0.33476619
## indiv.freedom           0.2892567     0.4079467           0.01818617
## personal.responsibility 0.4141992     0.0828661           0.31286082
##                         not.dictate indiv.freedom personal.responsibility
## too.much                  0.6823385    0.28925672               0.4141992
## not.pass.laws             0.2771883    0.40794667               0.0828661
## minimal.intervention      0.3347662    0.01818617               0.3128608
## not.dictate               1.0000000    0.27424993               0.4597244
## indiv.freedom             0.2742499    1.00000000               0.1045843
## personal.responsibility   0.4597244    0.10458434               1.0000000</code></pre>

###### Table displaying the correlation among survey questions regarding government involvement: excessive government interference (too.much), opposition to government legislation enabling individuals to act in their own interest (not.pass.laws), preference for minimal government intervention in economic affairs (minimal.intervention), disapproval of government imposition on personal lifestyle choices (not.dictate), reluctance for the government to prioritize social objectives over individual liberties (indiv.freedom), advocacy for increased personal accountability among individuals (personal.responsibility).

<br>

-   For questions on personal behavior :

```{r}
cor(cbind(WTP$PN_1, WTP$PN_2, WTP$PN_3, 
  WTP$PN_4, WTP$PN_6, WTP$PN_7))
```

<pre><code class="hljs">##                  buy.local indiv.impact feel.better public.transport
## buy.local        1.0000000    0.4824823   0.4282149        0.4226534
## indiv.impact     0.4824823    1.0000000   0.6315015        0.4375971
## feel.better      0.4282149    0.6315015   1.0000000        0.4596711
## public.transport 0.4226534    0.4375971   0.4596711        1.0000000
## conserve.energy  0.4138090    0.4994126   0.5219712        0.5668642
## reduce.emissions 0.4584007    0.6542377   0.5894731        0.3947270
##                  conserve.energy reduce.emissions
## buy.local              0.4138090        0.4584007
## indiv.impact           0.4994126        0.6542377
## feel.better            0.5219712        0.5894731
## public.transport       0.5668642        0.3947270
## conserve.energy        1.0000000        0.4551294
## reduce.emissions       0.4551294        1.0000000</code></pre>

###### Correlation table for survey items on ‘personal responsibility for the environment’: I buy locally to reduce emissions (buy.local), I am obliged to take impact of daily activities on climate (individual.impact), I feel better when reducing emissions (feel.better), I prefer to use public transport (public.transport), I feel uncomfortable when consuming energy (conserve.energy), I try to reduce emissions as much as possible (reduce.emissions).

<br>

<br>

#### **Calculating Cronbach's Alpha:**

We use the **`aplha`** function from the **`psych`** package to calculate Cronbach's Alpha.

<br>

```{r}
psych::alpha(WTP[c("scepticism_2", 
  "scepticism_6", "scepticism_7")])$total$std.alpha

psych::alpha(WTP[c("cog_1", "cog_2", "cog_3", 
  "cog_4", "cog_5", "cog_6")])$total$std.alpha

psych::alpha(WTP[c("PN_1", "PN_2", "PN_3", 
  "PN_4", "PN_6", "PN_7")])$total$std.alpha
```
<pre><code class="hljs">## [1] 0.6876079<br>
## [1] 0.7102249 <br> ## [1] 0.8543827</code></pre>

<div style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;">
  <p style="font-weight: bold; margin-bottom: 5px;">Code Explanation:</p>
  <p>These R commands are utilized to compute Cronbach`s alpha for various sets of variables—namely scepticism, cognitive, and PN—within the dataframe WTP, employing the psych package.</p>
</div>


<br>

The coefficient values in question are high, suggesting that the indicators within each category measure the same underlying concept.

<br>

<br>

## **1.1 Comparing characteristics of people in DC group and TWPL group**

<br>

For each group, we compare them on the following basis:

-   gender (`sex`)

-   age (`age`)

-   number of children (`kids_nr`)

-   household net income per month in euros (`hhnetinc`)

-   membership in environmental organization (`member`)

-   highest educational attainment (`education`)

<br>

```{r}
variables <- list(quo(sex), quo(age),
  quo(kids_nr), quo(hhnetinc),
  quo(member), quo(education))

result_list <- list()

for (i in seq_along(variables)) {
  result <- WTP %>%
    group_by(abst_format, !!variables[[i]]) %>%
    summarize(n = n()) %>%
    mutate(freq = n / sum(n) * 100) %>%  # Calculate percentage
    select(-n) %>%
    spread(abst_format, freq) %>%
    rename(TWPL = 'ladder', DC = 'ref')  # Rename columns
    
  result_list[[i]] <- result
}

print(result_list)
```
<pre><code class="hljs">## [[1]]
## # A tibble: 2 × 3
##   sex     TWPL    DC
##   &lt;chr&gt;  &lt;dbl&gt; &lt;dbl&gt;
## 1 female  51.8  52.3
## 2 male    48.2  47.7
## 
## [[2]]
## # A tibble: 6 × 3
##   age      TWPL    DC
##   &lt;chr&gt;   &lt;dbl&gt; &lt;dbl&gt;
## 1 18 - 24  9.49  9.64
## 2 25 - 29  8.30  8.65
## 3 30 - 39 17.8  17.2 
## 4 40 - 49 22.3  22.6 
## 5 50 - 59 24.1  23.9 
## 6 60 - 69 18.0  18.1 
## 
## [[3]]
## # A tibble: 5 × 3
##   kids_nr                 TWPL     DC
##   &lt;chr&gt;                  &lt;dbl&gt;  &lt;dbl&gt;
## 1 four or more children  0.988  0.895
## 2 no children           64.6   65.7  
## 3 one child             20.4   17.6  
## 4 three children         2.96   3.48 
## 5 two children          11.1   12.3  
## 
## [[4]]
## # A tibble: 12 × 3
##    hhnetinc                   TWPL     DC
##    &lt;chr&gt;                     &lt;dbl&gt;  &lt;dbl&gt;
##  1 1100 bis unter 1500 Euro 14.2   13.2  
##  2 1500 bis unter 2000 Euro 15.0   14.6  
##  3 2000 bis unter 2600 Euro 11.5   14.8  
##  4 2600 bis unter 3200 Euro 10.7   10.7  
##  5 3200 bis unter 4000 Euro 11.1    8.15 
##  6 4000 bis unter 5000 Euro  5.14   4.97 
##  7 500 bis unter 1100 Euro  13.4   14.2  
##  8 5000 bis unter 6000 Euro  2.77   1.69 
##  9 6000 bis unter 7500 Euro  0.791  0.398
## 10 7500 und mehr             0.395  0.497
## 11 bis unter 500 Euro        2.96   4.17 
## 12 do not want to answer    12.1   12.5  
## 
## [[5]]
## # A tibble: 2 × 3
##   member  TWPL    DC
##   &lt;chr&gt;  &lt;dbl&gt; &lt;dbl&gt;
## 1 no     92.3  91.4 
## 2 yes     7.71  8.65
## 
## [[6]]
## # A tibble: 6 × 3
##   education  TWPL    DC
##       &lt;dbl&gt; &lt;dbl&gt; &lt;dbl&gt;
## 1         1  1.19  1.29
## 2         2  1.98  2.09
## 3         3 34.2  32.8 
## 4         4 26.3  26.9 
## 5         5  6.92  6.86
## 6         6 29.4  30.0</code></pre>

<div style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;">
  <p style="font-weight: bold; margin-bottom: 5px;">Code Explanation:</p>
  <p>
This R code iterates through categorical variables (sex, age, kids_nr, hhnetinc, member, education) in the WTP dataframe. For each variable, it groups data by abst_format, calculates frequencies (freq) based on counts (n), removes the count column, and spreads the results into wide-format tables. This analysis helps understand how each variable distributes across different abst_format categories.</p>
</div>


<br>

<br>

```{r}
WTP %>%
  group_by(abst_format) %>%
  summarise_at(c("climate", "gov_intervention", 
      "pro_environment"),
    funs(mean, sd, min, max)) %>%

  gather(index, value, 
    climate_mean:pro_environment_max) %>%
  spread(abst_format, value) %>%
  rename(TWPL = 'ladder', DC = 'ref') %>%
  kable(., format = "markdown", digits = 2)
```
<table class="table table-condensed">
<thead>
<tr class="header">
<th align="left">index</th>
<th align="right">TWPL</th>
<th align="right">DC</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">climate_max</td>
<td align="right">5.00</td>
<td align="right">5.00</td>
</tr>
<tr class="even">
<td align="left">climate_mean</td>
<td align="right">2.29</td>
<td align="right">2.37</td>
</tr>
<tr class="odd">
<td align="left">climate_min</td>
<td align="right">1.00</td>
<td align="right">1.00</td>
</tr>
<tr class="even">
<td align="left">climate_sd</td>
<td align="right">0.84</td>
<td align="right">0.85</td>
</tr>
<tr class="odd">
<td align="left">gov_intervention_max</td>
<td align="right">5.00</td>
<td align="right">5.00</td>
</tr>
<tr class="even">
<td align="left">gov_intervention_mean</td>
<td align="right">3.15</td>
<td align="right">3.19</td>
</tr>
<tr class="odd">
<td align="left">gov_intervention_min</td>
<td align="right">1.00</td>
<td align="right">1.00</td>
</tr>
<tr class="even">
<td align="left">gov_intervention_sd</td>
<td align="right">0.70</td>
<td align="right">0.66</td>
</tr>
<tr class="odd">
<td align="left">pro_environment_max</td>
<td align="right">5.00</td>
<td align="right">5.00</td>
</tr>
<tr class="even">
<td align="left">pro_environment_mean</td>
<td align="right">3.03</td>
<td align="right">3.01</td>
</tr>
<tr class="odd">
<td align="left">pro_environment_min</td>
<td align="right">1.00</td>
<td align="right">1.00</td>
</tr>
<tr class="even">
<td align="left">pro_environment_sd</td>
<td align="right">0.79</td>
<td align="right">0.82</td>
</tr>
</tbody>
</table>


<br>

### **Observation:**

The two groups are quite similar in attitudes. We can be reasonably confident that any differences in survey responses is due to the question format rather than differences in attitudes or demographics.

------------------------------------------------------------------------

<br>

<br>

# **2. Comparing Willingness to pay across methods and individual characteristics**

<br>

We begin this analysis by summarizing the distribution of WTP within each question format. We use the following column charts for this analysis.

<br>

#### **For minimum willingness to pay:**

![](/images/postimages/p4_2.png)

<br>

**Observation:** We can observe that a majority of people have the price of **48 euros** as the price they would definitely vote in favor of.

<br>

#### **For the maximum willingness to pay:**
![](/images/postimages/p4_3.png)


<br>

**Observation:** Despite the broad range in the maximum price at which individuals would cease to support, it is evident that **108 euros** is the price most frequently selected by the majority.

<br>

```{r}
df.plmax <- WTP %>%
  select(WTP_plmax_euro) %>%
  na.omit() %>%
  group_by(WTP_plmax_euro) %>%
  summarize(n = n()) %>%
  mutate(WTP_plmax_euro = factor(WTP_plmax_euro, 
    levels = wtp_euro_levels))

ggplot(df.plmax, aes(WTP_plmax_euro, n)) +
  geom_bar(stat = "identity", position = "identity") + 
  xlab("Maximum WTP (euros)") + 
  ylab("Frequency") +
  theme_bw()
```

::: {style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;"}
<p style="font-weight: bold; margin-bottom: 5px;">

**Code Explanation:**

</p>

<p>This R code uses **`ggplot2`** to create a bar plot showing how maximum willingness-to-pay **(`WTP_plmax_euro`)** values are distributed across predefined euro levels in the `WTP` data-set. It calculates frequencies for each **`WTP_plmax_euro`** value after grouping and converts it into a factor with specified levels. The plot displays frequencies on the y-axis and **`WTP_plmax_euro`** values on the x-axis, with a black and white theme for clarity.</p>
:::

<br>

<br>

### **Calculating average WTP for each individual**

<br>

```{r}
WTP <- WTP %>%
  rowwise() %>%
  mutate(., WTP_average = rowMeans(cbind(
    WTP_plmin_euro, WTP_plmax_euro))) %>%
  ungroup()
```

::: {style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;"}
<p style="font-weight: bold; margin-bottom: 5px;">

**Code Explanation:**

</p>

<p>This code calculates the row-wise average of two existing columns (**`WTP_plmin_euro`** and **`WTP_plmax_euro`**) in the `WTP` data frame and stores the result in a new column named **`WTP_average`**.</p>
:::

<br>

```{r}
mean(WTP$WTP_average, na.rm = TRUE)
median(WTP$WTP_average, na.rm = TRUE)
```
<pre><code class="hljs">## [1] 268.5345 <br>
## [2] 132</code></pre>

<br>

The above given are the values of mean and median of this average value variable created by us.

<br>

### **Calculating correlation coefficients**

<br>

```{r}
WTP %>%
  mutate(gender = 
    as.numeric(ifelse(sex == "female", 0, 1))) %>%
  select(WTP_average, education, gender,
    climate, gov_intervention, pro_environment) %>%
  cor(., use = "pairwise.complete.obs") -> M

M[, "WTP_average"]
```

<br>

::: {style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;"}
<p style="font-weight: bold; margin-bottom: 5px;">

**Code Explanation:**

</p>

<p>This code calculates the correlation between **`WTP_average`** (presumably a derived column from a previous operation) and several other selected variables (*`education`*, *`gender`*, *`climate`*, *`gov_intervention`*, *`pro_environment`*) in the `WTP` data frame. It first converts the `sex` column to numeric values in the `gender` column and then computes the correlation matrix using complete observations, storing the result in `M`. Finally, it extracts and likely inspects the correlations involving `WTP_average`.</p>
:::

<br>

**Observation:**

-   **`WTP_average` with `education`**:

    -   Correlation coefficient of 0.13817368.

    -   Indicates a weak positive correlation between `WTP_average` and `education`

-   **`WTP_average` with `gender`**:

    -   Correlation coefficient of 0.03694972.

    -   Indicates a very weak positive correlation between `WTP_average` and `gender`

-   **`WTP_average` with `climate`**:

    -   Correlation coefficient of -0.14462072.

    -   Indicates a moderate negative correlation between `WTP_average` and `climate`. This means as concerns or perceptions related to climate increase, the average willingness to pay (`WTP_average`) tends to decrease.

-   **`WTP_average` with `gov_intervention`**:

    -   Correlation coefficient of -0.18845205.

    -   Indicates a moderate negative correlation between `WTP_average` and `gov_intervention`. This suggests that as support or favorability towards government intervention increases, the average willingness to pay (`WTP_average`) tends to decrease.

-   **`WTP_average` with `pro_environment`**:

    -   Correlation coefficient of 0.18750331.

    -   Indicates a moderate positive correlation between `WTP_average` and `pro_environment`.

<br>

<br>

## **2.1 Comparing question formats**

<br>

### **For individuals who answered DC(*dichotomous choice)* questions**

Each individual was given an amount and had to decide ‘yes’, ‘no’, or ‘no vote/abstain from deciding'.

<br>

<table class="table table-condensed">
<thead>
<tr class="header">
<th align="right">costs</th>
<th align="right">Abstain</th>
<th align="right">No</th>
<th align="right">Yes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">48</td>
<td align="right">12</td>
<td align="right">21</td>
<td align="right">32</td>
</tr>
<tr class="even">
<td align="right">72</td>
<td align="right">11</td>
<td align="right">30</td>
<td align="right">40</td>
</tr>
<tr class="odd">
<td align="right">84</td>
<td align="right">12</td>
<td align="right">24</td>
<td align="right">45</td>
</tr>
<tr class="even">
<td align="right">108</td>
<td align="right">7</td>
<td align="right">35</td>
<td align="right">31</td>
</tr>
<tr class="odd">
<td align="right">156</td>
<td align="right">13</td>
<td align="right">31</td>
<td align="right">40</td>
</tr>
<tr class="even">
<td align="right">192</td>
<td align="right">11</td>
<td align="right">25</td>
<td align="right">25</td>
</tr>
<tr class="odd">
<td align="right">252</td>
<td align="right">9</td>
<td align="right">32</td>
<td align="right">28</td>
</tr>
<tr class="even">
<td align="right">324</td>
<td align="right">16</td>
<td align="right">41</td>
<td align="right">27</td>
</tr>
<tr class="odd">
<td align="right">432</td>
<td align="right">11</td>
<td align="right">35</td>
<td align="right">29</td>
</tr>
<tr class="even">
<td align="right">540</td>
<td align="right">9</td>
<td align="right">31</td>
<td align="right">22</td>
</tr>
<tr class="odd">
<td align="right">720</td>
<td align="right">12</td>
<td align="right">39</td>
<td align="right">13</td>
</tr>
<tr class="even">
<td align="right">960</td>
<td align="right">14</td>
<td align="right">28</td>
<td align="right">15</td>
</tr>
<tr class="odd">
<td align="right">1200</td>
<td align="right">11</td>
<td align="right">42</td>
<td align="right">21</td>
</tr>
<tr class="even">
<td align="right">1440</td>
<td align="right">19</td>
<td align="right">42</td>
<td align="right">15</td>
</tr>
</tbody>
</table>

This table illustrates the distribution of individuals according to their voting choices and the associated costs.

<br>

```{r}
WTP_DC <- WTP %>%
  group_by(costs, DC_ref_outcome) %>%
  summarize(n = n()) %>%
  na.omit() %>%
  mutate_at("DC_ref_outcome", 
    funs(recode(., 
      "do not support referendum and no pay" = "No",
      "support referendum and pay" = "Yes",
      "would not vote" = "Abstain"))) %>%
  spread(DC_ref_outcome, n)

kable(WTP_DC, format = "markdown", digits = 2)
```

::: {style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;"}
<p style="font-weight: bold; margin-bottom: 5px;">

**Code Explanation:**

</p>

<p>The code groups the `WTP` data by `costs` and `DC_ref_outcome`, counts the number of observations for each group, removes any rows with missing values, and then recategorizes specific values in `DC_ref_outcome` before spreading the summarized data into a table format suitable for Markdown output.</p>
:::

<br>

<br>

### **Modifying the table to include proportions**

<br>

<table class="table table-condensed">
<thead>
<tr class="header">
<th align="right">costs</th>
<th align="right">Abstain</th>
<th align="right">No</th>
<th align="right">Yes</th>
<th align="right">total</th>
<th align="right">prop_no</th>
<th align="right">prop_yes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">48</td>
<td align="right">12</td>
<td align="right">21</td>
<td align="right">32</td>
<td align="right">65</td>
<td align="right">0.51</td>
<td align="right">0.49</td>
</tr>
<tr class="even">
<td align="right">72</td>
<td align="right">11</td>
<td align="right">30</td>
<td align="right">40</td>
<td align="right">81</td>
<td align="right">0.51</td>
<td align="right">0.49</td>
</tr>
<tr class="odd">
<td align="right">84</td>
<td align="right">12</td>
<td align="right">24</td>
<td align="right">45</td>
<td align="right">81</td>
<td align="right">0.44</td>
<td align="right">0.56</td>
</tr>
<tr class="even">
<td align="right">108</td>
<td align="right">7</td>
<td align="right">35</td>
<td align="right">31</td>
<td align="right">73</td>
<td align="right">0.58</td>
<td align="right">0.42</td>
</tr>
<tr class="odd">
<td align="right">156</td>
<td align="right">13</td>
<td align="right">31</td>
<td align="right">40</td>
<td align="right">84</td>
<td align="right">0.52</td>
<td align="right">0.48</td>
</tr>
<tr class="even">
<td align="right">192</td>
<td align="right">11</td>
<td align="right">25</td>
<td align="right">25</td>
<td align="right">61</td>
<td align="right">0.59</td>
<td align="right">0.41</td>
</tr>
<tr class="odd">
<td align="right">252</td>
<td align="right">9</td>
<td align="right">32</td>
<td align="right">28</td>
<td align="right">69</td>
<td align="right">0.59</td>
<td align="right">0.41</td>
</tr>
<tr class="even">
<td align="right">324</td>
<td align="right">16</td>
<td align="right">41</td>
<td align="right">27</td>
<td align="right">84</td>
<td align="right">0.68</td>
<td align="right">0.32</td>
</tr>
<tr class="odd">
<td align="right">432</td>
<td align="right">11</td>
<td align="right">35</td>
<td align="right">29</td>
<td align="right">75</td>
<td align="right">0.61</td>
<td align="right">0.39</td>
</tr>
<tr class="even">
<td align="right">540</td>
<td align="right">9</td>
<td align="right">31</td>
<td align="right">22</td>
<td align="right">62</td>
<td align="right">0.65</td>
<td align="right">0.35</td>
</tr>
<tr class="odd">
<td align="right">720</td>
<td align="right">12</td>
<td align="right">39</td>
<td align="right">13</td>
<td align="right">64</td>
<td align="right">0.80</td>
<td align="right">0.20</td>
</tr>
<tr class="even">
<td align="right">960</td>
<td align="right">14</td>
<td align="right">28</td>
<td align="right">15</td>
<td align="right">57</td>
<td align="right">0.74</td>
<td align="right">0.26</td>
</tr>
<tr class="odd">
<td align="right">1200</td>
<td align="right">11</td>
<td align="right">42</td>
<td align="right">21</td>
<td align="right">74</td>
<td align="right">0.72</td>
<td align="right">0.28</td>
</tr>
<tr class="even">
<td align="right">1440</td>
<td align="right">19</td>
<td align="right">42</td>
<td align="right">15</td>
<td align="right">76</td>
<td align="right">0.80</td>
<td align="right">0.20</td>
</tr>
</tbody>
</table>

At this point, in addition to the numerical counts, we can observe the proportion of votes allocated to each option.

<br>

### **Visualizing WTP on a Demand Curve**

<br>

![](/images/postimages/p4_4.png)


<br>

The demand curve typically exhibits a downward slope, indicating that the proportion of individuals voting 'yes' tends to diminish as the monetary amount rises.

<br>

<br>

### **Doing the analysis excluding individuals who chose 'abstain**

<br>

<table class="table table-condensed">
<colgroup>
<col width="7%">
<col width="10%">
<col width="3%">
<col width="5%">
<col width="7%">
<col width="10%">
<col width="11%">
<col width="11%">
<col width="14%">
<col width="15%">
</colgroup>
<thead>
<tr class="header">
<th align="right">costs</th>
<th align="right">Abstain</th>
<th align="right">No</th>
<th align="right">Yes</th>
<th align="right">total</th>
<th align="right">prop_no</th>
<th align="right">prop_yes</th>
<th align="right">total_ex</th>
<th align="right">prop_no_ex</th>
<th align="right">prop_yes_ex</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">48</td>
<td align="right">12</td>
<td align="right">21</td>
<td align="right">32</td>
<td align="right">65</td>
<td align="right">0.51</td>
<td align="right">0.49</td>
<td align="right">53</td>
<td align="right">0.40</td>
<td align="right">0.60</td>
</tr>
<tr class="even">
<td align="right">72</td>
<td align="right">11</td>
<td align="right">30</td>
<td align="right">40</td>
<td align="right">81</td>
<td align="right">0.51</td>
<td align="right">0.49</td>
<td align="right">70</td>
<td align="right">0.43</td>
<td align="right">0.57</td>
</tr>
<tr class="odd">
<td align="right">84</td>
<td align="right">12</td>
<td align="right">24</td>
<td align="right">45</td>
<td align="right">81</td>
<td align="right">0.44</td>
<td align="right">0.56</td>
<td align="right">69</td>
<td align="right">0.35</td>
<td align="right">0.65</td>
</tr>
<tr class="even">
<td align="right">108</td>
<td align="right">7</td>
<td align="right">35</td>
<td align="right">31</td>
<td align="right">73</td>
<td align="right">0.58</td>
<td align="right">0.42</td>
<td align="right">66</td>
<td align="right">0.53</td>
<td align="right">0.47</td>
</tr>
<tr class="odd">
<td align="right">156</td>
<td align="right">13</td>
<td align="right">31</td>
<td align="right">40</td>
<td align="right">84</td>
<td align="right">0.52</td>
<td align="right">0.48</td>
<td align="right">71</td>
<td align="right">0.44</td>
<td align="right">0.56</td>
</tr>
<tr class="even">
<td align="right">192</td>
<td align="right">11</td>
<td align="right">25</td>
<td align="right">25</td>
<td align="right">61</td>
<td align="right">0.59</td>
<td align="right">0.41</td>
<td align="right">50</td>
<td align="right">0.50</td>
<td align="right">0.50</td>
</tr>
<tr class="odd">
<td align="right">252</td>
<td align="right">9</td>
<td align="right">32</td>
<td align="right">28</td>
<td align="right">69</td>
<td align="right">0.59</td>
<td align="right">0.41</td>
<td align="right">60</td>
<td align="right">0.53</td>
<td align="right">0.47</td>
</tr>
<tr class="even">
<td align="right">324</td>
<td align="right">16</td>
<td align="right">41</td>
<td align="right">27</td>
<td align="right">84</td>
<td align="right">0.68</td>
<td align="right">0.32</td>
<td align="right">68</td>
<td align="right">0.60</td>
<td align="right">0.40</td>
</tr>
<tr class="odd">
<td align="right">432</td>
<td align="right">11</td>
<td align="right">35</td>
<td align="right">29</td>
<td align="right">75</td>
<td align="right">0.61</td>
<td align="right">0.39</td>
<td align="right">64</td>
<td align="right">0.55</td>
<td align="right">0.45</td>
</tr>
<tr class="even">
<td align="right">540</td>
<td align="right">9</td>
<td align="right">31</td>
<td align="right">22</td>
<td align="right">62</td>
<td align="right">0.65</td>
<td align="right">0.35</td>
<td align="right">53</td>
<td align="right">0.58</td>
<td align="right">0.42</td>
</tr>
<tr class="odd">
<td align="right">720</td>
<td align="right">12</td>
<td align="right">39</td>
<td align="right">13</td>
<td align="right">64</td>
<td align="right">0.80</td>
<td align="right">0.20</td>
<td align="right">52</td>
<td align="right">0.75</td>
<td align="right">0.25</td>
</tr>
<tr class="even">
<td align="right">960</td>
<td align="right">14</td>
<td align="right">28</td>
<td align="right">15</td>
<td align="right">57</td>
<td align="right">0.74</td>
<td align="right">0.26</td>
<td align="right">43</td>
<td align="right">0.65</td>
<td align="right">0.35</td>
</tr>
<tr class="odd">
<td align="right">1200</td>
<td align="right">11</td>
<td align="right">42</td>
<td align="right">21</td>
<td align="right">74</td>
<td align="right">0.72</td>
<td align="right">0.28</td>
<td align="right">63</td>
<td align="right">0.67</td>
<td align="right">0.33</td>
</tr>
<tr class="even">
<td align="right">1440</td>
<td align="right">19</td>
<td align="right">42</td>
<td align="right">15</td>
<td align="right">76</td>
<td align="right">0.80</td>
<td align="right">0.20</td>
<td align="right">57</td>
<td align="right">0.74</td>
<td align="right">0.26</td>
</tr>
</tbody>
</table>

This table establishes new proportions for individuals who have participated in voting, excluding those who have chosen to abstain.

<br>

![](/images/postimages/p4_5.png)

<br>

<br>

## **Comparing DC and TWPL question formats**

For the DC format, willingness to pay is recorded in the `costs` variable, so we select all observations where the `DC_ref_outcome` variable indicates the individual voted ‘yes’ and drop any missing observations. For the TWPL format we use the `WTP_average` variable that we created.

<br>

#### **Difference in means, standard deviations and number of observations**

<br>

```{r}
DC_WTP <- WTP %>% subset(
  DC_ref_outcome == "support referendum and pay") %>%
  select(costs) %>%
  filter(!is.na(costs)) %>%
  as.matrix()

TWPL_WTP <- WTP %>%
  select(WTP_average) %>%
  filter(!is.na(WTP_average)) %>%
  as.matrix()
```

::: {style="border: 1px solid black; padding: 10px; background-color: #f0f0f0; margin-bottom: 10px;"}
<p style="font-weight: bold; margin-bottom: 5px;">

**Code Explanation:**

</p>

<p>The code first extracts a subset **`DC_WTP`** from the `WTP` data frame, containing only the **`costs`** values where **`DC_ref_outcome`** is "support referendum and pay". It ensures no missing values are included. Then, it creates **`TWPL_WTP`** by extracting non-missing `WTP_average` values from `WTP` and converts both subsets into matrices for further analysis.</p>
:::

<br>

<br>

```{r}
cat(sprintf("DC Format -> mean: %.1f, 
  standard deviation %.1f, count %d\n",
  mean(DC_WTP), sd(DC_WTP), length((DC_WTP))))
```

<br>

```{r}
cat(sprintf("TWPL Format -> mean: %.1f, 
  standard deviation %.1f, count %d\n", 
  mean(TWPL_WTP), sd(TWPL_WTP),
  length((TWPL_WTP))))
```
<pre><code class="hljs">## DC Format -&gt; mean: 348.2, 
##   standard deviation 378.6, count 383</code></pre>

<pre><code class="hljs">## TWPL Format -&gt; mean: 268.5, 
##   standard deviation 287.7, count 348</code></pre>

<br>

These statistics elucidate the variation in WTP values between the two survey formats. The mean values represent the average amount respondents are willing to pay under each format, while the standard deviations measure the variability in WTP responses. The counts reflect the number of valid responses used for each calculation, ensuring the reliability and representativeness of the findings for each survey method. Comparing these metrics aids in determining whether different survey formats elicit significantly different WTP responses from respondents.

<br>

### **95% Confidence Intervals**

A confidence interval, in statistics, refers to the probability that a population parameter will fall between a set of values for a certain proportion of times. Thus, if a point estimate is generated from a statistical model of 10.00 with a 95% confidence interval of 9.50 to 10.50, it means one is 95% confident that the true value falls within that range.

<br>

A 95% confidence interval is a range of possible values within which the true value might lie. It is estimated from the mean and standard deviation of the data. 

> As the name suggests, confidence intervals tell us how much confidence we can place in our estimates, or in other words, how precisely the sample mean is estimated. The confidence interval gives us a margin of error for our estimate of the true value.

<br>

```{r}
t.test(DC_WTP, TWPL_WTP, conf.level = 0.05)$conf.int
```
<pre><code class="hljs">## [1] 78.10141 81.20560
## attr(,"conf.level")
## [1] 0.05</code></pre>

<br>

The ‘width’ of the confidence interval is **1.55**, so the confidence interval is [79.65 – 1.55, 79.65 + 1.55], which is [78.10, 81.21].

The substantial difference in means (approximately 80 euros) is precisely estimated, indicating that the observed disparity is unlikely due to chance. Therefore, **willingness to pay (WTP) is higher under the dichotomous choice (DC) format compared to the two-way payment ladder (TWPL) format.**

<br>

------------------------------------------------------------------------

<br>

<br>

# **3. Meeting the Paris Agreement Targets**

<br>

## **Paris Agreement**

The Paris Agreement is a **legally binding international treaty on climate change**. It was adopted by 196 Parties at the UN Climate Change Conference (COP21) in Paris, France, on 12 December 2015. It entered into force on 4 November 2016.\
\
Its overarching goal is to hold “the increase in the global average temperature to well below 2°C above pre-industrial levels” and pursue efforts “to limit the temperature increase to 1.5°C above pre-industrial levels.”

<br>

## **3.1 Investment needed to achieve Paris Agreement Targets**

<br>

For analyzing the investment needed to achieve Paris Agreement Targets, we will use *Climate Investment Potential (\$ billion)* data given in the report [**Climate Investment Opportunities in Emerging Markets: An IFC Analysis**](https://www.ifc.org/content/dam/ifc/doc/mgrt/3503-ifc-climate-investment-opportunity-report-dec-final.pdf)

<br>

```{r}
invst <- read_excel("climate investment oppurtunites-ifc-climate.xlsx")
```

<br>

The data looks something like this:
<pre><code class="hljs">## # A tibble: 6 × 12
##   `(in billions)`   Wind Solar Biomass `Small Hydro` Geothermal `All Renewables`
##   &lt;chr&gt;            &lt;dbl&gt; &lt;dbl&gt;   &lt;dbl&gt;         &lt;dbl&gt;      &lt;dbl&gt;            &lt;dbl&gt;
## 1 East Asia Pacif…   231   537      48            34         16              866
## 2 Latin America a…   118    44      45            11         14              232
## 3 South Asia         111   211      16             0          0              338
## 4 Europe and Cent…    51    39       6             7          6              109
## 5 Sub-Saharan Afr…    27    63       3             3         27              123
## 6 Middle East and…    50    46       0             1          0               97
## # ℹ 5 more variables: `Electric Transmission and Distribution` &lt;dbl&gt;,
## #   `Industrial Energy` &lt;dbl&gt;, Buildings &lt;dbl&gt;, Transport &lt;dbl&gt;, Waste &lt;dbl&gt;</code></pre>

<br>

This data contains the various climate investment opportunities in different sectors identified by IFC in various regions of the world that are necessary to achieve the Paris Agreement targets by 2030.

**Note:** This study was done in 2015-2016.

> The estimates in this report are based on the 21 NDCs submitted to the United Nations Framework Convention on Climate Change by IFC’s countries of focus, as well as the national plans, policies, and targets that underpin them. IFC assessed the national climate change commitments and other policies in 21 countries that represent 48 percent of global greenhousegas (GHG) emissions

<br>
![](/images/postimages/p4_6.png)

As we can clearly see, that the East Asia Pacific region needs the most investment. This large proportion of this region in this pie chart, is due to the inclusion of China in this region. Currently, China and India are the two most important countries in the fight against climate change.

The achievement of these targets depend alot on the pace at which India and China achieve the target.

<br>

### **Investment needed in different sectors**

This report, majorly talks about the below given sectors where investments are needed:

-   Wind

-   Solar

-   Biomass

-   Small Hydro

-   Geothermal

-   All Renewables

-   Electric Transmission and Distribution

-   Industrial Energy

-   Buildings Transport Waste

<br>

![](/images/postimages/p4_7.png)

<br>

![](/images/postimages/p4_8.png)

<br>

![](/images/postimages/p4_9.png)

<br>

![](/images/postimages/p4_10.png)
<br>

These graphs enable us to analyze which regions require greater investment in specific sectors.

<br>

------------------------------------------------------------------------

<br>

### **Comparison between different Sectors**

<br>

![](/images/postimages/p4_11.png)

<br>

The above graph shows, that the most investment is needed in the Building/Infrastructure sector.

> China, Indonesia, the Philippines, and Vietnam have a climate-smart investment potential of **\$16 trillion**, most of which is concentrated in the construction of new green buildings.

Opportunities for new green buildings and energy-efficient retrofits for millions of existing buildings are massive.

### <br>

### **Observation:**

According to the graphs presented, IFC has projected that approximately **\$23 trillion** will be necessary between the years 2016 and 2030 to achieve the objectives set forth by the Paris Agreement. A significant portion of this investment needs to be allocated to the combined regions of South and East Pacific Asia.

<br>

<br>

## **3.2 Climate Tax needed to achieve the targets**

As previously discussed, achieving the targets set by the Paris Agreement will require an investment of \$23 trillion over the next 15 years.

To generate this substantial amount of funding, one potential strategy the government might consider is **imposing a climate tax on the population.**

<br>

```{r}
median(DC_WTP)
```

<pre><code class="hljs">## [1] 192</code></pre>

<br>

Given the global adult population of **5.16 billion**, implementing a climate tax equivalent to the median value of **`DC_WTP`** would generate approximately 1 trillion euros annually, which is roughly equal to **1 trillion USD per year.**

Over a span of 15 years, this would result in the accumulation of **15 trillion USD**, which falls short of the required amount.

<br>

#### **Challenge:**

-   Implementing a climate tax amounting to 192 euros presents a significant challenge in developing and impoverished nations such as India and those in Southeast Asia, regions which collectively account for a substantial portion of the global population.

<br>

Given that the survey analyzed in this report was conducted in Europe, if such a tax were to be imposed solely within Europe, the **885 million adult population (including Russia)** would contribute approximately \$169 billion annually. This would result in a total collection of only **\$2.5 trillion** over a span of 15 years.

<br>

## **Conclusion:**

Our study of willingness to pay (WTP) and the financial needs for the Paris Agreement targets highlights challenges and opportunities.

-   The link between WTP and factors like education, gender, and environmental attitudes shows the importance of public awareness. Governments must educate citizens on the urgency of climate action and the benefits of sustainable investments.

-   To address the \$23 trillion funding gap by 2030, innovative financing beyond traditional taxes is needed. Governments can boost WTP through policies like tax credits for renewable energy or subsidies for green technologies. Engaging stakeholders across sectors and regions is crucial for global climate resilience and sustainable development.

Achieving these targets requires global collaboration. International partnerships, technology transfers, and capacity-building are key to helping developing countries transition to low-carbon economies. By creating a supportive policy environment and ensuring inclusivity, we can secure a sustainable future.

In conclusion, financial challenges can be overcome with political will and collective action. By raising public awareness, using innovative financing, and fostering international cooperation, we can move towards a climate-resilient world that meets the Paris Agreement goals.

<br>

------------------------------------------------------------------------

<br>

# References

1.  Population Data(world): <https://data.worldbank.org/indicator/SP.POP.1564.TO>

2.  Population (Europe): <https://www.worldometers.info/world-population/europe-population>

3.  Climate Investment Opportunities in Emerging Markets: An IFC Analysis

    <https://www.ifc.org/content/dam/ifc/doc/mgrt/3503-ifc-climate-investment-opportunity-report-dec-final.pdf>

    **Page VI**

4.  Paris Agreement: <https://www.un.org/en/climatechange/paris-agreement>
