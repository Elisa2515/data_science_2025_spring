Antibiotics
================
Elisa Camacho
2020-

*Purpose*: Creating effective data visualizations is an *iterative*
process; very rarely will the first graph you make be the most
effective. The most effective thing you can do to be successful in this
iterative process is to *try multiple graphs* of the same data.

Furthermore, judging the effectiveness of a visual is completely
dependent on *the question you are trying to answer*. A visual that is
totally ineffective for one question may be perfect for answering a
different question.

In this challenge, you will practice *iterating* on data visualization,
and will anchor the *assessment* of your visuals using two different
questions.

*Note*: Please complete your initial visual design **alone**. Work on
both of your graphs alone, and save a version to your repo *before*
coming together with your team. This way you can all bring a diversity
of ideas to the table!

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

    ## Warning: package 'readr' was built under R version 4.4.3

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
library(ggrepel)
```

    ## Warning: package 'ggrepel' was built under R version 4.4.3

*Background*: The data\[1\] we study in this challenge report the
[*minimum inhibitory
concentration*](https://en.wikipedia.org/wiki/Minimum_inhibitory_concentration)
(MIC) of three drugs for different bacteria. The smaller the MIC for a
given drug and bacteria pair, the more practical the drug is for
treating that particular bacteria. An MIC value of *at most* 0.1 is
considered necessary for treating human patients.

These data report MIC values for three antibiotics—penicillin,
streptomycin, and neomycin—on 16 bacteria. Bacteria are categorized into
a genus based on a number of features, including their resistance to
antibiotics.

``` r
## NOTE: If you extracted all challenges to the same location,
## you shouldn't have to change this filename
filename <- "./data/antibiotics.csv"

## Load the data
df_antibiotics <- read_csv(filename)
```

    ## Rows: 16 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): bacteria, gram
    ## dbl (3): penicillin, streptomycin, neomycin
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
df_antibiotics %>% knitr::kable()
```

| bacteria                        | penicillin | streptomycin | neomycin | gram     |
|:--------------------------------|-----------:|-------------:|---------:|:---------|
| Aerobacter aerogenes            |    870.000 |         1.00 |    1.600 | negative |
| Brucella abortus                |      1.000 |         2.00 |    0.020 | negative |
| Bacillus anthracis              |      0.001 |         0.01 |    0.007 | positive |
| Diplococcus pneumonia           |      0.005 |        11.00 |   10.000 | positive |
| Escherichia coli                |    100.000 |         0.40 |    0.100 | negative |
| Klebsiella pneumoniae           |    850.000 |         1.20 |    1.000 | negative |
| Mycobacterium tuberculosis      |    800.000 |         5.00 |    2.000 | negative |
| Proteus vulgaris                |      3.000 |         0.10 |    0.100 | negative |
| Pseudomonas aeruginosa          |    850.000 |         2.00 |    0.400 | negative |
| Salmonella (Eberthella) typhosa |      1.000 |         0.40 |    0.008 | negative |
| Salmonella schottmuelleri       |     10.000 |         0.80 |    0.090 | negative |
| Staphylococcus albus            |      0.007 |         0.10 |    0.001 | positive |
| Staphylococcus aureus           |      0.030 |         0.03 |    0.001 | positive |
| Streptococcus fecalis           |      1.000 |         1.00 |    0.100 | positive |
| Streptococcus hemolyticus       |      0.001 |        14.00 |   10.000 | positive |
| Streptococcus viridans          |      0.005 |        10.00 |   40.000 | positive |

# Visualization

<!-- -------------------------------------------------- -->

### **q1** Prototype 5 visuals

To start, construct **5 qualitatively different visualizations of the
data** `df_antibiotics`. These **cannot** be simple variations on the
same graph; for instance, if two of your visuals could be made identical
by calling `coord_flip()`, then these are *not* qualitatively different.

For all five of the visuals, you must show information on *all 16
bacteria*. For the first two visuals, you must *show all variables*.

*Hint 1*: Try working quickly on this part; come up with a bunch of
ideas, and don’t fixate on any one idea for too long. You will have a
chance to refine later in this challenge.

*Hint 2*: The data `df_antibiotics` are in a *wide* format; it may be
helpful to `pivot_longer()` the data to make certain visuals easier to
construct.

#### Visual 1 (All variables)

In this visual you must show *all three* effectiveness values for *all
16 bacteria*. This means **it must be possible to identify each of the
16 bacteria by name.** You must also show whether or not each bacterium
is Gram positive or negative.

