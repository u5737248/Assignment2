# Assignment2
 install.packages("pacman", repos = "http://cran.us.r-project.org")
library(pacman)

p_load(bookdown, tidyverse, ggforce, flextable, latex2exp, png, magick,metafor) 
Question 1: Correct analysis of Clark et al. (2020) data (i.e., OA_activitydat_20190302_BIOL3207.csv) to generate the summary statistics (means, SD, N) for each of the fish species’ average activity for each treatment.

#Upload the OA_activitydat_20190302_BIOL3207.csv dataset

path <- "C:/Users/Rain/Desktop/Biol3207/Workshop2/raw_data/OA_activitydat_20190302_BIOL3207.csv"
  
data <- read_csv(path)
## New names:
## Rows: 589 Columns: 9
## ── Column specification
## ──────────────────────────────────────────────────────── Delimiter: "," chr
## (5): loc, species, treatment, size, comment dbl (4): ...1, animal_id, sl,
## activity
## ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
## Specify the column types or set `show_col_types = FALSE` to quiet this message.
## • `` -> `...1`
#generate the summary statistics

library(tidyverse)
OA <- data %>% filter(!(is.na(activity))) 
OA_data <- OA %>% select(species, treatment, animal_id, activity)
library(tidyverse)
OA_data %>% count(species,treatment)
OA_data1 <- OA_data %>% filter(species !="whitedams") 
OA_new <- OA_data1 %>%group_by(species, treatment)%>%summarise(mean=mean(activity), sd=sd(activity), number=length(activity))
ft <- flextable(OA_new)
ft <- theme_vanilla(ft)
ft <- color(ft, part = "body",color = "#666666")
ft <- set_caption(ft, caption = "Mean, Standard Error And The Sample Sizes")
ft
#change the horizontal dataset to a vertical dataset for better merge the later dataset

OA_new2 = data.frame(species =c("acantho","ambon","chromis","humbug","lemon"),
                     ctrl.n = c(99, 21,20,71,30),
                     ctrl.mean = c(28, 17, 26,36,20),
                     ctrl.sd = c(11.7, 11.5, 9.1,13.8,12.3),
                     oa.n = c(95,22,14,69,19),
                     oa.mean = c(26,18,25,40,26),
                     oa.sd = c(10.8,12.2,11.0,12.2,11.0))
Question 2: Correct analysis of Clark et al. (2020) data (i.e., OA_activitydat_20190302_BIOL3207.csv) to generate the summary statistics (means, SD, N) for each of the fish species’ average activity for each treatment.

#upload the Clark et al. dataset

path2 <- "C:/Users/Rain/Desktop/Biol3207/Workshop2/raw_data/clark_paper_data.csv"
  
data_new <- read_csv(path2)
## Rows: 1 Columns: 15
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (9): Study, Authors, Title, Journal, Effect type, Climate (FishBase), En...
## dbl (5): Year (online), Year (print), Pub year IF, 2017 IF, Average_n
## lgl (1): Cue/stimulus type
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#merge two new dataset together

total<-merge.data.frame(data_new,OA_new2)
Question 3: Through coding, correctly merge the combined summary statistics and metadata from Clark et al. (2020) (output from 1 & 2) into the larger meta-analysis dataset (i.e., ocean_meta_data.csv).

#upload the ocean_meta_data.csv

path3 <- "C:/Users/Rain/Desktop/Biol3207/Workshop2/raw_data/ocean_meta_data.csv"
  
data_new2 <- read_csv(path3)
## Rows: 818 Columns: 22
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (13): Study, Authors, Title, Journal, Pub year IF, 2017 IF, Effect type,...
## dbl  (9): Year (online), Year (print), Average_n, ctrl.n, ctrl.mean, ctrl.sd...
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#merge all together

data_new2$`Pub year IF`<-as.numeric(data_new2$`Pub year IF`)
## Warning: NAs introduced by coercion
data_new2$`2017 IF`<-as.numeric(data_new2$`2017 IF`)
## Warning: NAs introduced by coercion
total_new<-dplyr::bind_rows(total,data_new2)

total_new <- total_new %>%  mutate(residual = 1:n())
Question 4: Correctly calculate the log response ratio (lnRR) effect size for every row of the dataframe using metafor’s escalc() function.

