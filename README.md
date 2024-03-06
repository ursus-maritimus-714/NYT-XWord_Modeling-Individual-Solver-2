# Predictive Modeling of Individual Solver 2 (IS2) Performance on the New York Times Crossword Puzzle

## Introduction

### Project Overview and Data Sources
This summary reports on the results of predictive modeling of an individual solver's (Individual Solver 2; IS2) performance over a large subset of 6 years (Mar. 2018 - Mar. 2024) of the [New York Times (NYT) crossword puzzle](https://www.nytimes.com/crosswords).  Previously, I conducted a [comprehensive exploratory data analysis (EDA) of IS2 performance](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Individual-Solver-2/blob/main/README.md) over this sample period. From this EDA, numerous features pertaining to both the puzzles themselves as well as to IS2 past performance were identified as candidate features for predictive modeling of IS2 solve times.    

Without access to two specific data sources this project would not have been possible. The first, [XWord Info: New York Times Crossword Answers and Insights](https://www.xwordinfo.com/), was my source for data on the puzzles themselves. This included a number of proprietary metrics pertaining to the grids, answers, clues and constructors. XWord Info has a contract with NYT for access to the raw data underlying these metrics, but I unfortunately do not. Therefore, I will not be able to share raw or processed data that I've acquired from their site. Nonetheless, [Jupyter notebooks](https://jupyter.org/) with all of my Python code for analysis and figure generation can be found [here](). The second, [XWStats](https://xwstats.com/), was my source for historical solve time data for both IS2 and for the "Global Median Solver" (GMS). XWStats (Matt) derives the GMS solve time as that at the 50th percentile out of ~1-2K individual solvers providing their solve times per puzzle. In the context of IS2 modeling, historical GMS solve times were used to derive a "Strength of Schedule" adjustment to features capturing IS2 past performance prior to a given solve time to be predicted (see Methods section for details). I have previously completed an [EDA](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Global-Median-Solver?tab=readme-ov-file#readme) and [predictive modeling](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Global-Median-Solver/blob/main/README.md) for the GMS. In addition, I have also completed [EDA](https://github.com/ursus-maritimus-714/NYT-XWord-EDA-Individual-Solver-1/blob/main/README.md) and [predictive modeling](https://github.com/ursus-maritimus-714/NYT-XWord-Modeling-Individual-Solver-1/blob/main/README.md) for another individual solver (IS1; me) as well. 

Please visit, explore and strongly consider financially supporting both of these wonderful sites; XWord Info via [membership purchase at one of several levels](https://www.xwordinfo.com/Pay) and XWStats via [BuyMeACoffee](https://www.buymeacoffee.com/xwstats). 

### Overview of NYT Crossword and IS2 Characteristics
The NYT crossword has been published since 1942, and many consider the "modern era" to have started with the arrival of Will Shortz as (only) its 4th editor 30 years ago. A new puzzle for each day is published online at either 6 PM (Sunday and Monday puzzles) or 10 PM (Tuesday-Saturday puzzles) ET the prior evening. Difficulty for the 15x15 grids (Monday-Saturday) is intended to increase gradually across the week, with Thursday generally including a gimmick or trick of some sort (e.g., "rebuses" where the solver must enter more than one character into one or more squares). Additionally, nearly all Sunday through Thursday puzzles have themes, some of which are revealed via letters placed in circled or shaded squares. Friday and Saturday are almost always themeless puzzles, and tend to have considerably more open constructions and longer (often multiword) answers than the early week puzzles. The clue sets tend to be more wordplay heavy/punny as the week goes on, and the answers become less common in the aggregate as well. Sunday puzzles have larger grids (21x21), and almost always feature a wordplay-intensive theme to which the longest answers in the puzzle pertain. The intended difficulty of the Sunday puzzle is approximately somewhere between a tough Wednesday and an easy Thursday. 

**Figure 1** shows dimensionality reduction via Principal Component Analysis (PCA) of 23 grid, clue and answer-related features obtained from XWord Info. This analysis demonstrates that, while puzzles from a given puzzle day do indeed aggregate with each other in n-dimensional "puzzle property space", the puzzle days themselves nonetheless exist along a continuum. Sunday is well-separated from the other puzzle days in this analysis by PCA1, which undoubtedly incorporates one or more grid size-contingent features.   

**<h4>Figure 1. PCA of Select Puzzle Grid, Clue and Answer Features**                                                                  

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/b19f5358-9be4-46d9-a0c8-032149f6f6f4)
*<h5>The first 3 principal components accounted for 47.4% of total variance. All puzzles issued from Jan. 1, 2018- Mar. 2, 2024 were included in this analysis (N=2,253).*

###

The overlapping distributions of per puzzle day IS2 solve times across the entire sample period (**Figure 2**) show a parallel performance phenomenon to the continuum of puzzle properties seen in **Fig. 1**. While solve difficulty increased as the week progressed, puzzle days of adjacent difficulty still had substantially overlapping IS2 solve time distributions. Other than for the "easy" days (Monday and Tuesday), distributions of IS2 solve times were quite broad. It should be noted, however, that the broadness of each puzzle day-specific IS2 solve time distribution was also increased by the dramatic improvement in IS2 performance over the full sample period.      

**<h4>Figure 2. Distributions of IS2 Solve Times by Puzzle Day for Full Sample Period**                   

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/e695ce55-5aeb-49da-9069-fb793d2d979c)
*<h5>All puzzles completed by IS2 in the sample period were included in this analysis (N=1,230).* 

