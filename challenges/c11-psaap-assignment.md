Regression Case Study: PSAAP II
================
Elisa Camacho
2020-

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [Orientation: Exploring Simulation
  Results](#orientation-exploring-simulation-results)
  - [**q1** Perform your “initial checks” to get a sense of the
    data.](#q1-perform-your-initial-checks-to-get-a-sense-of-the-data)
  - [**q2** Visualize `T_norm` against `x`. Note that there are multiple
    simulations at different values of the Input variables: Each
    simulation result is identified by a different value of
    `idx`.](#q2-visualize-t_norm-against-x-note-that-there-are-multiple-simulations-at-different-values-of-the-input-variables-each-simulation-result-is-identified-by-a-different-value-of-idx)
  - [Modeling](#modeling)
    - [**q3** The following code chunk fits a few different models.
      Compute a measure of model accuracy for each model on
      `df_validate`, and compare their
      performance.](#q3-the-following-code-chunk-fits-a-few-different-models-compute-a-measure-of-model-accuracy-for-each-model-on-df_validate-and-compare-their-performance)
    - [**q4** Interpret this model](#q4-interpret-this-model)
  - [Contrasting CI and PI](#contrasting-ci-and-pi)
    - [**q5** The following code will construct a predicted-vs-actual
      plot with your model from *q4* and add prediction intervals. Study
      the results and answer the questions below under
      *observations*.](#q5-the-following-code-will-construct-a-predicted-vs-actual-plot-with-your-model-from-q4-and-add-prediction-intervals-study-the-results-and-answer-the-questions-below-under-observations)
- [Case Study: Predicting Performance
  Ranges](#case-study-predicting-performance-ranges)
  - [**q6** You are consulting with a team that is designing a prototype
    heat transfer device. They are asking you to help determine a
    *dependable range of values* for `T_norm` they can design around for
    this *single prototype*. The realized value of `T_norm` must not be
    too high as it may damage the downstream equipment, but it must also
    be high enough to extract an acceptable amount of
    heat.](#q6-you-are-consulting-with-a-team-that-is-designing-a-prototype-heat-transfer-device-they-are-asking-you-to-help-determine-a-dependable-range-of-values-for-t_norm-they-can-design-around-for-this-single-prototype-the-realized-value-of-t_norm-must-not-be-too-high-as-it-may-damage-the-downstream-equipment-but-it-must-also-be-high-enough-to-extract-an-acceptable-amount-of-heat)
- [References](#references)

*Purpose*: Confidence and prediction intervals are useful for studying
“pure sampling” of some distribution. However, we can combine CI and PI
with regression analysis to equip our modeling efforts with powerful
notions of uncertainty. In this challenge, you will use fluid simulation
data in a regression analysis with uncertainty quantification (CI and
PI) to support engineering design.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category | Needs Improvement | Satisfactory |
|----|----|----|
| Effort | Some task **q**’s left unattempted | All task **q**’s attempted |
| Observed | Did not document observations, or observations incorrect | Documented correct observations based on analysis |
| Supported | Some observations not clearly supported by analysis | All observations clearly supported by analysis (table, graph, etc.) |
| Assessed | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support |
| Specified | Uses the phrase “more data are necessary” without clarification | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability | Code sufficiently close to the [style guide](https://style.tidyverse.org/) |

## Submission

<!-- ------------------------- -->

Make sure to commit both the challenge report (`report.md` file) and
supporting files (`report_files/` folder) when you are done! Then submit
a link to Canvas. **Your Challenge submission is not complete without
all files uploaded to GitHub.**

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(modelr)
library(broom)
```

    ## 
    ## Attaching package: 'broom'
    ## 
    ## The following object is masked from 'package:modelr':
    ## 
    ##     bootstrap

``` r
## Helper function to compute uncertainty bounds
add_uncertainties <- function(data, model, prefix = "pred", ...) {
  df_fit <-
    stats::predict(model, data, ...) %>%
    as_tibble() %>%
    rename_with(~ str_c(prefix, "_", .))

  bind_cols(data, df_fit)
}
```

# Orientation: Exploring Simulation Results

*Background*: The data you will study in this exercise come from a
computational fluid dynamics (CFD) [simulation
campaign](https://www.sciencedirect.com/science/article/abs/pii/S0301932219308651?via%3Dihub)
that studied the interaction of turbulent flow and radiative heat
transfer to fluid-suspended particles\[1\]. These simulations were
carried out to help study a novel design of [solar
receiver](https://en.wikipedia.org/wiki/Concentrated_solar_power),
though they are more aimed at fundamental physics than detailed device
design. The following code chunk downloads and unpacks the data to your
local `./data/` folder.

``` r
## NOTE: No need to edit this chunk
## Download PSAAP II data and unzip
url_zip <- "https://ndownloader.figshare.com/files/24111269"
filename_zip <- "./data/psaap.zip"
filename_psaap <- "./data/psaap.csv"

curl::curl_download(url_zip, destfile = filename_zip)
unzip(filename_zip, exdir = "./data")
df_psaap <- read_csv(filename_psaap)
```

    ## Rows: 140 Columns: 22
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (22): x, idx, L, W, U_0, N_p, k_f, T_f, rho_f, mu_f, lam_f, C_fp, rho_p,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

![PSAAP II irradiated core flow](./images/psaap-setup.png) Figure 1. An
example simulation, frozen at a specific point in time. An initial
simulation is run (HIT SECTION) to generate a turbulent flow with
particles, and that swirling flow is released into a rectangular domain
(RADIATED SECTION) with bulk downstream flow (left to right).
Concentrated solar radiation transmits through the optically transparent
fluid, but deposits heat into the particles. The particles then convect
heat into the fluid, which heats up the flow. The false-color image
shows the fluid temperature: Notice that there are “hot spots” where hot
particles have deposited heat into the fluid. The dataset `df_psaap`
gives measurements of `T_norm = (T - T0) / T0` averaged across planes at
various locations along the RADIATED SECTION.

### **q1** Perform your “initial checks” to get a sense of the data.

``` r
## TODO: Perform your initial checks
str(df_psaap)
```

    ## spc_tbl_ [140 × 22] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ x     : num [1:140] 0.25 0.25 0.25 0.25 0.25 0.25 0.25 0.25 0.25 0.25 ...
    ##  $ idx   : num [1:140] 1 2 3 4 5 6 7 8 9 10 ...
    ##  $ L     : num [1:140] 0.19 0.151 0.169 0.135 0.201 ...
    ##  $ W     : num [1:140] 0.0342 0.0464 0.0398 0.0325 0.0441 ...
    ##  $ U_0   : num [1:140] 1.86 2.23 2.04 2.45 1.7 ...
    ##  $ N_p   : num [1:140] 1600067 2215857 1707729 2076161 1945272 ...
    ##  $ k_f   : num [1:140] 0.0832 0.1112 0.0867 0.1208 0.0904 ...
    ##  $ T_f   : num [1:140] 300 243 290 358 252 ...
    ##  $ rho_f : num [1:140] 1.16 1.13 1.1 1.23 1.44 ...
    ##  $ mu_f  : num [1:140] 1.52e-05 1.84e-05 2.18e-05 2.23e-05 2.28e-05 ...
    ##  $ lam_f : num [1:140] 0.0316 0.0259 0.0349 0.037 0.0356 ...
    ##  $ C_fp  : num [1:140] 1062 1114 952 998 937 ...
    ##  $ rho_p : num [1:140] 8416 10648 10806 10966 7819 ...
    ##  $ d_p   : num [1:140] 1.07e-05 1.10e-05 1.24e-05 9.73e-06 1.14e-05 ...
    ##  $ C_pv  : num [1:140] 468 383 529 463 414 ...
    ##  $ h     : num [1:140] 6279 4667 6148 4921 6019 ...
    ##  $ I_0   : num [1:140] 7876978 6551358 6121350 6363488 8512473 ...
    ##  $ eps_p : num [1:140] 0.443 0.325 0.403 0.389 0.439 ...
    ##  $ avg_q : num [1:140] 689523 684218 619206 1070186 577245 ...
    ##  $ avg_T : num [1:140] 485 291 402 447 393 ...
    ##  $ rms_T : num [1:140] 7.61 4.19 5.61 4.48 6.95 ...
    ##  $ T_norm: num [1:140] 0.616 0.198 0.386 0.251 0.559 ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   x = col_double(),
    ##   ..   idx = col_double(),
    ##   ..   L = col_double(),
    ##   ..   W = col_double(),
    ##   ..   U_0 = col_double(),
    ##   ..   N_p = col_double(),
    ##   ..   k_f = col_double(),
    ##   ..   T_f = col_double(),
    ##   ..   rho_f = col_double(),
    ##   ..   mu_f = col_double(),
    ##   ..   lam_f = col_double(),
    ##   ..   C_fp = col_double(),
    ##   ..   rho_p = col_double(),
    ##   ..   d_p = col_double(),
    ##   ..   C_pv = col_double(),
    ##   ..   h = col_double(),
    ##   ..   I_0 = col_double(),
    ##   ..   eps_p = col_double(),
    ##   ..   avg_q = col_double(),
    ##   ..   avg_T = col_double(),
    ##   ..   rms_T = col_double(),
    ##   ..   T_norm = col_double()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

``` r
summary(df_psaap)
```

    ##        x               idx           L                W          
    ##  Min.   :0.2500   Min.   : 1   Min.   :0.1292   Min.   :0.03198  
    ##  1st Qu.:0.4375   1st Qu.: 9   1st Qu.:0.1448   1st Qu.:0.03539  
    ##  Median :0.6250   Median :18   Median :0.1623   Median :0.03983  
    ##  Mean   :0.6250   Mean   :18   Mean   :0.1631   Mean   :0.04022  
    ##  3rd Qu.:0.8125   3rd Qu.:27   3rd Qu.:0.1819   3rd Qu.:0.04482  
    ##  Max.   :1.0000   Max.   :35   Max.   :0.2009   Max.   :0.04960  
    ##       U_0             N_p               k_f               T_f       
    ##  Min.   :1.667   Min.   :1527347   Min.   :0.07954   Min.   :241.9  
    ##  1st Qu.:1.846   1st Qu.:1707729   1st Qu.:0.08674   1st Qu.:262.3  
    ##  Median :2.075   Median :1909414   Median :0.09822   Median :291.4  
    ##  Mean   :2.094   Mean   :1929614   Mean   :0.09964   Mean   :298.3  
    ##  3rd Qu.:2.340   3rd Qu.:2154872   3rd Qu.:0.11123   3rd Qu.:331.7  
    ##  Max.   :2.583   Max.   :2387055   Max.   :0.12360   Max.   :370.5  
    ##      rho_f             mu_f               lam_f              C_fp       
    ##  Min.   :0.9637   Min.   :1.519e-05   Min.   :0.02393   Min.   : 813.2  
    ##  1st Qu.:1.0728   1st Qu.:1.672e-05   1st Qu.:0.02642   1st Qu.: 922.2  
    ##  Median :1.1943   Median :1.893e-05   Median :0.02976   Median :1013.4  
    ##  Mean   :1.2059   Mean   :1.902e-05   Mean   :0.03033   Mean   :1025.0  
    ##  3rd Qu.:1.3358   3rd Qu.:2.126e-05   3rd Qu.:0.03352   3rd Qu.:1131.3  
    ##  Max.   :1.4871   Max.   :2.340e-05   Max.   :0.03762   Max.   :1262.9  
    ##      rho_p            d_p                 C_pv             h       
    ##  Min.   : 7159   Min.   :8.497e-06   Min.   :362.2   Min.   :4569  
    ##  1st Qu.: 8053   1st Qu.:9.493e-06   1st Qu.:413.9   1st Qu.:5134  
    ##  Median : 9058   Median :1.061e-05   Median :462.5   Median :5830  
    ##  Mean   : 9144   Mean   :1.068e-05   Mean   :464.8   Mean   :5820  
    ##  3rd Qu.:10339   3rd Qu.:1.185e-05   3rd Qu.:516.9   3rd Qu.:6414  
    ##  Max.   :11128   Max.   :1.308e-05   Max.   :565.0   Max.   :7056  
    ##       I_0              eps_p            avg_q             avg_T      
    ##  Min.   :5664363   Min.   :0.3193   Min.   : 335025   Min.   :291.4  
    ##  1st Qu.:6363488   1st Qu.:0.3540   1st Qu.: 619232   1st Qu.:423.0  
    ##  Median :6943899   Median :0.3958   Median : 689560   Median :491.3  
    ##  Mean   :7095833   Mean   :0.4018   Mean   : 777490   Mean   :513.0  
    ##  3rd Qu.:7953745   3rd Qu.:0.4427   3rd Qu.: 978892   3rd Qu.:582.3  
    ##  Max.   :8849196   Max.   :0.4950   Max.   :1498542   Max.   :938.2  
    ##      rms_T           T_norm      
    ##  Min.   :3.387   Min.   :0.1215  
    ##  1st Qu.:4.937   1st Qu.:0.3889  
    ##  Median :5.698   Median :0.6328  
    ##  Mean   :5.961   Mean   :0.7360  
    ##  3rd Qu.:6.948   3rd Qu.:0.9795  
    ##  Max.   :9.254   Max.   :2.2840

``` r
head(df_psaap)
```

    ## # A tibble: 6 × 22
    ##       x   idx     L      W   U_0     N_p    k_f   T_f rho_f    mu_f  lam_f  C_fp
    ##   <dbl> <dbl> <dbl>  <dbl> <dbl>   <dbl>  <dbl> <dbl> <dbl>   <dbl>  <dbl> <dbl>
    ## 1  0.25     1 0.190 0.0342  1.86  1.60e6 0.0832  300. 1.16  1.52e-5 0.0316 1062.
    ## 2  0.25     2 0.151 0.0464  2.23  2.22e6 0.111   243. 1.13  1.84e-5 0.0259 1114.
    ## 3  0.25     3 0.169 0.0398  2.04  1.71e6 0.0867  290. 1.10  2.18e-5 0.0349  952.
    ## 4  0.25     4 0.135 0.0325  2.45  2.08e6 0.121   358. 1.23  2.23e-5 0.0370  998.
    ## 5  0.25     5 0.201 0.0441  1.70  1.95e6 0.0904  252. 1.44  2.28e-5 0.0356  937.
    ## 6  0.25     6 0.160 0.0379  1.96  1.82e6 0.0798  280. 0.964 2.13e-5 0.0249 1224.
    ## # ℹ 10 more variables: rho_p <dbl>, d_p <dbl>, C_pv <dbl>, h <dbl>, I_0 <dbl>,
    ## #   eps_p <dbl>, avg_q <dbl>, avg_T <dbl>, rms_T <dbl>, T_norm <dbl>

``` r
sum(is.na(df_psaap))
```

    ## [1] 0

``` r
sapply(df_psaap, function(x) length(unique(x)))
```

    ##      x    idx      L      W    U_0    N_p    k_f    T_f  rho_f   mu_f  lam_f 
    ##      4     35     35     35     35     35     35     35     35     35     35 
    ##   C_fp  rho_p    d_p   C_pv      h    I_0  eps_p  avg_q  avg_T  rms_T T_norm 
    ##     35     35     35     35     35     35     35    140    140    140    140

**Observations**:

- (Take notes here!)

The important variables in this dataset are:

| Variable | Category | Meaning                           |
|----------|----------|-----------------------------------|
| `x`      | Spatial  | Channel location                  |
| `idx`    | Metadata | Simulation run                    |
| `L`      | Input    | Channel length                    |
| `W`      | Input    | Channel width                     |
| `U_0`    | Input    | Bulk velocity                     |
| `N_p`    | Input    | Number of particles               |
| `k_f`    | Input    | Turbulence level                  |
| `T_f`    | Input    | Fluid inlet temp                  |
| `rho_f`  | Input    | Fluid density                     |
| `mu_f`   | Input    | Fluid viscosity                   |
| `lam_f`  | Input    | Fluid conductivity                |
| `C_fp`   | Input    | Fluid isobaric heat capacity      |
| `rho_p`  | Input    | Particle density                  |
| `d_p`    | Input    | Particle diameter                 |
| `C_pv`   | Input    | Particle isochoric heat capacity  |
| `h`      | Input    | Convection coefficient            |
| `I_0`    | Input    | Radiation intensity               |
| `eps_p`  | Input    | Radiation absorption coefficient  |
| `avg_q`  | Output   | Plane-averaged heat flux          |
| `avg_T`  | Output   | Plane-averaged fluid temperature  |
| `rms_T`  | Output   | Plane-rms fluid temperature       |
| `T_norm` | Output   | Normalized fluid temperature rise |

The primary output of interest is `T_norm = (avg_T - T_f) / T_f`, the
normalized (dimensionless) temperature rise of the fluid, due to heat
transfer. These measurements are taken at locations `x` along a column
of fluid, for different experimental settings (e.g. different dimensions
`W, L`, different flow speeds `U_0`, etc.).

### **q2** Visualize `T_norm` against `x`. Note that there are multiple simulations at different values of the Input variables: Each simulation result is identified by a different value of `idx`.

``` r
## TODO: Visualize the data in df_psaap with T_norm against x;
##       design your visual to handle the multiple simulations,
##       each identified by different values of idx
# Create a line plot with different colors for each simulation (idx)
ggplot(df_psaap, aes(x = x, y = T_norm, group = idx, color = factor(idx))) +
  geom_line(alpha = 0.7) +
  labs(title = "Normalized Temperature (T_norm) vs. Position (x) by Simulation",
       x = "Position (x)",
       y = "Normalized Temperature (T_norm)",
       color = "Simulation ID") +
  theme_minimal() +
  theme(legend.position = "bottom")
```

![](c11-psaap-assignment_files/figure-gfm/q2-task-1.png)<!-- -->

## Modeling

The following chunk will split the data into training and validation
sets.

``` r
## NOTE: No need to edit this chunk
# Addl' Note: These data are already randomized by idx; no need
# to additionally shuffle the data!
df_train <- df_psaap %>% filter(idx %in% 1:20)
df_validate <- df_psaap %>% filter(idx %in% 21:36)
```

One of the key decisions we must make in modeling is choosing predictors
(features) from our observations to include in the model. Ideally we
should have some intuition for why these predictors are reasonable to
include in the model; for instance, we saw above that location along the
flow `x` tends to affect the temperature rise `T_norm`. This is because
fluid downstream has been exposed to solar radiation for longer, and
thus is likely to be at a higher temperature.

Reasoning about our variables—at least at a *high level*—can help us to
avoid including *fallacious* predictors in our models. You’ll explore
this idea in the next task.

### **q3** The following code chunk fits a few different models. Compute a measure of model accuracy for each model on `df_validate`, and compare their performance.

``` r
## NOTE: No need to edit these models
fit_baseline <- 
  df_train %>% 
  lm(formula = T_norm ~ x)

fit_cheat <- 
  df_train %>% 
  lm(formula = T_norm ~ avg_T)

fit_nonphysical <- 
  df_train %>% 
  lm(formula = T_norm ~ idx)

## TODO: Compute a measure of accuracy for each fit above;
##       compare their relative performance
# Compute RMSE for each model on validation data
# Compute accuracy metrics for each model
# Function to calculate RMSE
calculate_rmse <- function(model, data) {
  predictions <- predict(model, newdata = data)
  sqrt(mean((data$T_norm - predictions)^2))
}

# 
rmse_baseline <- calculate_rmse(fit_baseline, df_validate)
rmse_cheat <- calculate_rmse(fit_cheat, df_validate)
rmse_nonphysical <- calculate_rmse(fit_nonphysical, df_validate)

#results table
results <- data.frame(
  Model = c("Baseline (x only)", "Cheat (avg_T)", "Nonphysical (idx)"),
  RMSE = c(rmse_baseline, rmse_cheat, rmse_nonphysical),
  R_squared = c(summary(fit_baseline)$r.squared,
               summary(fit_cheat)$r.squared,
               summary(fit_nonphysical)$r.squared)
)

results
```

    ##               Model      RMSE   R_squared
    ## 1 Baseline (x only) 0.2844778 0.348376352
    ## 2     Cheat (avg_T) 0.2317709 0.776630527
    ## 3 Nonphysical (idx) 0.3988128 0.001301518

**Observations**:

- Which model is *most accurate*? Which is *least accurate*?
  - Most Accurate: The one with the lowest RMSE 

  - Least accurate: the one with the highest RMSE
- What *Category* of variable is `avg_T`? Why is it such an effective
  predictor?
  - It is the same as T_norm just scaled differently
- Would we have access to `avg_T` if we were trying to predict a *new*
  value of `T_norm`? Is `avg_T` a valid predictor?
  - It not a valid predictor because we wouldn’t have this for real
    predictors.
- What *Category* of variable is `idx`? Does it have any physical
  meaning?
  - idx is a simulation ID number and it has no real physical meaning.

### **q4** Interpret this model

Interpret the following model by answering the questions below.

*Note*. The `-` syntax in R formulas allows us to exclude columns from
fitting. So `T_norm ~ . - x` would fit on all columns *except* `x`.

``` r
## TODO: Inspect the regression coefficients for the following model
fit_q4 <- 
  df_train %>% 
  # lm(formula = T_norm ~ . - idx - avg_q - avg_T - rms_T)
  lm(formula = T_norm ~ L + W + U_0 + N_p + k_f + T_f)
  # lm(formula = T_norm ~ L - W - U_0 - N_p - k_f - T_f)
tidy(fit_q4) %>% arrange(p.value)
```

    ## # A tibble: 7 × 5
    ##   term             estimate   std.error statistic     p.value
    ##   <chr>               <dbl>       <dbl>     <dbl>       <dbl>
    ## 1 (Intercept)   4.88        0.834           5.86  0.000000126
    ## 2 W           -44.4         8.03           -5.53  0.000000467
    ## 3 T_f          -0.00478     0.00115        -4.15  0.0000874  
    ## 4 N_p          -0.000000402 0.000000203    -1.98  0.0515     
    ## 5 L             2.57        2.00            1.28  0.203      
    ## 6 U_0          -0.168       0.185          -0.910 0.366      
    ## 7 k_f          -2.28        3.33           -0.685 0.496

``` r
#sd(df_psaap$x)
#sd(df_psaap$T_f)
```

**Observations**:

- Which columns are excluded in the model formula above? What categories
  do these belong to? Why are these important quantities to leave out of
  the model?
  - Excluded: idx, avg_q, avg_T, rms_T
  - These are either identifiers (idx) or measurements that weren’t
    available for prediction avg_q, avg_T, rms_T
  - Important to leave out because they’re either not predictive or
    would form data leakage
- Which inputs are *statistically significant*, according to the model?
  - Most significant: W, U_0, L (lowest p-values)
  - Less significant: k_f, N_p, T_f (higher p-values)
- What is the regression coefficient for `x`? What about the regression
  coefficient for `T_f`?
  - x isn’t included in this model

  - T_f coefficient: -4.78
- What is the standard deviation of `x` in `df_psaap`? What about the
  standard deviation of `T_f`?
  - x: about 0.28

  - T_f: about 38.94
- How do these standard deviations relate to the regression coefficients
  for `x` and `T_f`?
  - Variables with larger standard deviations tend to have smaller
    coefficients because their natural variation is larger
  - This scaling helps make coefficients comparable across variables
- Note that literally *all* of the inputs above have *some* effect on
  the output `T_norm`; so they are all “significant” in that sense. What
  does this tell us about the limitations of statistical significance
  for interpreting regression coefficients?
  - A variable can be statistically significant but have minimal
    practical effect.

## Contrasting CI and PI

Let’s revisit the ideas of confidence intervals (CI) and prediction
intervals (PI). Let’s fit a very simple model to these data, one which
only considers the channel location and ignores all other inputs. We’ll
also use the helper function `add_uncertainties()` (defined in the
`setup` chunk above) to add approximate CI and PI to the linear model.

``` r
## NOTE: No need to edit this chunk
fit_simple <-
  df_train %>%
  lm(data = ., formula = T_norm ~ x)

df_intervals <-
  df_train %>%
  add_uncertainties(fit_simple, interval = "confidence", prefix = "ci") %>%
  add_uncertainties(fit_simple, interval = "prediction", prefix = "pi")
```

The following figure visualizes the regression CI and PI against the
objects they are attempting to capture:

``` r
## NOTE: No need to edit this chunk
df_intervals %>%
  select(T_norm, x, matches("ci|pi")) %>%
  pivot_longer(
    names_to = c("method", ".value"),
    names_sep = "_",
    cols = matches("ci|pi")
  ) %>%

  ggplot(aes(x, fit)) +
  geom_errorbar(
    aes(ymin = lwr, ymax = upr, color = method),
    width = 0.05,
    size = 1
  ) +
  geom_smooth(
    data = df_psaap %>% mutate(method = "ci"),
    mapping = aes(x, T_norm),
    se = FALSE,
    linetype = 2,
    color = "black"
   ) +
  geom_point(
    data = df_validate %>% mutate(method = "pi"),
    mapping = aes(x, T_norm),
    size = 0.5
  ) +

  facet_grid(~method) +
  theme_minimal() +
  labs(
    x = "Channel Location (-)",
    y = "Normalized Temperature Rise (-)"
  )
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : pseudoinverse used at 0.24625

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : neighborhood radius 0.50375

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : reciprocal condition number 7.4302e-16

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : There are other near singularities as well. 0.25376

![](c11-psaap-assignment_files/figure-gfm/data-simple-model-vis-1.png)<!-- -->

Under the `ci` facet we have the regression confidence intervals and the
mean trend (computed with all the data `df_psaap`). Under the `pi` facet
we have the regression prediction intervals and the `df_validation`
observations.

**Punchline**:

- Confidence intervals are meant to capture the *mean trend*
- Prediction intervals are meant to capture *new observations*

Both CI and PI are a quantification of the uncertainty in our model, but
the two intervals designed to answer different questions.

Since CI and PI are a quantification of uncertainty, they should tend to
*narrow* as our model becomes more confident in its predictions.
Building a more accurate model will often lead to a reduction in
uncertainty. We’ll see this phenomenon in action with the following
task:

### **q5** The following code will construct a predicted-vs-actual plot with your model from *q4* and add prediction intervals. Study the results and answer the questions below under *observations*.

``` r
## TODO: Run this code and interpret the results
## NOTE: No need to edit this chunk
## NOTE: This chunk will use your model from q4; it will predict on the
##       validation data, add prediction intervals for every prediction,
##       and visualize the results on a predicted-vs-actual plot. It will
##       also compare against the simple `fit_simple` defined above.
bind_rows(
  df_psaap %>% 
    add_uncertainties(fit_simple, interval = "prediction", prefix = "pi") %>% 
    select(T_norm, pi_lwr, pi_fit, pi_upr) %>% 
    mutate(model = "x only"),
  df_psaap %>% 
    add_uncertainties(fit_q4, interval = "prediction", prefix = "pi") %>% 
    select(T_norm, pi_lwr, pi_fit, pi_upr) %>% 
    mutate(model = "q4"),
) %>% 
  
  ggplot(aes(T_norm, pi_fit)) +
  geom_abline(slope = 1, intercept = 0, color = "grey80", size = 2) +
  geom_errorbar(
    aes(ymin = pi_lwr, ymax = pi_upr),
    width = 0
  ) +
  geom_point() +
  
  facet_grid(~ model, labeller = label_both) +
  theme_minimal() +
  labs(
    title = "Predicted vs Actual",
    x = "Actual T_norm",
    y = "Predicted T_norm"
  )
```

![](c11-psaap-assignment_files/figure-gfm/q5-task-1.png)<!-- -->

**Observations**:

- Which model tends to be more accurate? How can you tell from this
  predicted-vs-actual plot?
  - The q4 model is more accurate and I can tell because the points are
    closer to the 1:1 line.
- Which model tends to be *more confident* in its predictions? Put
  differently, which model has *narrower prediction intervals*?
  - The q4 model has narrower prediction intervals
  - The additional predictors reduce the uncertainty in predictions
- How many predictors does the `fit_simple` model need in order to make
  a prediction? What about your model `fit_q4`?
  - The fit_sample needs just 1 predictor
  - fit_q4 needs 6 predictors

Based on these results, you might be tempted to always throw every
reasonable variable into the model. For some cases, that might be the
best choice. However, some variables might be *outside our control*; for
example, variables involving human behavior cannot be fully under our
control. Other variables may be *too difficult to measure*; for example,
it is *in theory* possible to predict the strength of a component by
having detailed knowledge of its microstructure. However, it is
*patently infeasible* to do a detailed study of *every single component*
that gets used in an airplane.

In both cases—human behavior and variable material properties—we would
be better off treating those quantities as random variables. There are
at least two ways we could treat these factors: 1. Explicitly model some
inputs as random variables and construct a model that *propagates* that
uncertainty from inputs to outputs, or 2. Implicitly model the
uncontrolled the uncontrolled variables by not including them as
predictors in the model, and instead relying on the error term
$\epsilon$ to represent these unaccounted factors. You will pursue
strategy 2. in the following Case Study.

# Case Study: Predicting Performance Ranges

### **q6** You are consulting with a team that is designing a prototype heat transfer device. They are asking you to help determine a *dependable range of values* for `T_norm` they can design around for this *single prototype*. The realized value of `T_norm` must not be too high as it may damage the downstream equipment, but it must also be high enough to extract an acceptable amount of heat.

In order to maximize the conditions under which this device can operate
successfully, the design team has chosen to fix the variables listed in
the table below, and consider the other variables to fluctuate according
to the values observed in `df_psaap`.

| Variable | Value    |
|----------|----------|
| `x`      | 1.0      |
| `L`      | 0.2      |
| `W`      | 0.04     |
| `U_0`    | 1.0      |
| (Other)  | (Varies) |

Your task is to use a regression analysis to deliver to the design team
a *dependable range* of values for `T_norm`, given their proposed
design, and at a fairly high level `0.8`. Perform your analysis below
(use the helper function `add_uncertainties()` with the `level`
argument!), and answer the questions below.

*Hint*: This problem will require you to *build a model* by choosing the
appropriate variables to include in the analysis. Think about *which
variables the design team can control*, and *which variables they have
chosen to allow to vary*. You will also need to choose between computing
a CI or PI for the design prediction.

``` r
# NOTE: No need to change df_design; this is the target the client
#       is considering
df_design <- tibble(x = 1, L = 0.2, W = 0.04, U_0 = 1.0)
# NOTE: This is the level the "probability" level customer wants
pr_level <- 0.8

## TODO: Fit a model, assess the uncertainty in your prediction, 
#        use the validation data to check your uncertainty estimates, and 
#        make a recommendation on a *dependable range* of values for T_norm
#        at the point `df_design`
# Fit model with design-relevant predictors
fit_q6 <- lm(T_norm ~ x + L + W + U_0, data = df_train)
df_design_pred <- add_uncertainties(df_design, fit_q6, 
                                   interval = "prediction", 
                                   level = pr_level,
                                   prefix = "pred")

df_validate_pred <- add_uncertainties(df_validate, fit_q6,
                                     interval = "prediction",
                                     level = pr_level,
                                     prefix = "pred")

coverage <- df_validate_pred %>%
  mutate(within_interval = T_norm >= pred_lwr & T_norm <= pred_upr) %>%
  summarise(coverage_rate = mean(within_interval))

# results
df_design_pred
```

    ## # A tibble: 1 × 7
    ##       x     L     W   U_0 pred_fit pred_lwr pred_upr
    ##   <dbl> <dbl> <dbl> <dbl>    <dbl>    <dbl>    <dbl>
    ## 1     1   0.2  0.04     1     1.88     1.46     2.30

``` r
coverage
```

    ## # A tibble: 1 × 1
    ##   coverage_rate
    ##           <dbl>
    ## 1         0.933

**Recommendation**:

- How much do you trust your model? Why?
  - I moderately trust this model because it includes key design
    variables but may miss some variability.
- What kind of interval—confidence or prediction—would you use for this
  task, and why?
  - I would use Prediction interval because we’re predicting individual
    observations, not mean behavior.
- What fraction of validation cases lie within the intervals you
  predict? (NB. Make sure to calculate your intervals *based on the
  validation data*; don’t just use one single interval!) How does this
  compare with `pr_level`?
  - Coverage rate is 0.93 this is close to the requested pr_level of
    0.8.
- What interval for `T_norm` would you recommend the design team to plan
  around?
  - Recomended Range \[1.45685, 2.296426\]
- Are there any other recommendations you would provide?
  - I would recommend controlling additional variables if tighter bound
    are needed.

*Bonus*: One way you could take this analysis further is to recommend
which other variables the design team should tightly control. You could
do this by fixing values in `df_design` and adding them to the model. An
exercise you could carry out would be to systematically test the
variables to see which ones the design team should more tightly control.

# References

- \[1\] Jofre, del Rosario, and Iaccarino “Data-driven dimensional
  analysis of heat transfer in irradiated particle-laden turbulent
  flow” (2020) *International Journal of Multiphase Flow*,
  <https://doi.org/10.1016/j.ijmultiphaseflow.2019.103198>