lnRR <- escalc(measure="ROM",n1i=ctrl.n, n2i=oa.n,m1i=ctrl.mean, m2i=oa.mean, sd1i=ctrl.sd, sd2i=oa.sd,data=total_new,var.names=c("lnRR","V_lnRR"))
## Warning in log(m1i/m2i): NaNs produced
Question 5: Correct meta-analytic model fitted to the data that controls for the sampling variance of lnRR. The model should include a random effect of study and observation. Use metafor’s rma.mv() function.

MLMA<-rma.mv( lnRR~1, V = V_lnRR, random=list(~1|Study, ~1 |species,~1|residual), test = "t", dfs = "contain", data=lnRR)
## Warning: Rows with NAs omitted from model fitting.
## Warning: Ratio of largest to smallest sampling variance extremely large. May not
## be able to obtain stable results.
MLMA
## 
## Multivariate Meta-Analysis Model (k = 800; method: REML)
## 
## Variance Components:
## 
##             estim    sqrt  nlvls  fixed    factor 
## sigma^2.1  0.2443  0.4942     92     no     Study 
## sigma^2.2  0.2131  0.4616     61     no   species 
## sigma^2.3  3.9263  1.9815    800     no  residual 
## 
## Test for Heterogeneity:
## Q(df = 799) = 736088977.4985, p-val < .0001
## 
## Model Results:
## 
## estimate      se     tval  df    pval    ci.lb   ci.ub    
##  -0.0613  0.1231  -0.4984  60  0.6200  -0.3075  0.1849    
## 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
Question 6: Written paragraph of the findings and what they mean which is supported with a figure.

#Correct presentation and interpretation of overall meta-analytic mean and measures of uncertainty around the mean estimate (e.g., 95% confidence intervals).

predict(MLMA, transf = "transf.ztor")
## 
##     pred     se   ci.lb  ci.ub   pi.lb  pi.ub 
##  -0.0613 0.1231 -0.3075 0.1849 -4.2566 4.1339
##To know the overall metanalytic mean effect size across the studies actually is estimated to be, here we use the predict.rma function to predicted values for objects of class “rma”. “pred” means predicted values is -0.0613 and that the mean of lnRR value is negative. “se” is corresponding standard error(s). “ci.lb” is lower bound of the 95% confidence interval(s) and “ci.ub” is upper bound of the 95% confidence interval(s). So the 95% confidence intervals which range from -0.3075 to 0.1849. Our mean is -0.0613 and is between this -0.3075 to 0.1849 95% confidence intervals, so 95% of the confidence intervals constructed would contain the true meta-analytic mean or 95% of the time we would expect the true mean to fall between lnRR values of 0.029 to 0.167.

#Measures of heterogeneity in effect size estimates across studies (i.e., I2 and/or prediction intervals - see predict() function in metafor)

i2_vals<-orchaRd::i2_ml(MLMA)
i2 <- tibble(type = orchaRd::firstup(gsub("I2_", "", names(i2_vals))), I2 = i2_vals)
flextable(i2) %>%
    align(part = "header", align = "center") %>%
    compose(part = "header", j = 1, value = as_paragraph(as_b("Type"))) %>%
    compose(part = "header", j = 2, value = as_paragraph(as_b("I"), as_b(as_sup("2")),
        as_b("(%)")))
Type

I2(%)

Total

100.0

Study

5.6

Species

4.9

Residual

89.6

##To know the extent to which effects vary within and across studies needs to be considered when interpreting meta-analytic mean estimates, we measured the ‘heterogeneity’ also known as the variability. Here we use the i2_ml function to measure the variability from mutilevel. From all three level , the total sampling variation is 100%. From the study level, only 5.6% of the total variation in effect size estimates is the result of differences between studies.From the species level, only 4.9% of the total variation in effect size estimates is the result of differences between studies. From the residual level, the variability is 89.6 for random effect. We have high heterogeneous effect size data because sampling variation not contributes to the total variation in effects.

#Forest plot showing the mean estimate, 95% confidence interval, and prediction interval with clearly labelled axes, number of samples and studies plotted on figure