### Key Outcomes from the IS2 EDA

One of the most important findings from the EDA as far as implications for predictive modeling was that, despite several periods of volatility across puzzle days, IS2 demonstrated continual, marked improvement over the course of the sample period across all puzzle days (**Figure 3**). Coupled to the fact that puzzle day-specific recent past performance ('Recent Performance Baseline' ['RPB']) was highly positively correlated to performance on the next puzzle both overall (r=.73) and across puzzle days (**Figure 4**), this created an imperative to explore and potentially include different variants of this feature type in the predictive modeling stage.    

**<h4>Figure 3. IS2 Solve Time Overview by Puzzle Day: 10-Puzzle Moving Averages and Distributions of Raw Values**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/6fd76d22-9e6e-4e64-b00b-3fbecc79a8e9)
*<h5>All puzzles completed by IS2 in the sample period were included in this analysis (N=1,230).* 

**<h4>Figure 4. Puzzle Day-Specific, Recent Performance Baseline (RPB) Correlation to IS2 Performance on the Next Puzzle**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/8c5af853-9584-4141-b940-8b7f011131ec)
*<h5> Puzzle-day specific, recent past performance (x-axis) was calculated over the 10 day-specific puzzles previous to the next solve (y-axis) to obtain 'RPB'. All puzzles completed by IS1 from Jan. 1, 2020- Mar. 2, 2024 were included in this analysis (N=1,132).* 

###
Along with the 'RPB' discussed above, multiple features pertaining to the puzzles themselves demonstrated moderately strong or strong correlations with IS2 performance on individual puzzles (**Figure 5**). Two that stood out in particular for their correlational strength with IS2 performance were 'Average Answer Length' and 'Freshness Factor', the latter of which is a proprietary XWord Info measure of the rareness of a given answer in the NYT puzzle. The strengths of these positive correlations with IS2 solve times (for all 15x15 puzzles: r=.56 and .57, respectively) can be seen both in the correlation heatmap (A; top row - 5th and 11th columns) as well as in the overall (black) and per-puzzle day (colors) feature correlation scatterplots in panel B. The feature density plots (C) show that the distributions of these features were well-separated across puzzle days. This is an important property for candidate predictive features to have since, as is shown in **Fig. 2**, distribution peaks of solve times for individual puzzle days were themselves well-separated.

**<h4>Figure 5. Correlations of Puzzle-Related and Past Performance Features to IS2 Solve Times**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/5a214e65-79a1-45cc-b090-d3c6b07cd415)
*<h5> All puzzles completed by IS2 from Jan. 1, 2020- Mar. 2, 2024 were included in this analysis (N=1,132).*

## Methods
### Predictive Feature Generation

