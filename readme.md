
# ggcor

The `ggcor` package can be used to visualize simply and directly a
correlation matrix based on ‘ggplot2’. It provides a solution for
reordering the correlation matrix, displaying the different significance
level on the plot and other details. The most important parts, It also
provides a graphical display of any correlation analysis and their
combination (such as Mantel test, Partial correlation analysis, and so
on).

## Installation

Now `ggcor` is not on cran, You can install the development version of
ggcor from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("houyunhuang/ggcor")
```

## Draw correlation plot quickly

This is a basic example which shows you how to draw correlation plot
quickly:

``` r
library(ggplot2)
library(ggcor)
#> Registered S3 method overwritten by 'ggcor':
#>   method            from
#>   print.correlation nlme
quickcor(mtcars) + geom_colour()
```

<img src="man/figures/README-example01-1.png" width="100%" />

``` r
quickcor(mtcars, type = "upper") + geom_circle2()
```

<img src="man/figures/README-example01-2.png" width="100%" />

``` r
quickcor(mtcars, cor.test = TRUE) +
  geom_square(data = get_data(type = "lower", show.diag = FALSE)) +
  geom_mark(data = get_data(type = "upper", show.diag = FALSE), size = 2.5) +
  geom_abline(slope = -1, intercept = 12)
```

<img src="man/figures/README-example01-3.png" width="100%" />

## Grouped by rows

``` r
grp <- mtcars$gear
quickcor(mtcars[-10], type = "lower", group = grp) + 
  geom_colour() +
  add_diag_label(hjust = 0) +
  expand_axis(x = 12) +
  remove_axis("x") +
  facet_wrap(~.group)
#> Warning in cor(x, y, use = use, method = method): 标准差为零

#> Warning in cor(x, y, use = use, method = method): 标准差为零
```

<img src="man/figures/README-example02-1.png" width="100%" />

## Mantel test plot

``` r
library(vegan)
#> Loading required package: permute
#> Loading required package: lattice
#> This is vegan 2.5-6
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
data("varechem")
data("varespec")
set.seed(20191224)
sam_grp <- sample(paste0("sample", 1:3), 24, replace = TRUE)
mantel01 <- fortify_mantel(varespec, varechem, group = sam_grp,
                           spec.select = list(spec01 = 1:5, 
                                              spec02 = 6:12,
                                              spec03 = 7:18,
                                              spec04 = 20:29,
                                              spec05 = 30:44),
                           mantel.fun = "mantel.randtest")
quickcor(mantel01, legend.title = "Mantel's r") + 
  geom_colour() + geom_cross() + facet_grid(rows = vars(.group))
```

<img src="man/figures/README-example03-1.png" width="100%" />

``` r
mantel02 <- fortify_mantel(varespec, varechem, 
                         spec.select = list(1:10, 5:14, 7:22, 9:32)) %>% 
  mutate(r = cut(r, breaks = c(-Inf, 0.25, 0.5, Inf), 
                 labels = c("<0.25", "0.25-0.5", ">=0.5"),
                 right = FALSE),
         p.value = cut(p.value, breaks = c(-Inf, 0.001, 0.01, 0.05, Inf),
                       labels = c("<0.001", "0.001-0.01", "0.01-0.05", ">=0.05"),
                       right = FALSE))
quickcor(varechem, type = "upper") + geom_square() + 
  add_link(mantel02, mapping = aes(colour = p.value, size = r),
           diag.label = TRUE) +
  scale_size_manual(values = c(0.5, 1.5, 3)) +
  add_diag_label() + remove_axis("x")
```

<img src="man/figures/README-example03-2.png" width="100%" />

# network

``` r
library(tidygraph)
#> 
#> Attaching package: 'tidygraph'
#> The following object is masked from 'package:stats':
#> 
#>     filter
library(ggraph)
net <- fast_correlate(varespec) %>% 
  as_tbl_graph(r.thres = 0.5, p.thres = 0.05) %>% 
  mutate(degree = tidygraph::centrality_degree(mode = "all"))
#> 

ggraph(net, "circle") + 
  geom_edge_fan(aes(edge_width = r, edge_linetype = r < 0), 
                edge_colour = "grey80", show.legend = FALSE) +
  geom_node_point(aes(size = degree, colour = name), 
                  show.legend = FALSE) +
  geom_node_text(aes(x = x * 1.08, y = y * 1.08, 
                     angle = -((-node_angle(x, y) + 90) %% 180) + 90, 
                     label = name), size = 4, hjust= "outward",
                 show.legend = FALSE)  +
  scale_edge_color_gradientn(colours = c("blue", "red")) +
  scale_edge_width_continuous(range = c(0.1, 1)) +
  coord_fixed(xlim = c(-1.4, 1.4), ylim = c(-1.4, 1.4)) +
  theme_graph()
```

<img src="man/figures/README-unnamed-chunk-2-1.png" width="100%" />