orchaRd::orchard_plot(MLMA, mod = 1, group = "Study", data = lnRR,
    xlab = "log response ratio (lnRR)", angle = 45)+labs(caption = "mean= -0.0613,
95% confidence interval=-0.3075 to 0.1849,
i2=100%")
 #Forest plot also known as orchard plot, showing the mean lnRR for effect size estimated between study. k = the number of effect sizes and the number of studies are in brackets.

Question 7: Funnel plot for visually assessing the possibility of publication bias.

funnel(x = lnRR$lnRR, vi = lnRR$V_lnRR, yaxis = "seinv",xlim=c(-2,2),ylim=c(0.1,100), digits=2,level = c(0.1, 0.05,0.01), shade = c("white","gray25" ,"gray55", "gray 75"),
    las = 1, xlab = "Correlation Coefficient (r)", atransf = tanh, legend = TRUE, col="black")
 Question 8: Time-lag plot assessing how effect sizes may or may not have changed through time.

ggplot(lnRR, aes(y = lnRR, x = Year..print., size = 1/sqrt(V_lnRR))) + geom_point(alpha = 0.3) +
    geom_smooth(method = lm, col = "purple", show.legend = FALSE) + labs(x = "Publication Year",
    y = "Fisher's Log Response Ratio Correlation Coefficient (lnRR)", size = "Precision (1/SE)") +
    theme_classic() 
## `geom_smooth()` using formula 'y ~ x'
## Warning: Removed 23 rows containing non-finite values (stat_smooth).
## Warning: Removed 23 rows containing missing values (geom_point).
 Question 9: Formal meta-regression model that includes year as a moderator (fixed effect) to test for time-lag bias

metareg <- rma.mv(lnRR ~ Year..print., V = V_lnRR, random = list(~1 | Study,~1 | species,~1 | residual),
    test = "t", dfs = "contain", data = lnRR)
## Warning: Rows with NAs omitted from model fitting.
## Warning: Ratio of largest to smallest sampling variance extremely large. May not
## be able to obtain stable results.
summary(metareg)
## 
## Multivariate Meta-Analysis Model (k = 800; method: REML)
## 
##     logLik    Deviance         AIC         BIC        AICc   
## -1781.7747   3563.5494   3573.5494   3596.9599   3573.6251   
## 
## Variance Components:
## 
##             estim    sqrt  nlvls  fixed    factor 
## sigma^2.1  0.1747  0.4179     92     no     Study 
## sigma^2.2  0.2029  0.4505     61     no   species 
## sigma^2.3  3.9365  1.9841    800     no  residual 
## 
## Test for Residual Heterogeneity:
## QE(df = 798) = 161794132.4225, p-val < .0001
## 
## Test of Moderators (coefficient 2):
## F(df1 = 1, df2 = 90) = 7.6068, p-val = 0.0070
## 
## Model Results:
## 
##                estimate       se     tval  df    pval      ci.lb     ci.ub     
## intrcpt       -224.8029  81.4837  -2.7589  59  0.0077  -387.8513  -61.7544  ** 
## Year..print.     0.1115   0.0404   2.7580  90  0.0070     0.0312    0.1918  ** 
## 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
Question 10: Formal meta-regression model that includes inverse sampling variance (i.e., 1vlnRR) to test for file-drawer biases

metareg_new <- rma.mv(lnRR ~ (1 / V_lnRR), V = V_lnRR, random = list(~1 | Study, ~1 |species,~1 | residual), test = "t", dfs = "contain", data = lnRR)
## Warning: Rows with NAs omitted from model fitting.
## Warning: Ratio of largest to smallest sampling variance extremely large. May not
## be able to obtain stable results.
summary(metareg_new)
## 
## Multivariate Meta-Analysis Model (k = 800; method: REML)
## 
##     logLik    Deviance         AIC         BIC        AICc   
## -1787.1969   3574.3938   3582.3938   3601.1273   3582.4442   
## 
## Variance Components:
## 
##             estim    sqrt  nlvls  fixed    factor 
## sigma^2.1  0.2443  0.4942     92     no     Study 
## sigma^2.2  0.2131  0.4616     61     no   species 
## sigma^2.3  3.9263  1.9815    800     no  residual 
## 
## Test for Heterogeneity:
## Q(df = 799) = 736088977.4985, p-val < .0001
## 
## Model Results:
## 
## estimate      se     tval  df    pval    ci.lb   ci.ub    
##  -0.0613  0.1231  -0.4984  60  0.6200  -0.3075  0.1849    
## 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
time <- orchaRd::r2_ml(metareg_new)  
time
##    R2_marginal R2_conditional 
##            0.0            0.1
11.A written paragraph that discusses the potential for publication bias based on the meta-regression results. What type of publication bias, if any, appears to be present in the data? If publication bias is present, what does it mean and what might be contributing to such bias? ##Not only publication bias, also have citation bias, time-lag bias, mutiple publication bias, language bias and outcome reporting bias also can distort the evidence obtain from the meta-analysis. We test about the time-lag bias in the data. Publication bias refers to the fact that statistically significant study results are more likely to be reported and published than non-statistically significant and invalid results. When researchers, reviewers, or editors select papers for publication, the reliance on the direction and strength of findings creates a bias so that the publication process is not a random event, thus inhibiting the publication of certain studies. Funnel plots, the most commonly used method for identifying publication bias in meta-analyses, reflect estimates of intervention effects for individual studies at a given sample size or study precision. When conducting a meta-analysis, the presence of bias must be tested. However, publication bias is difficult to avoid, and for its identification, the most commonly used method is the funnel plot method, which is a funnel plot expression that can detect the presence of bias more intuitively. Theoretically, the point estimates of the independent studies included in the Meta-analysis should be set in the plane coordinate system in the shape of an inverted funnel, so it is called funnel plot. The sample size was small, the research accuracy was low, and the distribution was scattered at the bottom of the funnel plot. Large sample sizes and high study precision were distributed at the top of the funnel plot and concentrated toward the middle. As can be seen from the funnel plots drawn in our study, our funnel plots are asymmetric and the asymmetry is very pronounced, indicating that there is a bias that may overestimate the treatment effect. But publication bias is not the only cause of funnel plot asymmetry. We have large heterogeneity (I^2), which may be responsible for the funnel plot asymmetry. However, looking at the white area of the funnel plot, there is an insignificant area where there are many points, indicating that there are many articles that have not been published, suggesting that there may be publication bias. Looking at the meta-regression plots we have drawn, we find that there does seem to be a significant positive correlation between the difference between the mean and year for the experimental and control groups. In addition, the sampling variance of the earlier studies was lower, and the effect of these earlier studies appeared to be lower compared with studies conducted in later years. ##For another time-lag bias, we use the Time-lag plot. Studies with significant results were generally published earlier than studies with adverse results. This means that the results of recently conducted studies usually already have positive results, whereas those with non-significant results do not.In the time-lag plot, we can see points are scaled in relation to their precision. Small points indicate effects with low precision or high sampling variance.From the graph, the relationship between average effect size and the year of publication is positive.R2_marginal is equal to 0.00 so time-lag explain 0% of the variation. So there is no time lag bias.

12.Identify any studies contributing to publication bias. How do your updated meta-analysis results compare with a meta-analysis by Clement et. al. (2022)? Are there any concerns about these studies? If so, describe using references to existing papers what concerns have been raised? #The results of our meta-analysis have similarities and differences with those of Clement et al. (2022).Although the positive correlations I derived for time lag bias are inconsistent with the literature, they all have the potential for time lag bias to occur. It is clear from my meta-analysis that the mean difference analysis of the earlier treatment group has higher precision compared to the mean group, which suggests that the current mean difference analysis is not precise enough and may have some effect on the experimental results influences. Meta-analysis faces many challenges. A major challenge is that the quality of the results of the meta-analyses was not superior to the studies used. The second challenge is publishing results that are biased towards supportive (often statistically significant) results. If a supporting study is published and a non-significant finding is not published, the results of the meta-analysis will show upward bias. The third problem arises from the fact that there is only a small amount of research in a field of study. Just as any study can gain greater credibility from a larger sample, a meta-analysis can yield more meaningful results when it includes more studies, representing more participants. Finally, there is the question of how to deal with heterogeneous effects. If heterogeneity cannot be resolved by mediation analysis, we doubt that mean results can be meaningfully interpreted (ref:Levine, T. R. et al. (2009) Sample Sizes and Effect Sizes are Negatively Correlated in Meta-Analyses: Evidence and Implications of a Publication Bias Against NonSignificant Findings. Communication monographs. [Online] 76 (3), 286–302.)