For predictive feature generation, all puzzles completed by IS2 from Apr. 21, 2018-Mar. 2, 2024 (N=1,230) were included. This full sample included puzzles issued by NYT as early as March, 2018. The right panel of **Figure 6** summarizes predictive features included in the modeling stage (N=40) by broad class. A few key example features from each class are mentioned below. **Supplementary Table 1** comprehensively lists out, classifies and describes all included features.  

* Solver 'Past Performance' features (n=6) included 'IS_RPB_l10'. This feature captured puzzle day-specific 'Recent Performance Baseline' ('RPB') over the 10 puzzles immediately prior to a puzzle with a time being predicted. A number of temporal integration windows and time-decay weighting curves for this feature were tested in [preliminary univariate linear regression modeling](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/tree/main/notebooks/learning%20rate%20modeling), and a 10-puzzle window (l10) with *no* time-decay weighting yielded the lowest root mean square error (RMSE) mean training error. Furthermore, it was found that predictions with these parameters further improved with adjustment of this past performance feature by the performance of *the GMS* over the same set of puzzles relative to the *GMS' own* recent performance prior to those puzzles. This is referred to as 'Strength of Schedule adjustment' ('SOS adjustment') from here forward, and the left panel of **Fig. 6** depicts creation of this feature. Also note that another feature in this class, which captured normalized past performance against the constructor(s) of a puzzle being predicted ('IS Past Perf vs Constr'), used this 'SOS adjustment' in calculation of the baseline solve time expectation component (see **Supp. Table 1**).   
 
* 'Puzzle: Clue or Answer' features (n=19) included 'Freshness Factor', which measured the aggregate rarity of answers in a given puzzle across all NYT crossword puzzles from before or since the issue date. This class also included other measures of answer rarity, including the number of entirely unique answers in a puzzle ('Unique Answer #') and the 'Scrabble Score', which assigns corresponding Scrabble tile values to each letter in an answer (rarer letters = higher tile values, hence a different angle at assessing answer rarity). On the clue side of the ledger, this class also included a count of the frequency of wordplay in clues for a given puzzle ('Wordplay #'). Later week puzzles contained more such clues, and early week puzzles often contained very, very few. 

* 'Puzzle: Grid' features (n=11) included both the number of answers ('Answer #') and 'Average Answer Length' in a given puzzle. As puzzles got more difficult across the week, the former tended to decrease and the latter tended to increase. This class also included 'Open Squares #', which is a proprietary measure of XWord info capturing white squares not bordered by black squares (tended to increase as puzzles increased in difficulty across the week). Additionally, this category included features capturing other design principles of puzzles, including 'Unusual Symmetry'. This feature captured puzzles deviating from standard rotational symmetry (e.g., those with left-right mirror or diagonal symmetry) that could have had implications for their difficulty.

* 'Circadian' features (n=3) included 'Solve Day Phase', which broke puzzles completion timestamps (obtained per solve via XWStats) into four 6-hour time bins. Per puzzle being predicted, 'IS_per_sdp_avg_past_diff_from_RPB' was a feature that measured how recent puzzle day-specific performance in the pertinent 'Solve Day Phase' compared to 'RPB' across all solve phases. Calculation of this feature used 'SOS-adjustment' (see 'Solver Past Performance' features) in deriving 'RPB'.

* 'Puzzle Day of Week' (n=1) was a class of one ('DOW_num'), simply assigning a number to the puzzle day of week for a given solve.

**<h4>Figure 6. Overview of Solver Past Performance Features Development, and Predictive Features By Class**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/89775946-4c1a-4538-984d-847d5c6ab6e8)

### Machine Learning Regression Modeling 
For the modeling phase, puzzles completed in the first solve period (2018-2019) were removed to minimize the potential negative effects of high baseline performance volatility (see EDA linked in Intro and **Fig. 3**). This reduced the overall sample size to N=1,132. Importantly, as they were generated prior to this filtering, solver 'Past Performance' features included in modeling accrued from the beginning of the solve period (April 2018). Additionally, for the main model 21x21 puzzles (Sun) were also removed from the sample. This resulted in a final modeling 15x15 puzzle N=979. The 21x21 (Sun) puzzles (N=153) were, however, included in by-puzzle-day modeling. 

