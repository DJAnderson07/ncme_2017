---
title       : Visualizing Effect Sizes Across the Full Distribution
subtitle    : Daniel Anderson,  Joseph Stevens,  Joseph Nese 
author      : Univeristy of Oregon
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : hemisu-light      # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
--- .quote
<style>
em {
  font-style: italic
}
</style>

<style>
strong {
  font-weight: bold;
}
</style>



## Background
* Effect sizes generally defined by standardized mean differences
    + Cohen's *d*
    + Hedges' *g*
* Particularly in non-experimental settings, interest may lie at other locations of the scale
    + Achievement gaps at proficiency cut scores on statewide tests
* Depending on the shape of each distribution, magnitude of group differences may depend upon scale location

---- &twocol
## Cohen's *d* & Hedges' *g*

<br>
<br>
<br>
<br>

*** =left

$$d = \frac{\bar{X}_{foc} - \bar{X}_{ref}}
        {\sqrt{\frac{(n_{foc} - 1)Var_{foc} + (n_{ref} - 1)Var_{ref}}
                  {n_{foc} + n_{ref} - 2}}}$$

*** =right

$$g = d\Big(1 - \frac{3}{4(n_{foc} + n_{ref}) - 9}\Big)$$

---- &twocol
## Percentage above the cut effect sizes

*** =left
# Percentage Above the Cut
$$d^{pac} = PAC_{ref} - PAC_{foc}$$
* Highly dependent on scale location

*** =right
# Transformed Percentage Above the Cut
$$d^{tpac} = \Phi^{-1}(PAC_{ref}) - \Phi^{-1}(PAC_{foc})$$
* Assumes both distributions are normally distributed with equal variance

---- &twocol
## Probability-Probability Plots



*** =left

![plot of chunk ecdf1](assets/fig/ecdf1-1.png)

*** =right

![plot of chunk pp_plot1](assets/fig/pp_plot1-1.png)

----
## Area Under the PP Curve

![plot of chunk auc](assets/fig/auc-1.png)

----
## Putting AUC in SD units 
<span style="color:gray">Ho and colleagues</span>
<br>

$$V = \sqrt{2}\Phi^{-1}(AUC)$$ 

* Scale invariant
* Assumes respective normality
	+ Normal with respect to each other under a shared transformation 



<br>

* *AUC* and *V* make fewer assumptions about the data, but are nonetheless summary measures.
* May miss nuances in the data that can be picked up by visualizations - particularly if the magnitude of the effect depends on scale location.

---- .segue
# Implementation in `esvis`


---- &twocol
## R package actively in development

*** =left

* Install using the *devtools* package


```r
install.packages("devtools")
library(devtools) 
install_github("DJAnderson07/esvis")
```

* Release to CRAN planned for summer
* Has many useful features currently

<br>

See current development at 

https://github.com/DJAnderson07/esvis

*** =right
![esvis](./assets/img/esvis.png)

----
## Example data

I have stored a dataset in an object called `d`. Below are the first six rows of these data.


```
##         sid cohort     sped  ethnicity frl     ell season reading math
## 2873 332347      1 Non-Sped   Hispanic FRL  Active Spring     167  192
## 162  400047      1 Non-Sped Native Am. FRL Non-ELL Spring     191  191
## 355  400047      1 Non-Sped Native Am. FRL Non-ELL   Fall     183  182
## 387  400047      1 Non-Sped Native Am. FRL Non-ELL Winter     178  179
## 230  400277      1 Non-Sped Native Am. FRL Non-ELL Winter     199  197
## 648  400277      1 Non-Sped Native Am. FRL Non-ELL   Fall     203  196
```



----
## Standard argument structure
* All functions in *esvis* take a common argument structure, as follows


```r
fun_name(outcome ~ group, data, additional_optional_args)
```

---- &twocol
## PP Plots

*** =left

* Examine math differences by free or reduced lunch status


```r
pp_plot(math ~ frl, d)
```

* Notice shading by default when only two groups are compared.
* *AUC* and *V* annotated to the plot, by default
* Plot is fully customizable with calls to base plotting functions (e.g., `main`, `col`, etc.)

*** =right

![plot of chunk frl_pp_plot_eval](assets/fig/frl_pp_plot_eval-1.png)

----
## More than one group?
* Highest performing group selected by default