``` r
# WRITE YOUR CODE HERE
# Revised Visual 1: Ordered bar plot with log scale and reference line
df_long <- df_antibiotics %>%
  pivot_longer(cols = c(penicillin, streptomycin, neomycin), 
               names_to = "antibiotic", 
               values_to = "MIC") %>%
  # Order bacteria by average MIC
  mutate(bacteria = fct_reorder(bacteria, MIC, .fun = mean))

ggplot(df_long, aes(x = bacteria, y = MIC, fill = antibiotic)) +
  geom_bar(stat = "identity", position = "dodge") +
  geom_hline(yintercept = 0.1, linetype = "dashed", color = "red") +
  scale_y_log10() +
  facet_wrap(~gram, scales = "free_x") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  labs(title = "MIC Values for Antibiotics by Bacteria and Gram Stain",
       subtitle = "Dashed line shows MIC = 0.1 threshold for human treatment",
       x = "Bacteria",
       y = "Minimum Inhibitory Concentration (MIC, log10 scale)",
       fill = "Antibiotic")
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.1-1.png)<!-- -->

#### Visual 2 (All variables)

In this visual you must show *all three* effectiveness values for *all
16 bacteria*. This means **it must be possible to identify each of the
16 bacteria by name.** You must also show whether or not each bacterium
is Gram positive or negative.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE
# Revised Visual 2: Point plot with text labels
ggplot(df_long, aes(x = antibiotic, y = MIC, color = gram)) +
  geom_point(size = 3) +
  geom_text_repel(aes(label = bacteria), size = 3) +
  geom_hline(yintercept = 0.1, linetype = "dashed", color = "red") +
  scale_y_log10() +
  labs(title = "MIC Values for Antibiotics by Bacteria and Gram Stain",
       subtitle = "Dashed line shows MIC = 0.1 threshold for human treatment",
       x = "Antibiotic",
       y = "Minimum Inhibitory Concentration (MIC, log10 scale)",
       color = "Gram Stain")
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.2-1.png)<!-- -->

#### Visual 3 (Some variables)

In this visual you may show a *subset* of the variables (`penicillin`,
`streptomycin`, `neomycin`, `gram`), but you must still show *all 16
bacteria*.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE
# Revised Visual 3: Box plot with log scale
ggplot(df_long, aes(x = antibiotic, y = MIC, fill = gram)) +
  geom_boxplot() +
  geom_hline(yintercept = 0.1, linetype = "dashed", color = "red") +
  scale_y_log10() +
  labs(title = "Distribution of MIC Values by Antibiotic and Gram Stain",
       subtitle = "Dashed line shows MIC = 0.1 threshold for human treatment",
       x = "Antibiotic",
       y = "Minimum Inhibitory Concentration (MIC, log10 scale)",
       fill = "Gram Stain")
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.3-1.png)<!-- -->

#### Visual 4 (Some variables)

In this visual you may show a *subset* of the variables (`penicillin`,
`streptomycin`, `neomycin`, `gram`), but you must still show *all 16
bacteria*.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE
# Visual 4: Heatmap with log scale
ggplot(df_long, aes(x = antibiotic, y = fct_reorder(bacteria, MIC), fill = MIC)) +
  geom_tile() +
  geom_text(aes(label = round(MIC, 2)), size = 3) +
  scale_fill_gradient(low = "green", high = "red", trans = "log10") +
  facet_wrap(~gram, scales = "free_y") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1),
        axis.text.y = element_text(size = 8)) +
  labs(title = "Heatmap of Antibiotic Effectiveness",
       subtitle = "Color intensity shows MIC value (log10 scale)",
       x = "Antibiotic",
       y = "Bacteria",
       fill = "MIC (log10)")
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.4-1.png)<!-- -->

#### Visual 5 (Some variables)

In this visual you may show a *subset* of the variables (`penicillin`,
`streptomycin`, `neomycin`, `gram`), but you must still show *all 16
bacteria*.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE
# Visual 5: Parallel coordinates with log scale
ggplot(df_long, aes(x = antibiotic, y = MIC, group = bacteria, color = gram)) +
  geom_line() +
  geom_point() +
  geom_hline(yintercept = 0.1, linetype = "dashed", color = "red") +
  scale_y_log10() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  labs(title = "Parallel Coordinates Plot of Antibiotic Effectiveness",
       subtitle = "Dashed line shows MIC = 0.1 threshold for human treatment",
       x = "Antibiotic",
       y = "MIC (log10 scale)",
       color = "Gram Stain")
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.5-1.png)<!-- -->

### **q2** Assess your visuals

There are **two questions** below; use your five visuals to help answer
both Guiding Questions. Note that you must also identify which of your
five visuals were most helpful in answering the questions.

*Hint 1*: It’s possible that *none* of your visuals is effective in
answering the questions below. You may need to revise one or more of
your visuals to answer the questions below!

*Hint 2*: It’s **highly unlikely** that the same visual is the most
effective at helping answer both guiding questions. **Use this as an
opportunity to think about why this is.**

#### Guiding Question 1

> How do the three antibiotics vary in their effectiveness against
> bacteria of different genera and Gram stain?

*Observations* - What is your response to the question above?

Penicillin shows clear Gram-positive preference (lower MIC values)

Neomycin is most consistently effective across Gram types

Streptomycin shows wide variability, with some Gram-negatives being
susceptible

The clinical threshold (MIC ≤ 0.1) is most consistently achieved by
neomycin

- Which of your visuals above (1 through 5) is **most effective** at
  helping to answer this question?
- Why?

Visual 3 (Box plot) is most effective because:

Clearly shows distribution differences between Gram types

Highlights the clinical threshold with reference line

Allows easy comparison across all three antibiotics

#### Guiding Question 2

In 1974 *Diplococcus pneumoniae* was renamed *Streptococcus pneumoniae*,
and in 1984 *Streptococcus fecalis* was renamed *Enterococcus fecalis*
\[2\].

> Why was *Diplococcus pneumoniae* was renamed *Streptococcus
> pneumoniae*?

*Observations* - What is your response to the question above? The
resistance profile (high penicillin MIC, low neomycin MIC) matches other
Streptococcus species

Both are Gram-positive

Similar antibiotic susceptibility patterns suggest shared biological
characteristics

Renaming likely reflected improved understanding of genetic
relationships

- Which of your visuals above (1 through 5) is **most effective** at
  helping to answer this question?
- Why? Visual 5 (Parallel coordinates) is most effective because, it
  shows complete resistance profiles for each bacterium. Clearly
  demonstrates similar patterns between Diplococcus and Streptococcus
  and maintains individual bacterium identity while showing all three
  antibiotics

# References

<!-- -------------------------------------------------- -->

\[1\] Neomycin in skin infections: A new topical antibiotic with wide
antibacterial range and rarely sensitizing. Scope. 1951;3(5):4-7.

\[2\] Wainer and Lysen, “That’s Funny…” *American Scientist* (2009)
[link](https://www.americanscientist.org/article/thats-funny)