After predictive features were generated for each puzzle, the best regression model for prediction of the Target Feature (TF) (raw IS2 solve time, in minutes) was found ('Best Model'). To find 'Best Model', 4 different regression models were explored using [scikitlearn (scikit-learn 1.1.1)](https://scikit-learn.org/stable/auto_examples/release_highlights/plot_release_highlights_1_1_0.html): Linear, Random Forest, Gradient Boosting, and HistGradient Boosting. For evaluation of models including all 15x15 puzzles, an 80/20 training/test split (764/192 puzzles) and 5-fold training set cross-validation were used. Additionally, hyperparameter grid search optimization was used per model as warranted (for ex., for Gradient Boosting Regression the grid search was conducted for imputation type, scaler type, learning rate, maximum depth, maximum features, and subsample proportion used for fitting individual base learners). 'Best Model' (ie, lowest RMSE training error when hyperparameter-optimized) was a Gradient Boosting Regression model. See the 'Model Metrics' csv files in the 'Reporting' folder for details, including how this model performed relative to the other models. Also, see **Supplementary Figure 1** for some details on the performance of 'Best Model' (ie, Data Quality Assessment and Feature Importances). 

## Key Modeling Results

**1)** 'Best Model' predicted the TF (raw IS2 solve time, in minutes) more accurately (by 12.5%) than a univariate linear model with puzzle day-specific (PDS) mean IS2 solve time *across the entire sample period* as the sole predictive input ('Mean PDS IST'). 'Best Model' also outperformed (by 34.3%) a variant that simply guessed the mean of the training set TF *across all 15x15 puzzle days*, for each individual puzzle ('Dummy')(**Figure 7**). The 'Best Model' had a mean training error of 8.02 minutes, which corresponded to a 44.3% difference from the training set mean across all 15x15 puzzle days. In contrast, the 'Mean PDS IST' and 'Dummy' benchmark models had mean training errors of 9.45 and 13.03 minutes, respectively (corresponding to 52.2% and 71.9% differences from the training set mean). 

**<h4>Figure 7. Best Model Prediction Quality vs Benchmark Models**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/85b746b4-3c27-4a74-8cd5-de7187421046)
*<h5> 'Best Model' was a Gradient Boosting Regression Model.* 

###
**2)** When individual feature classes or adjustments were systematically subtracted in the modeling stage ('Subtraction Analysis')(**Figure 8**), subtraction of 'Past Performance' features resulted in the largest increase in model error relative to 'Best Model' (4.5%). This increase in model error was *nearly doubled* (8.04%) when the lone 'Puzzle Day' feature ('DOW_num'), constituting the only other overt information regarding puzzle day in the feature set, was *additionally* removed. Subtraction of the 'Answer' features class (3.8%) also resulted in a substantial increase in model error, and removal of the 'Puzzle Day of Week' feature *on its own* resulted in a 1.3% increase in model error. Removal of the 'Circadian' class of features, which were based off of puzzle completion timestamps, also resulted in a 1.3% increase in model error. Each other feature class or adjustment subtracted from 'Best Model' resulted in a <1% increase in model error, with 'Clue' features being the most impactful of these (0.44%). **Fig. 8** shows, in decreasing order of negative impact on model prediction quality, the effect of removing individual feature classes or adjustments (hatched bars) from the full 'Best Model'.    

**<h4>Figure 8. Effect on Model Prediction Quality of Removing Individual Feature Classes or Adjustments from the Best Model**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/29fcf8c2-d984-465b-9d46-272072cd594a)