```r
pp_plot(reading ~ ethnicity, d)
```

![plot of chunk pp_plot_eth](assets/fig/pp_plot_eth-1.png)

---- &twocol
## Investigating ELL differences
* Three groups: Non, Active, Monitor
* Same syntax for estimates

*** =left
Default output


```r
coh_d(reading ~ ell, d)
```

```
##   ref_group foc_group    estimate
## 1   Monitor   Non-ELL  0.05421767
## 2   Monitor    Active  0.70109139
## 3   Non-ELL    Active  0.95679846
## 4    Active   Non-ELL -0.95679846
## 5    Active   Monitor -0.70109139
## 6   Non-ELL   Monitor -0.05421767
```

*** =right
Or choose a reference group


```r
auc(reading ~ ell, d, 
		ref_group = "Non-ELL")
```

```
##   ref_group foc_group  estimate
## 3   Non-ELL    Active 0.7552992
## 6   Non-ELL   Monitor 0.4965789
```

----
# Visualization provides more nuance


```r
pp_plot(reading ~ ell, d, ref_group = "Non-ELL")
```

![plot of chunk pp_plot_ell](assets/fig/pp_plot_ell-1.png)

----
## ECDFs
* Produced equivalently


```r
ecdf_plot(reading ~ ell, d)
```

![plot of chunk ecdf_ell1](assets/fig/ecdf_ell1-1.png)

----
## Cut-point?


```r
ecdf_plot(reading ~ ell, d, ref_cut = c(190, 200, 207))
```

![plot of chunk ecdf_ell2](assets/fig/ecdf_ell2-1.png)

----
## Add horizontal reference lines


```r
ecdf_plot(reading ~ ell, d, ref_cut = c(190, 200, 207), hor_ref = TRUE)
```

![plot of chunk ecdf_ell3](assets/fig/ecdf_ell3-1.png)

----
## Binned ES Plot

* Split each distribution into arbitrary (even) quantile bins
* Calculate mean difference within each bin
* Divide by overall pooled standard deviation

$$d_{[i]} = \frac{\bar{X}_{foc_{[i]}} - \bar{X}_{ref_{[i]}}}
        {\sqrt{\frac{(n_{foc} - 1)Var_{foc} + (n_{ref} - 1)Var_{ref}}
                  {n_{foc} + n_{ref} - 2}}}$$

* In this case, essentially equivalent to Cohen's *d*, except that there are multiple mean differences (one for each bin)

----
## Ethnicity differences


```r
binned_plot(math ~ ethnicity, d)
```

![plot of chunk binned_es_plot_eth1](assets/fig/binned_es_plot_eth1-1.png)

----
## Change binning
Quintile binning


```r
binned_plot(math ~ ethnicity, d, qtiles = seq(0, 1, .2))
```

![plot of chunk binned_es_plot_eth2](assets/fig/binned_es_plot_eth2-1.png)

----
## Change reference group


```r
binned_plot(math ~ ethnicity, d, ref_group = "Black", qtiles = seq(0, 1, .2))
```

![plot of chunk binned_es_plot_eth3](assets/fig/binned_es_plot_eth3-1.png)

---- &twocol bg:#363636
## Theme dark

*** =left


```r
pp_plot(math ~ ethnicity, d, 
	theme = "dark")
```

![plot of chunk theme_standard](assets/fig/theme_standard-1.png)

*** =right


```r
binned_plot(math ~ ethnicity, d, 
	theme = "dark")
```

![plot of chunk theme_dark](assets/fig/theme_dark-1.png)

----
# Estimation 
`esvis` will also calculate a number of effect sizes using the same argument structure, including:
* Cohen's *d*
* Hedges' *g*
* *AUC*
* *V*
* *PAC* with **any set** of cut scores
* *TPAC* with **one** cut score (currently)

By default, effect sizes are produced for all possible pairwise comparisons, but reference groups can be selected as well.

----
## Summary and future developments
* Visualizing group differences across the full scale, or at particular points of the scale, is important for interpretation and communication.
* `esvis` provides a simple interface to produce powerful visualizations

# Future development
* Interactions with `:`
* Interactions via panel plotting
* Others?

---- .segue
# Thanks!

Slides available at: https://djanderson07.github.io/ncme_2017/
