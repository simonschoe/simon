---
date: 2022-12-22
title: Model Calibration
subtitle: From Uncalibrated Confidence Scores to Calibrated Probabilities
summary: Model calibrations convert a model's uncalibrated (soft) probabilities, also refered to as confidence scores, into interpretable, well-calibrated probabilities. They establish a link between model uncertainty and actual probabilities (i.e., calibrated predicted probabilities).
format: hugo
draft: false
slug: model-calibration
---

### Confidence Scores

Model calibrations convert a model's uncalibrated (soft) probabilities, also refered to as *confidence scores*, into interpretable, well-calibrated probabilities. They establish a link between model uncertainty and actual probabilities (i.e., calibrated predicted probabilities). Misscalibrations can be identified using various visual diagnostics, e.g., implemented in the [probably](https://github.com/tidymodels/probably/) R package.

``` r
library(tidyverse)
library(probably)
```

First, I map logits into confidence scores using the `softmax()` function (for now I ignore the scaling parameter `temp`):

``` r
softmax <- function(x, temp = 1) {
  exp(x / temp) / sum(exp(x / temp))
}
```

``` r
df <- data %>% 
  mutate(.pred_neg = map(logits, ~ softmax(.) %>% .[1]) %>% unlist,
         .pred_pos = map(logits, ~ softmax(.) %>% .[2]) %>% unlist)

df
## # A tibble: 500 x 4
##    labels logits    .pred_neg .pred_pos
##    <fct>  <list>        <dbl>     <dbl>
##  1 0      <dbl [2]>    0.781    0.219  
##  2 0      <dbl [2]>    0.975    0.0249 
##  3 0      <dbl [2]>    0.0332   0.967  
##  4 0      <dbl [2]>    0.997    0.00270
##  5 0      <dbl [2]>    0.551    0.449  
##  6 0      <dbl [2]>    0.996    0.00365
##  7 0      <dbl [2]>    0.997    0.00266
##  8 0      <dbl [2]>    0.993    0.00722
##  9 0      <dbl [2]>    0.960    0.0401 
## 10 1      <dbl [2]>    0.265    0.735  
## # ... with 490 more rows
```

### Reliability Diagrams

Reliability diagrams for diagnosing model miscalibration was advocated in [On Calibration of Modern Neural Networks](https://arxiv.org/abs/1706.04599). Reliability diagrams plot the event rate as a function of model confidence using out-of-sample calibration data (i.e., data unseen by the model). Ideally, the model's predicted probabilities should reflect the true event rate in the calibration data (indicated by the dashed line below). True event rate \> (\<) model confidence indicates underprediction (overprediction), i.e., more false negatives (false positives). For example, for observations that receive a confidence score between 0.5 and 0.6, the true event rate in the calibration data should be similar. In this example, it is much lower (slightly above 25%), suggesting that the model severely underpredicts in this range.

``` r
rel_plot <- function(data, proba_pos, label) {
  
  proba_pos <- enquo(proba_pos)
  label <- enquo(label)
  
  breaks = seq(0, 1, 0.1)
  
  data %>% 
    mutate(bin = cut(!! proba_pos, breaks, labels = F) / 10,
           .pred_class = if_else(!! proba_pos > 0.5, 1L, 0L)) %>% 
    summarise(accuracy = mean(as.integer(!! label) - 1), .by = bin) %>% 
    ggplot(aes(x = bin, y = accuracy)) +
    geom_col(just = 1, fill = "darkblue", color = "black", alpha = 0.8) +
    geom_abline(intercept = 0, slope = 1, lty = "dashed") +
    scale_x_continuous(breaks = breaks) +
    scale_y_continuous(limits = c(0L, 1L)) +
    theme_bw() +
    labs(
      x = "Model Confidence",
      y = "True Event Rate"
    )
}
```

``` r
df %>% 
  rel_plot(.pred_pos, labels)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-7-1.png" style="width:75.0%" data-fig-align="center" />

### Calibration Plot

Calibration plots are almost similar to reliability diagrams. They again visualize the relationship between the model's confidence score and the true event rate in the calibration data. As before, a well-calibrated model should output scores that show a perfect, linear correlation with the true event rate. The rugs on the bottom (top) indicate events (non-events) in this case, while the light-blue ribbon illustrate confidence bands. In contrast to the previous reliability diagram, a smooth GAM is fitted to approximate the relationship.

``` r
df %>% 
  cal_plot_logistic(labels, .pred_pos, conf_level = 0.95, smooth = T, event_level = "second") +
  scale_x_continuous(breaks = seq(0, 1, 0.1))
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-8-1.png" style="width:100.0%" data-fig-align="center" />

### Temperature Scaling

After having diagnosed miscalibrations in the model's output, I'd like to mitigate the issue by applying some postprocessing techniques, in this case temperature scaling. Using a simple scaling factor (`temp`) in the softmax function to transform the logits, the distribution of model confidence scores can be smoothed out without affecting the performance the model itself. That is, `temp` effectively squishes or spreads out the confidence scores when selecting `temp > 1` or `temp < 1` without changing the rank ordering of the predictions. More well-behaved model scores can be obtained by selecting `temp` to better approximate the dashed, bisecting line.

``` r
for (i in seq(1, 3, 0.25)) {
  
  p <- data %>% 
  mutate(.pred_neg = map(logits, ~ softmax(., temp = i) %>% .[1]) %>% unlist,
         .pred_pos = map(logits, ~ softmax(., temp = i) %>% .[2]) %>% unlist)%>% 
  cal_plot_logistic(labels, .pred_pos, conf_level = 0.95, smooth = T, event_level = "second") +
  scale_x_continuous(breaks = seq(0, 1, 0.1)) +
  labs(title = glue::glue("Temperature: {i}"))
    
  print(p)
  
}
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-1.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-2.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-3.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-4.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-5.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-6.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-7.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-8.png" style="width:75.0%" data-fig-align="center" />

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-9.png" style="width:75.0%" data-fig-align="center" />