###
**3)** Because subtraction of 'Past Performance' features resulted in substantial reduction in prediction quality, a subanalysis looked at the impact of removing individual features from this class. 'Past Performance' features included 'SOS-adjusted' IS2 performance on the immediately previous 10 puzzle day-specific puzzles ('Recent Performance Baseline' ['RPB']), standard deviation of 'RPB' ('IS_RPB_l10_stdev'), normalized past performance against the constructor(s) of a given puzzle ('IS Past Perf vs Constr'), and number of past solves on both a puzzle day-specific and non-puzzle day-specific basis ('Prior Solves # - DS' and 'Prior Solves # - NDS', respectively; 'IS Solves l7' was the non-puzzle day specific number of solves in the prior 7 days). The **left panel** of **Figure 9** shows that the majority of the error increase resulting from removal of *all* features in this class (4.5%) resulted from removal of the subset providing information on the number of past solves (3.5%). Subtraction of normalized past performance vs constructor(s) (.79%) and standard deviation of 'RPB' (.72%) also led to measurable increases in model mean training error, though removal of 'RPB' itself had virtually no effect (<.1%). 

Subtraction individually of 6 features from classes other than 'Past Performance' (**Fig. 9**; right panel) that were relatively important to 'Best Model' generation (Gradient Boosting Regression; see **Fig. S1**) had impact on model training error comparable to that of subtraction of the impactful 'Past Performance' features described above. Most negatively impactful on model performance was subtraction of the 'Puzzle Day of Week' feature, which we already saw had a non-linear negative impact on model quality when removed alongside all 'Past Performance' features. Removal of 'Freshness Factor' (and two percentile derivatives) also resulted in a >1% (1.2%) increase in mean training error compared to 'Best Model'. 'Freshness Factor' is a proprietary XWord Info measure that assesses the aggregate relative novelty of all answers in a given crossword puzzle as compared to those in all other crossword puzzles in the NYT archive. 'Answer #' stood out as a 'Grid' feature important to model quality; subtraction resulted in a 1% decrease in model quality. Finally, subtraction of another 'Answer' feature, 'Scrabble Average', also resulted in nearly a 1% increase in model error (.81%). This feature is an indirect assessment of answer rarity, assigning points per square as if the letter in that square were the equivalent Scrabble tile (higher point value = rarer letter). In addition to these 5 features, a number of other non-'Past Performance' features had between a .1-.5% impact on model error with their removal (e.g., the 'Grid' features 'Open Squares #' and 'Average Answer Length', the 'Answer' feature 'Fill-in-the-Blank #' and the 'Circadian' feature 'Completion Hour').  

**<h4>Figure 9. Effect on Model Prediction Quality of Removing Key Individual Features**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/f4d9a5fc-b1e8-4ebc-8207-8f8f1475aceb)

###
**4)** 'Best Model' was discovered on a puzzle day-specific basis ('BPDM'), including for the lone 21x21 puzzle day Sunday (**Figure 10**). Because IS2 mean solve time per puzzle day varied considerably, training errors in **Fig. 10** were normalized to percentage difference from training set mean for that puzzle day. The 'Dummy' model in this puzzle day-specific context is analogous to the 'Mean PDS IST' benchmark model in **Fig. 7**, as the 'Dummy' for all 15x15 puzzles guessed the *overall sample mean* for each puzzle regardless of puzzle day. 

Though the number of puzzles included in the 'BPDMs' (N= 153-177) was much smaller than that in the all 15x15 puzzles model, each still outperformed its particular (not so dumb) 'Dummy'. Sunday (27.4% mean 'BPDM' training error), Monday (27.8%) and Tuesday (29.9%) stood out as the most predictable individual puzzle days, though standard deviations for Monday and Tuesday were large compared to the gap between mean 'BPDM' and 'Dummy' error. There was an overall trend toward increased mean error as puzzle days became more difficult, with Friday and Saturday both having mean error greater >45% each. It is worth noting, however, that despite the much smaller sample size, each 'BPDM' outperformed the all 15x15 puzzle days 'Best Model' (see **Fig. 7** and associated text) on a mean training error as a % of mean solve time basis (50.7% for 'Best Model'). 

**<h4>Figure 10. Best Puzzle Day-Specific Model (BPDM) Prediction Quality**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/4f43d513-d366-4790-8ec6-f9d0f223c55d)
*<h5> 'BPDM' for each day was a Gradient Boosting Regression Model, with hyperparameter optimization specific to that puzzle day. Due to the relatively small number of puzzles in the sample for each puzzle day, a 90/10 training/testing split was used to find each 'BPDM' (range of 153 [Mon] to 177 [Fri] total puzzles per puzzle day). Data Quality Assessments across 'BPDMs' uniformly indicated that model quality continued to increase at 90% training set inclusion.* 


## Discussion

#### *Best Model for all 15x15 puzzles outperformed benchmarks* 
The main result of this study was that the full 'Best Model', incorporating both Individual Solver 2 (IS2) past-performance features as well as numerous features capturing different aspects of individual puzzle grid, clue and answer properties, outperformed several benchmark models. 'Best Model' greatly outperformed (by ~5 minutes on average) a 'Dummy' model, which guessed the total 15x15 puzzles sample mean solve time for each individual puzzle. This is unsurprising, since different puzzle days included in the modeling set had distinct solve time distributions and peaks for IS2. More encouraging that this modeling approach was an effective one is that 'Best Model' also (by ~1.3 minutes on average) outperformed a model which guessed the sample mean of the *specific puzzle day* for each individual puzzle. 

#### *For Best Model, IS2 past performance features were more important than puzzle-specific features*
Another clear finding of this study was that, as a class, 'Past Performance' features had a larger impact on prediction quality than did any other class. Removing this class entirely (subtraction analysis), along with the only other overt encoding of puzzle day in the feature set ('DOW_num'), resulted in a >8% increase in training error (~.8 minutes on average) compared to 'Best Model'. In contrast, removal of no other feature class in isolation resulted in more than a 3.8% increase in training error. In golf terms, the recent playing form of the golfer was much more important than the specific characteristics of the course being played. With that said, it is critical to point out that 'Past Performance' features themselves had puzzle-day specific information built into them. 'RPB' and standard deviation of 'RPB', for example, were specific to the puzzle day of the puzzle being predicted. Thus, because "courses" per puzzle day tended to have similar characteristics, many puzzle features themselves may have been somewhat redundant to information carried by 'RPB'. The inverse was clearly not true, however, as puzzle characteristics did not carry any "memory" of how an individual performed relative to their values in the past.

#### *Recent Performance Baseline (RPB) wasn't critical for obtaining Best Model for IS2*
For both the Global Median Solver (GMS) and IS1, one of the most important aspects to obtaining 'Best Model' was accurately parameterizing puzzle day-specific recent past performance. For IS1, this included discovery of the optimal temporal integration window for averaging past results (previous 8 puzzles), optimizing decay time weighting over those puzzles, and adjusting that feature by difficulty of that same stretch of puzzles for the GMS ('Strength of Schedule' adjustment; 'SOS adjustment'). Though optimal values for these parameters were also discovered and implemented for IS2, in the context of the full 'Best Model' this 'RPB' feature was determined by 'subtraction analysis' to be the least important of the 'Past Performance' class to model quality (<.1%). Furthermore, removal of 'SOS adjustment' from 'RPB' and several other features (e.g., past performance vs constructor(s)) also had only a minimal effect on model quality. In contrast, for IS1 removal of 'RPB' resulted in a *~5%* decrease in model quality and removal of 'SOS' on its own resulted in a ~1.5% decrease in model quality. As stated above, the 'Past Performance' features class was very important to model quality for IS2 as it was for the GMS and IS1, but the contribution of individual features in this class was more distributed for this solver than for the other two solver, with subtraction of individual features ranging from <.1% for 'RPB' up to 3.5% for the feature subclass providing information on both the number of past solves and the 'instantaneous rate' of solving prior to a given solve. A possible explanation for the relative importance of this feature subclass is that this solver had a considerably more variable rate of solving per unit time than the other two solvers.

#### *Though not as important to Best Model quality as past performance features, puzzle-specific features were important for IS2*
Another reason that 'RPB' may have been de-emphasized for IS2 model quality relative to the other solvers is that features of other classes were relatively more important than they were for the other solvers. 'Answer' features (3.8%), 'Puzzle Day of Week' (1.3%), 'Circadian' features (1.3%) and 'Clue' features (.44%) and all made measurable contributions to model quality. These were all larger contributions than any feature class other than 'Past Performance' had on IS1 model quality, and only subtraction of the 'Answer' class impacted GMS performance by more than .5%. At the level of individual puzzle-specific features, the largest impact other than removal of 'Puzzle Day of Week' (1.3%) came from removal of 'Freshness Factor' and its percentile derivatives. These features, proprietary to NYT XWord Info, provide a direct assessment of aggregate answer rarity in a given puzzle under prediction. Subtraction of 'Scrabble Average', a more indirect 'Answer' class measure of answer rarity (answer squares are assigned their according value in Scrabble), had nearly as large a negative impact on model error for IS2 (.81%). Between 'Freshness Factor' and 'Scrabble Average' in terms of model error increase with substraction was 'Answer #', a 'Grid' feature that simply counts the number of answers in a given puzzle. Interestingly, 'Freshness Factor' and derivatives were, by far, the most impactful non-'Past Performance' feature for the GMS (1.1% model training error increase when subtracted), though removal removal of this feature subclass had only a negligible effect on best model quality for IS1.   

In the context of quantity and type of features that were important in 'Best Model' generation for IS2, it is notable that 'Best Model' was a Gradient Boosting Regression (GBR) model. In contrast, performance for the GMS and IS1 were both best modeled with simple Linear Regression models. GBR combines weak learners (e.g., decision trees) in a sequential manner, with each subsequent model focused on correcting errors made by the previous models. The result of this iterative process is effective capture of non-linear relationships. The broader impact of features across classes for IS2 than for the other solvers is very likely deeply related to the optimality of a non-linear model. It is also possible, and not mutually exclusive to this observation, that learning rate was more variable in this solver than in the other two. This would increase the dependence on puzzle-specific features by reducing the accuracy with which 'RPB' was captured by the methods employed in the present study. One possible improvement in this regard would be to discover the best parameters for 'RPB' over relatively small stretches of time, possibly also adjusting by the 'instantaneous solve rate' (as measured by the feature 'IS_solves_l7').

#### *Day-specific models were difficult to interpret, but a larger sample per puzzle day could yield superior model accuracy* 
Unequivocally the most difficult to interpret result of the study was the by-puzzle-day modeling ('BPDM'). The obvious reason for this difficulty was the greatly reduced sample size of each puzzle day relative to the full 15x15 data set employed in generating 'Best Model' (training set sizes of ~150-175 instead of ~760, despite changing the train/test split from 80/20 to 90/10). To this point, relatively large standard deviations can be seen in **Fig. 10**, and these standard deviations were particularly large for the more heterogenous and more difficult later week puzzle days. Also, though Sunday and the early week puzzle days (Mon-Tue) appeared to have distinctly higher quality predictions than the other puzzle days, guessing just based on day-specific sample mean ('Dummy') was also proportionately more accurate for those puzzle days than the others. This implies greater homogeneity of puzzle characteristics for those two days relative to the others that led to generally more clustered solve times. Clearly more data would be a great boon to this approach, a fact backed up even by the Data Quality Analysis for full 'Best Model', where model quality may not have leveled off at the maximum training set size (see **Fig. S1**). 

It is interesting to point out with regard to the 'BPDMs', however, that despite small sample sizes each of these puzzle day-specific models was *more* accurate than the all 15x15 puzzles 'Best Model' in terms of training error as a percentage difference from sample mean. The likely explanation is that, though inadequately powered, each day-specific model contained only puzzles with properties/features highly relevant to those in the puzzle under prediction. On the flip-side, it's likely that the cross-days heterogeneity of puzzles included in the all 15x15 puzzles model added algorithmic challenge even as the increased sample size helped model convergence to a somewhat counterbalancing degree. A reasonable way to "have our cake and eat it too" here would be to have a median-type solver complete many hundreds of puzzles appropriate for individual puzzle days within a reasonably short timeframe (say, months) and then either model them separately or perhaps cluster Mon-Wed in an "easy" model and Thu-Sat in a "hard" model. The main challenge to the first approach would be to accurately model 'RPB' with an unusual rate of solving, and the main challenge to the latter would be that each puzzle day truly does have its own idiosyncrasies (and splitting the sample in half probably doesn't help overcome that).

#### *What's Missing?*
As a percentage of 15x15 puzzle overall sample mean, 'Best Model' for IS2 (50.7% mean training error) was not nearly as accurate as that for the GMS (23.7%). While it is likely that a large part of that discrepancy was due to the ~2x sample size advantage for the GMS, it is equally likely that a substantial amount is also due to the differences in modeling the performance of one individual solver vs modeling a mathematical construct. For one thing, the concept of a "bad day" for the GMS doesn't really exist; if one solver falls off, there's another to take their place smack in the middle of the distribution for that day. While "bad day" is hard to quantify for an individual solver, there are any number of variables that could be measured that might help predict performance variability on a given puzzle. Some examples of what could be plausibly measured consistently and accurately are sleep quality the night prior to a solve, body temperature, heart rate, ambient noise level, degree of caffeination or blood alcohol level, posture while solving, device solving occurred on and lighting conditions. In support of the concept that solve context matters for an individual solver, 'Circadian' features derived from puzzle completion timestamps did indeed have a meaningful impact on 'Best Model' quality. Nonetheless, it is almost certain that the performance of an individual solver will be more subject to noisiness that that of the mathematical construct 'mean solver', and this difference has very real consequences in the building of an accurate model.    

Of the different classes/subclasses of puzzle-specific features, the one I find to be the most lacking from this modeling iteration is unquestionably the 'Clue' class. Features capturing rarity of answers, including 'Freshness Factor', were important for prediction quality for both IS2 and the GMS. What I'd like to find or create myself is an analogous feature to get at the unusualness of words within clues. One can imagine puzzles where the "tough" words are in the clues not in the answers, and this would be missed to at least some extent by the current feature set. Also on the 'Clue' side, simply quantifying 'Average Clue Length' would be quite useful. It stands to reason that the more reading a solver must do to know what to answer, the slower the solve will be. There are also several other types of cross-classification trickiness in puzzle design that would be hard to quantify, but I think it worth a shot at doing so. One of these I call 'Answer Ambiguity', which arises when there is more than one plausible answer in an unfilled piece of grid for a given clue. My strong sense as an experienced solver myself is that this is a wheel spin-inducing art form wielded more by some constructors than others, and more typically on later-week puzzle days. There's also another concept that I call 'Design Isolation', where the details of the construction give a solver fewer "outs" in a tough corner of a puzzle than they'd otherwise have. I don't quite think current features like 'Unusual Symmetry', 'Cheater Square #', or 'Open Squares' are capturing this in a useful way. 

Finally, I'd like to point out that unlike other prediction projects I've carried though (most notably my [men's tennis match prediction project](https://github.com/ursus-maritimus-714/Mens-Tennis-Prediction?tab=readme-ov-file#readme), I have hardly dabbled in the space of "features derivatives"; the taking of "primary" features and combining them in various ways (for ex. creating ratios of puzzle parameters to other puzzle parameters) in hope that one or several just 'click' for a particular model algorithm. That process can be extremely time consuming (the many hundreds of hours I spent on the tennis project attest to that), but also very fruitful and often so in ways that are unexpected going into the modeling stage of a project. 

## Data Supplement

**<h4>Table S1. Features Included in Predictive Modeling**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/14d55287-66f4-4c36-ada4-16a7c990f698)

**<h4>Figure S1. Best Model Metrics**

![image](https://github.com/ursus-maritimus-714/NYT-XWord_Modeling-Individual-Solver-2/assets/90933302/dcfd762c-1fb4-45f2-b732-6b246ecebacf)
*<h5> The Gradient Boosting Regression Model yielded 'Best Model', out of those tested. See the Model Metrics file in Reporting folder for full details. Panel A shows a Data Quality Assessment for 'Best Model'. It appears that model quality may have been continuing to gradually improve at the number of samples used in the training set (n=~760). Panel B shows rank-ordered feature importances for 'Best Model'. See Table S1 above for descriptions of these features, and also for those not selected.*
