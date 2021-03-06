Improving memory for and production of singular <i>they</i> pronouns:
Experiment 2
================
Bethany Gardner
03/24/2022

-   [Load data](#load-data)
-   [Memory](#memory)
    -   [Descriptive Stats](#descriptive-stats)
    -   [Model](#model)
-   [Production](#production)
    -   [Descriptive Stats](#descriptive-stats-1)
    -   [Model](#model-1)
    -   [Three-Way Interaction](#three-way-interaction)
    -   [Producing they/them at least
        once](#producing-theythem-at-least-once)
-   [Memory Predicting Production](#memory-predicting-production)
    -   [Descriptive Stats](#descriptive-stats-2)
    -   [Model](#model-2)

# Load data

Read data, preprocessed from PCIbex output. See data/exp2_data_readme
for more details.

``` r
d_all <- read.csv("../data/exp2_data.csv", stringsAsFactors=TRUE) %>%
  rename("Biographies"="Story") #rename to match labeling in paper

d_all$Participant <- as.factor(d_all$Participant)
d_all$PSA <- as.factor(d_all$PSA)
d_all$Biographies <- as.factor(d_all$Biographies)
d_all$X <- NULL

str(d_all)
```

    ## 'data.frame':    11520 obs. of  18 variables:
    ##  $ Participant: Factor w/ 320 levels "1","2","3","4",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ SubjAge    : int  53 53 53 53 53 53 53 53 53 53 ...
    ##  $ SubjEnglish: Factor w/ 4 levels "Fully competent in speaking, listening, reading, and writing, but not native",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ SubjGender : Factor w/ 2 levels "female","male": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Condition  : Factor w/ 4 levels "both","neither",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ List       : Factor w/ 12 levels "both_1","both_2",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ PSA        : Factor w/ 2 levels "0","1": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Biographies: Factor w/ 2 levels "0","1": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Name       : Factor w/ 12 levels "Amanda","Andrew",..: 2 2 7 2 7 1 1 7 6 6 ...
    ##  $ Job        : Factor w/ 12 levels "accountant","doctor",..: 11 11 4 11 4 3 3 4 6 6 ...
    ##  $ Pet        : Factor w/ 3 levels "cat","dog","fish": 2 2 3 2 3 1 1 3 2 2 ...
    ##  $ Pronoun    : Factor w/ 3 levels "he/him","she/her",..: 1 1 2 1 2 2 2 2 1 1 ...
    ##  $ M_Type     : Factor w/ 3 levels "job","pet","pronoun": 2 3 1 1 3 1 2 2 1 2 ...
    ##  $ M_Response : Factor w/ 18 levels "accountant","cat",..: 4 8 11 10 15 9 6 2 5 4 ...
    ##  $ M_Acc      : int  1 1 0 0 1 0 0 0 0 1 ...
    ##  $ P_Text     : Factor w/ 2918 levels " after the work he  play with the kids",..: NA 2625 NA NA 2172 NA NA NA NA NA ...
    ##  $ P_Response : Factor w/ 4 levels "he/him","none",..: NA 4 NA NA 3 NA NA NA NA NA ...
    ##  $ P_Acc      : int  NA 0 NA NA 1 NA NA NA NA NA ...

Set up contrast coding for Pronoun Type. The first contrast compares
they to he+she. The second contrast compares he to she.

``` r
contrasts(d_all$Pronoun)=cbind("_T_HS"=c(.33,.33,-.66), 
                               "_H_S"=c(-.5,.5, 0))
contrasts(d_all$Pronoun)
```

    ##           _T_HS _H_S
    ## he/him     0.33 -0.5
    ## she/her    0.33  0.5
    ## they/them -0.66  0.0

Set up contrast coding for PSA and Biographies conditions. .5 are the
conditions related to singular they (gendered language PSA, they/them
biographies); -.5 are the unrelated conditions (unrelated PSA, he/him
and she/her biographies).

``` r
contrasts(d_all$PSA)=cbind("_GenLang"=c(-.5,.5))
contrasts(d_all$PSA)
```

    ##   _GenLang
    ## 0     -0.5
    ## 1      0.5

``` r
contrasts(d_all$Biographies)=cbind("_They"=c(-.5,.5))
contrasts(d_all$Biographies)
```

    ##   _They
    ## 0  -0.5
    ## 1   0.5

Remove pet and job rows, and the columns that aren???t used in the models.

``` r
d <- d_all %>% filter (M_Type=="pronoun") %>%
  select(Participant, Condition, PSA, Biographies, Name, Pronoun, 
         M_Acc, M_Response, P_Acc, P_Response)
```

By-participant mean accuracy for memory and production tasks.

``` r
d_subj <- d %>% group_by(PSA, Biographies, Participant, Pronoun) %>%
          summarize(M_Mean=mean(M_Acc), P_Mean=mean(P_Acc))
```

By-participant mean accuracy for production, split by memory accuracy.

``` r
d_acc <- d %>% group_by(PSA, Biographies, Participant, Pronoun, M_Acc) %>%
          summarize(P_Mean=mean(P_Acc))
```

If each participant selected/produced they/them at least once.

``` r
d_they <- d %>% 
  mutate(M_IsThey=ifelse(M_Response=="they/them", 1, 0)) %>%
  mutate(P_IsThey=ifelse(P_Response=="they/them", 1, 0)) %>%
  group_by(Participant, Condition, PSA, Biographies) %>%
  summarize(M_Count=sum(M_IsThey),
            P_Count=sum(P_IsThey)) %>%
  mutate(M_UseThey=ifelse(M_Count!=0, 1, 0)) %>%
  mutate(P_UseThey=ifelse(P_Count!=0, 1, 0))
```

# Memory

### Descriptive Stats

Mean accuracy for all three memory question types.

``` r
d_all %>% group_by(M_Type) %>%
          summarise(acc=mean(M_Acc))
```

    ## # A tibble: 3 x 2
    ##   M_Type    acc
    ##   <fct>   <dbl>
    ## 1 job     0.365
    ## 2 pet     0.540
    ## 3 pronoun 0.716

Mean accuracy, split by Pronoun Type, PSA, and Biographies conditions.
\[Both = gendered language PSA + they biographies; PSA = gendered
language PSA + he/she biographies; Story = unrelated PSA + they
biographies; Neither = unrelated PSA + he/she biographies.\]

``` r
d %>% group_by(Pronoun, Condition) %>%
      summarise(acc=mean(M_Acc))    
```

    ## # A tibble: 12 x 3
    ## # Groups:   Pronoun [3]
    ##    Pronoun   Condition   acc
    ##    <fct>     <fct>     <dbl>
    ##  1 he/him    both      0.75 
    ##  2 he/him    neither   0.819
    ##  3 he/him    psa       0.8  
    ##  4 he/him    story     0.784
    ##  5 she/her   both      0.788
    ##  6 she/her   neither   0.825
    ##  7 she/her   psa       0.831
    ##  8 she/her   story     0.806
    ##  9 they/them both      0.572
    ## 10 they/them neither   0.525
    ## 11 they/them psa       0.588
    ## 12 they/them story     0.506

90-95% of participants selected they/them at least once.

``` r
d_they %>% group_by(Condition) %>% 
  summarize(Select_They=sum(M_UseThey)) %>%
  mutate(n=80) %>%
  mutate(prop=Select_They/n)
```

    ## # A tibble: 4 x 4
    ##   Condition Select_They     n  prop
    ##   <fct>           <dbl> <dbl> <dbl>
    ## 1 both               76    80 0.95 
    ## 2 neither            72    80 0.9  
    ## 3 psa                73    80 0.912
    ## 4 story              74    80 0.925

### Model

Full model has interactions between Pronoun (2 contrasts), PSA, and
Biographies; random intercepts and slopes by participant and item.
buildmer finds the maximal model that will converge (but doesn???t then go
backward to remove non-significant terms, the default setting). The
final model includes all fixed effects/interactions and random
intercepts by name.

``` r
model_m_full <- M_Acc ~ Pronoun * PSA * Biographies + 
                (Pronoun|Participant) + (Pronoun|Name)

model_m <- buildmer(model_m_full, d, 
           family="binomial", direction=c("order"))
```

    ## Determining predictor order

    ## Fitting via glm: M_Acc ~ 1

    ## Currently evaluating LRT for: Biographies, Pronoun, PSA

    ## Fitting via glm: M_Acc ~ 1 + Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun

    ## Fitting via glm: M_Acc ~ 1 + PSA

    ## Updating formula: M_Acc ~ 1 + Pronoun

    ## Currently evaluating LRT for: Biographies, PSA

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + PSA

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies

    ## Currently evaluating LRT for: Pronoun:Biographies, PSA

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies +
    ##     Pronoun:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies + PSA

    ## Currently evaluating LRT for: Pronoun:Biographies, Pronoun:PSA,
    ##     PSA:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA +
    ##     Pronoun:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA +
    ##     PSA:Biographies

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA

    ## Currently evaluating LRT for: Pronoun:Biographies, PSA:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + PSA:Biographies

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies

    ## Currently evaluating LRT for: PSA:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies + PSA:Biographies

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies + PSA:Biographies

    ## Currently evaluating LRT for: Pronoun:PSA:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies + Biographies:PSA + Pronoun:PSA:Biographies

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies + Biographies:PSA + Pronoun:PSA:Biographies

    ## Fitting via glm: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies + Biographies:PSA + Pronoun:PSA:Biographies

    ## Currently evaluating LRT for: 1 | Name, 1 | Participant

    ## Fitting via glmer, with ML: M_Acc ~ 1 + Pronoun + Biographies + PSA +
    ##     Pronoun:PSA + Pronoun:Biographies + Biographies:PSA +
    ##     Pronoun:Biographies:PSA + (1 | Name)

    ## Fitting via glmer, with ML: M_Acc ~ 1 + Pronoun + Biographies + PSA +
    ##     Pronoun:PSA + Pronoun:Biographies + Biographies:PSA +
    ##     Pronoun:Biographies:PSA + (1 | Participant)

    ## Updating formula: M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA
    ##     + Pronoun:Biographies + Biographies:PSA + Pronoun:Biographies:PSA +
    ##     (1 | Name)

    ## Currently evaluating LRT for: Pronoun | Name, 1 | Participant

    ## Fitting via glmer, with ML: M_Acc ~ 1 + Pronoun + Biographies + PSA +
    ##     Pronoun:PSA + Pronoun:Biographies + Biographies:PSA +
    ##     Pronoun:Biographies:PSA + (1 + Pronoun | Name)

    ## boundary (singular) fit: see help('isSingular')

    ## Fitting via glmer, with ML: M_Acc ~ 1 + Pronoun + Biographies + PSA +
    ##     Pronoun:PSA + Pronoun:Biographies + Biographies:PSA +
    ##     Pronoun:Biographies:PSA + (1 | Name) + (1 | Participant)

    ## Ending the ordering procedure due to having reached the maximal
    ##     feasible model - all higher models failed to converge. The types of
    ##     convergence failure are: Singular fit lme4 reports not having
    ##     converged (-1)

``` r
summary(model_m)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) (p-values based on Wald z-scores) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: 
    ## M_Acc ~ 1 + Pronoun + Biographies + PSA + Pronoun:PSA + Pronoun:Biographies +  
    ##     Biographies:PSA + Pronoun:Biographies:PSA + (1 | Name)
    ##    Data: d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   4331.6   4412.9  -2152.8   4305.6     3827 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -2.2977 -1.0257  0.4830  0.5489  1.0425 
    ## 
    ## Random effects:
    ##  Groups Name        Variance Std.Dev.
    ##  Name   (Intercept) 0.008349 0.09137 
    ## Number of obs: 3840, groups:  Name, 12
    ## 
    ## Fixed effects:
    ##                                           Estimate Std. Error  z value Pr(>|z|)
    ## (Intercept)                                0.99624    0.04641 21.46737    0.000
    ## Pronoun_T_HS                               1.21757    0.07599 16.02350    0.000
    ## Pronoun_H_S                                0.14014    0.11078  1.26496    0.206
    ## Biographies_They                          -0.17570    0.07625 -2.30426    0.021
    ## PSA_GenLang                                0.02205    0.07625  0.28920    0.772
    ## Pronoun_T_HS:PSA_GenLang                  -0.36169    0.15193 -2.38062    0.017
    ## Pronoun_H_S:PSA_GenLang                    0.12121    0.19910  0.60879    0.543
    ## Pronoun_T_HS:Biographies_They             -0.16107    0.15192 -1.06024    0.289
    ## Pronoun_H_S:Biographies_They               0.04874    0.19911  0.24476    0.807
    ## Biographies_They:PSA_GenLang              -0.07341    0.15251 -0.48136    0.630
    ## Pronoun_T_HS:Biographies_They:PSA_GenLang -0.13107    0.30389 -0.43131    0.666
    ## Pronoun_H_S:Biographies_They:PSA_GenLang  -0.08755    0.39833 -0.21980    0.826
    ##                                           Pr(>|t|)    
    ## (Intercept)                                 <2e-16 ***
    ## Pronoun_T_HS                                <2e-16 ***
    ## Pronoun_H_S                                 0.2059    
    ## Biographies_They                            0.0212 *  
    ## PSA_GenLang                                 0.7724    
    ## Pronoun_T_HS:PSA_GenLang                    0.0173 *  
    ## Pronoun_H_S:PSA_GenLang                     0.5427    
    ## Pronoun_T_HS:Biographies_They               0.2890    
    ## Pronoun_H_S:Biographies_They                0.8066    
    ## Biographies_They:PSA_GenLang                0.6303    
    ## Pronoun_T_HS:Biographies_They:PSA_GenLang   0.6662    
    ## Pronoun_H_S:Biographies_They:PSA_GenLang    0.8260    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr) Pr_T_HS Pr_H_S Bgrp_T PSA_GL P_T_HS:P P_H_S:P Pr_T_HS:B_T
    ## Pronon_T_HS  0.171                                                          
    ## Pronoun_H_S  0.025  0.024                                                   
    ## Bigrphs_Thy -0.043 -0.038   0.001                                           
    ## PSA_GenLang -0.013 -0.019   0.013 -0.011                                    
    ## P_T_HS:PSA_ -0.016 -0.004   0.011 -0.007  0.207                             
    ## P_H_S:PSA_G  0.012  0.011  -0.023 -0.009  0.038  0.029                      
    ## Pr_T_HS:B_T -0.031 -0.031   0.001  0.207 -0.007 -0.007   -0.007             
    ## Prn_H_S:B_T  0.001  0.001  -0.061  0.038 -0.009 -0.007   -0.014   0.029     
    ## Bg_T:PSA_GL -0.009 -0.007  -0.008 -0.016 -0.052 -0.038    0.001  -0.019     
    ## P_T_HS:B_T: -0.007 -0.007  -0.006 -0.019 -0.038 -0.031    0.001  -0.004     
    ## P_H_S:B_T:P -0.007 -0.007  -0.013  0.014  0.001  0.001   -0.067   0.011     
    ##             Pr_H_S:B_T B_T:PS P_T_HS:B_T:
    ## Pronon_T_HS                              
    ## Pronoun_H_S                              
    ## Bigrphs_Thy                              
    ## PSA_GenLang                              
    ## P_T_HS:PSA_                              
    ## P_H_S:PSA_G                              
    ## Pr_T_HS:B_T                              
    ## Prn_H_S:B_T                              
    ## Bg_T:PSA_GL  0.014                       
    ## P_T_HS:B_T:  0.011      0.207            
    ## P_H_S:B_T:P -0.025      0.038  0.029

# Production

### Descriptive Stats

Mean accuracy, split by Pronoun Type, PSA, and Biographies conditions.
\[Both = gendered language PSA + they biographies; PSA = gendered
language PSA + he/she biographies; Story = unrelated PSA + they
biographies; Neither = unrelated PSA + he/she biographies.\]

``` r
d %>% group_by(Pronoun, Condition) %>%
      summarise(m=mean(P_Acc))    
```

    ## # A tibble: 12 x 3
    ## # Groups:   Pronoun [3]
    ##    Pronoun   Condition     m
    ##    <fct>     <fct>     <dbl>
    ##  1 he/him    both      0.834
    ##  2 he/him    neither   0.919
    ##  3 he/him    psa       0.809
    ##  4 he/him    story     0.875
    ##  5 she/her   both      0.828
    ##  6 she/her   neither   0.884
    ##  7 she/her   psa       0.778
    ##  8 she/her   story     0.828
    ##  9 they/them both      0.328
    ## 10 they/them neither   0.106
    ## 11 they/them psa       0.334
    ## 12 they/them story     0.119

### Model

Same model specifications as before. The maximal model contains all
fixed effects/interactions and by-item random intercepts.

``` r
model_p_full <- P_Acc ~ Pronoun * PSA * Biographies + 
                (Pronoun|Participant) + (Pronoun|Name)

model_p <- buildmer(model_p_full, d, 
           family="binomial", direction=c("order"))
```

    ## Determining predictor order

    ## Fitting via glm: P_Acc ~ 1

    ## Currently evaluating LRT for: Biographies, Pronoun, PSA

    ## Fitting via glm: P_Acc ~ 1 + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun

    ## Fitting via glm: P_Acc ~ 1 + PSA

    ## Updating formula: P_Acc ~ 1 + Pronoun

    ## Currently evaluating LRT for: Biographies, PSA

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA

    ## Currently evaluating LRT for: Biographies, Pronoun:PSA

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA

    ## Currently evaluating LRT for: Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies

    ## Currently evaluating LRT for: Pronoun:Biographies, PSA:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + Pronoun:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies

    ## Currently evaluating LRT for: Pronoun:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies + Pronoun:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies + Pronoun:Biographies

    ## Currently evaluating LRT for: Pronoun:PSA:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies + Pronoun:Biographies + Pronoun:PSA:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies + Pronoun:Biographies + Pronoun:PSA:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies + Pronoun:Biographies + Pronoun:PSA:Biographies

    ## Currently evaluating LRT for: 1 | Name, 1 | Participant

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA +
    ##     Biographies + PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + (1 | Name)

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA +
    ##     Biographies + PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + (1 | Participant)

    ## Updating formula: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies
    ##     + PSA:Biographies + Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     (1 | Name)

    ## Currently evaluating LRT for: Pronoun | Name, 1 | Participant

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA +
    ##     Biographies + PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + (1 + Pronoun | Name)

    ## boundary (singular) fit: see help('isSingular')

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA +
    ##     Biographies + PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + (1 | Name) + (1 | Participant)

    ## Ending the ordering procedure due to having reached the maximal
    ##     feasible model - all higher models failed to converge. The types of
    ##     convergence failure are: Singular fit lme4 reports not having
    ##     converged (-1)

``` r
summary(model_p)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) (p-values based on Wald z-scores) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: 
    ## P_Acc ~ 1 + Pronoun + PSA + Pronoun:PSA + Biographies + PSA:Biographies +  
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies + (1 | Name)
    ##    Data: d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   3464.0   3545.3  -1719.0   3438.0     3827 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -3.4309 -0.3672  0.3616  0.4550  3.0688 
    ## 
    ## Random effects:
    ##  Groups Name        Variance Std.Dev.
    ##  Name   (Intercept) 0.007172 0.08469 
    ## Number of obs: 3840, groups:  Name, 12
    ## 
    ## Fixed effects:
    ##                                           Estimate Std. Error  z value Pr(>|z|)
    ## (Intercept)                                0.69508    0.05161 13.46873    0.000
    ## Pronoun_T_HS                               3.15560    0.09577 32.95093    0.000
    ## Pronoun_H_S                               -0.26080    0.12341 -2.11336    0.035
    ## PSA_GenLang                                0.10699    0.09085  1.17767    0.239
    ## Biographies_They                          -0.05933    0.09084 -0.65305    0.514
    ## Pronoun_T_HS:PSA_GenLang                  -1.90734    0.19075 -9.99914    0.000
    ## Pronoun_H_S:PSA_GenLang                    0.26552    0.22695  1.16994    0.242
    ## PSA_GenLang:Biographies_They               0.42601    0.18168  2.34483    0.019
    ## Pronoun_T_HS:Biographies_They             -0.16030    0.19077 -0.84028    0.401
    ## Pronoun_H_S:Biographies_They               0.08454    0.22696  0.37248    0.710
    ## Pronoun_T_HS:PSA_GenLang:Biographies_They  0.88225    0.38148  2.31271    0.021
    ## Pronoun_H_S:PSA_GenLang:Biographies_They   0.13413    0.45382  0.29555    0.768
    ##                                           Pr(>|t|)    
    ## (Intercept)                                 <2e-16 ***
    ## Pronoun_T_HS                                <2e-16 ***
    ## Pronoun_H_S                                 0.0346 *  
    ## PSA_GenLang                                 0.2389    
    ## Biographies_They                            0.5137    
    ## Pronoun_T_HS:PSA_GenLang                    <2e-16 ***
    ## Pronoun_H_S:PSA_GenLang                     0.2420    
    ## PSA_GenLang:Biographies_They                0.0190 *  
    ## Pronoun_T_HS:Biographies_They               0.4007    
    ## Pronoun_H_S:Biographies_They                0.7095    
    ## Pronoun_T_HS:PSA_GenLang:Biographies_They   0.0207 *  
    ## Pronoun_H_S:PSA_GenLang:Biographies_They    0.7676    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##                (Intr) Pr_T_HS Pr_H_S PSA_GnL Bgrp_T Pr_T_HS:PSA_GL
    ## Pronon_T_HS     0.054                                             
    ## Pronoun_H_S    -0.071 -0.066                                      
    ## PSA_GenLang    -0.221  0.071   0.056                              
    ## Bigrphs_Thy    -0.055 -0.024   0.023  0.105                       
    ## Pr_T_HS:PSA_GL  0.062 -0.312   0.042  0.058   0.053               
    ## Pr_H_S:PSA_GL   0.054  0.044  -0.179 -0.084  -0.013 -0.060        
    ## PSA_GnL:B_T     0.093  0.053  -0.012 -0.063  -0.251 -0.024        
    ## Pr_T_HS:B_T    -0.020 -0.045   0.014  0.053   0.058  0.071        
    ## Prn_H_S:B_T     0.022  0.019  -0.071 -0.013  -0.084 -0.010        
    ## P_T_HS:PSA_GL:  0.047  0.073  -0.010 -0.024   0.071 -0.048        
    ## P_H_S:PSA_GL:  -0.011 -0.008   0.124  0.025   0.061  0.018        
    ##                Pr_H_S:PSA_GL PSA_GL: P_T_HS:B P_H_S:B P_T_HS:PSA_GL:
    ## Pronon_T_HS                                                         
    ## Pronoun_H_S                                                         
    ## PSA_GenLang                                                         
    ## Bigrphs_Thy                                                         
    ## Pr_T_HS:PSA_GL                                                      
    ## Pr_H_S:PSA_GL                                                       
    ## PSA_GnL:B_T     0.025                                               
    ## Pr_T_HS:B_T    -0.009         0.071                                 
    ## Prn_H_S:B_T     0.136         0.061  -0.060                         
    ## P_T_HS:PSA_GL:  0.018         0.058  -0.312    0.044                
    ## P_H_S:PSA_GL:  -0.076        -0.084   0.044   -0.194  -0.060

### Three-Way Interaction

The main model has Helmert coding for Pronoun and Effects coding (.5,
-.5) for PSA and Biographies. This means Pronoun (T vs HS) \* PSA \*
Biographies is testing the interaction between Pronoun and PSA across
both Biographies conditions.

Dummy coding Biographies with they/them biographies as 1 and he/she
biographies as 0 tests the interaction between Pronoun and PSA for just
the he/she Biographies:

``` r
d %<>% mutate(BioDummy_T=Biographies)
contrasts(d$BioDummy_T)=cbind("_They1"=c(0,1))

model_p_dummyT <- glmer(P_Acc ~ Pronoun * PSA * BioDummy_T + (1|Name),
                  data=d, family="binomial")

summary(model_p_dummyT)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: P_Acc ~ Pronoun * PSA * BioDummy_T + (1 | Name)
    ##    Data: d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   3464.0   3545.3  -1719.0   3438.0     3827 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -3.4309 -0.3672  0.3616  0.4550  3.0687 
    ## 
    ## Random effects:
    ##  Groups Name        Variance Std.Dev.
    ##  Name   (Intercept) 0.007172 0.08469 
    ## Number of obs: 3840, groups:  Name, 12
    ## 
    ## Fixed effects:
    ##                                           Estimate Std. Error z value Pr(>|z|)
    ## (Intercept)                                0.72475    0.07060  10.265   <2e-16
    ## Pronoun_T_HS                               3.23575    0.13822  23.411   <2e-16
    ## Pronoun_H_S                               -0.30307    0.17352  -1.747   0.0807
    ## PSA_GenLang                               -0.10602    0.13243  -0.801   0.4234
    ## BioDummy_T_They1                          -0.05933    0.09085  -0.653   0.5137
    ## Pronoun_T_HS:PSA_GenLang                  -2.34842    0.27618  -8.503   <2e-16
    ## Pronoun_H_S:PSA_GenLang                    0.19845    0.33288   0.596   0.5511
    ## Pronoun_T_HS:BioDummy_T_They1             -0.16029    0.19079  -0.840   0.4008
    ## Pronoun_H_S:BioDummy_T_They1               0.08451    0.22697   0.372   0.7096
    ## PSA_GenLang:BioDummy_T_They1               0.42602    0.18169   2.345   0.0190
    ## Pronoun_T_HS:PSA_GenLang:BioDummy_T_They1  0.88220    0.38159   2.312   0.0208
    ## Pronoun_H_S:PSA_GenLang:BioDummy_T_They1   0.13412    0.45385   0.296   0.7676
    ##                                              
    ## (Intercept)                               ***
    ## Pronoun_T_HS                              ***
    ## Pronoun_H_S                               .  
    ## PSA_GenLang                                  
    ## BioDummy_T_They1                             
    ## Pronoun_T_HS:PSA_GenLang                  ***
    ## Pronoun_H_S:PSA_GenLang                      
    ## Pronoun_T_HS:BioDummy_T_They1                
    ## Pronoun_H_S:BioDummy_T_They1                 
    ## PSA_GenLang:BioDummy_T_They1              *  
    ## Pronoun_T_HS:PSA_GenLang:BioDummy_T_They1 *  
    ## Pronoun_H_S:PSA_GenLang:BioDummy_T_They1     
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##                (Intr) Pr_T_HS Pr_H_S PSA_GnL BD_T_T Pr_T_HS:PSA_GL
    ## Pronon_T_HS     0.074                                             
    ## Pronoun_H_S    -0.093 -0.075                                      
    ## PSA_GenLang    -0.314  0.017   0.066                              
    ## BDmmy_T_Th1    -0.684 -0.056   0.071  0.244                       
    ## Pr_T_HS:PSA_GL  0.015 -0.367   0.050  0.078  -0.012               
    ## Pr_H_S:PSA_GL   0.065  0.050  -0.294 -0.102  -0.050 -0.073        
    ## P_T_HS:BD_T    -0.052 -0.722   0.049 -0.012   0.058  0.265        
    ## P_H_S:BD_T_     0.070  0.055  -0.705 -0.051  -0.084 -0.037        
    ## PSA_GL:BD_T     0.229 -0.012  -0.049 -0.729  -0.251 -0.057        
    ## P_T_HS:PSA_GL: -0.011  0.266  -0.036 -0.056   0.071 -0.724        
    ## P_H_S:PSA_GL:  -0.047 -0.036   0.215  0.074   0.061  0.054        
    ##                Pr_H_S:PSA_GL P_T_HS:B P_H_S:B PSA_GL: P_T_HS:PSA_GL:
    ## Pronon_T_HS                                                         
    ## Pronoun_H_S                                                         
    ## PSA_GenLang                                                         
    ## BDmmy_T_Th1                                                         
    ## Pr_T_HS:PSA_GL                                                      
    ## Pr_H_S:PSA_GL                                                       
    ## P_T_HS:BD_T    -0.036                                               
    ## P_H_S:BD_T_     0.225        -0.060                                 
    ## PSA_GL:BD_T     0.074         0.071    0.061                        
    ## P_T_HS:PSA_GL:  0.053        -0.312    0.044   0.058                
    ## P_H_S:PSA_GL:  -0.733         0.044   -0.194  -0.084  -0.060

Conversely, dummy coding Biographies with he/she biographies as 1 and
they biographies as 0 tests the interaction between Pronoun and PSA for
just the they Biographies.

``` r
d %<>% mutate(BioDummy_HS=Biographies)
contrasts(d$BioDummy_HS)=cbind("_HeShe"=c(1,0))

model_p_dummyHS <- glmer(P_Acc ~ Pronoun * PSA * BioDummy_HS + (1|Name),
                        data=d, family="binomial")
summary(model_p_dummyHS)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: P_Acc ~ Pronoun * PSA * BioDummy_HS + (1 | Name)
    ##    Data: d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   3464.0   3545.3  -1719.0   3438.0     3827 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -3.4310 -0.3672  0.3616  0.4550  3.0688 
    ## 
    ## Random effects:
    ##  Groups Name        Variance Std.Dev.
    ##  Name   (Intercept) 0.007173 0.08469 
    ## Number of obs: 3840, groups:  Name, 12
    ## 
    ## Fixed effects:
    ##                                            Estimate Std. Error z value Pr(>|z|)
    ## (Intercept)                                 0.66541    0.06685   9.954  < 2e-16
    ## Pronoun_T_HS                                3.07546    0.13207  23.286  < 2e-16
    ## Pronoun_H_S                                -0.21855    0.16160  -1.352   0.1762
    ## PSA_GenLang                                 0.31999    0.12441   2.572   0.0101
    ## BioDummy_HS_HeShe                           0.05934    0.09085   0.653   0.5137
    ## Pronoun_T_HS:PSA_GenLang                   -1.46623    0.26337  -5.567 2.59e-08
    ## Pronoun_H_S:PSA_GenLang                     0.33259    0.30870   1.077   0.2813
    ## Pronoun_T_HS:BioDummy_HS_HeShe              0.16030    0.19081   0.840   0.4008
    ## Pronoun_H_S:BioDummy_HS_HeShe              -0.08453    0.22703  -0.372   0.7096
    ## PSA_GenLang:BioDummy_HS_HeShe              -0.42602    0.18172  -2.344   0.0191
    ## Pronoun_T_HS:PSA_GenLang:BioDummy_HS_HeShe -0.88224    0.38179  -2.311   0.0208
    ## Pronoun_H_S:PSA_GenLang:BioDummy_HS_HeShe  -0.13417    0.45422  -0.295   0.7677
    ##                                               
    ## (Intercept)                                ***
    ## Pronoun_T_HS                               ***
    ## Pronoun_H_S                                   
    ## PSA_GenLang                                *  
    ## BioDummy_HS_HeShe                             
    ## Pronoun_T_HS:PSA_GenLang                   ***
    ## Pronoun_H_S:PSA_GenLang                       
    ## Pronoun_T_HS:BioDummy_HS_HeShe                
    ## Pronoun_H_S:BioDummy_HS_HeShe                 
    ## PSA_GenLang:BioDummy_HS_HeShe              *  
    ## Pronoun_T_HS:PSA_GenLang:BioDummy_HS_HeShe *  
    ## Pronoun_H_S:PSA_GenLang:BioDummy_HS_HeShe     
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##                (Intr) Pr_T_HS Pr_H_S PSA_GnL BD_HS_ Pr_T_HS:PSA_GL
    ## Pronon_T_HS     0.035                                             
    ## Pronoun_H_S    -0.058 -0.049                                      
    ## PSA_GenLang    -0.145  0.131   0.049                              
    ## BDmmy_HS_HS    -0.637 -0.025   0.041  0.106                       
    ## Pr_T_HS:PSA_GL  0.122 -0.252   0.035  0.036  -0.090               
    ## Pr_H_S:PSA_GL   0.049  0.038  -0.061 -0.063  -0.035 -0.045        
    ## P_T_HS:BD_H    -0.024 -0.689   0.031 -0.090   0.058  0.174        
    ## P_H_S:BD_HS     0.040  0.029  -0.648 -0.035  -0.084 -0.025        
    ## PSA_GL:BD_H     0.099 -0.090  -0.034 -0.685  -0.251 -0.025        
    ## P_T_HS:PSA_GL: -0.084  0.172  -0.023 -0.025   0.071 -0.690        
    ## P_H_S:PSA_GL:  -0.033 -0.026   0.042  0.043   0.061  0.031        
    ##                Pr_H_S:PSA_GL P_T_HS:B P_H_S:B PSA_GL: P_T_HS:PSA_GL:
    ## Pronon_T_HS                                                         
    ## Pronoun_H_S                                                         
    ## PSA_GenLang                                                         
    ## BDmmy_HS_HS                                                         
    ## Pr_T_HS:PSA_GL                                                      
    ## Pr_H_S:PSA_GL                                                       
    ## P_T_HS:BD_H    -0.026                                               
    ## P_H_S:BD_HS     0.042        -0.060                                 
    ## PSA_GL:BD_H     0.043         0.071    0.061                        
    ## P_T_HS:PSA_GL:  0.031        -0.312    0.044   0.058                
    ## P_H_S:PSA_GL:  -0.680         0.044   -0.194  -0.084  -0.060

The three models to compare:

``` r
interaction_1 <- tidy(model_p@model) %>% 
  select(term, estimate, p.value) %>%
  filter(term=="Pronoun_T_HS" | term=="PSA_GenLang" |
           term=="Pronoun_T_HS:PSA_GenLang") %>%
  rename("AcrossBio_Est"="estimate", "AcrossBio_p"="p.value")

interaction_2 <- tidy(model_p_dummyT) %>% 
  select(term, estimate, p.value) %>%
  filter(term=="Pronoun_T_HS" | term=="PSA_GenLang" |
           term=="Pronoun_T_HS:PSA_GenLang") %>%
  rename("HeSheBio_Est"="estimate", "HeSheBio_p"="p.value") %>%
  left_join(interaction_1)
```

    ## Joining, by = "term"

``` r
interaction_3 <- tidy(model_p_dummyHS) %>% 
  select(term, estimate, p.value) %>%
  filter(term=="Pronoun_T_HS" | term=="PSA_GenLang" |
           term=="Pronoun_T_HS:PSA_GenLang") %>%
  rename("TheyBio_Est"="estimate", "TheyBio_p"="p.value") %>%
  left_join(interaction_2)
```

    ## Joining, by = "term"

``` r
interaction_3
```

    ## # A tibble: 3 x 7
    ##   term   TheyBio_Est TheyBio_p HeSheBio_Est HeSheBio_p AcrossBio_Est AcrossBio_p
    ##   <chr>        <dbl>     <dbl>        <dbl>      <dbl>         <dbl>       <dbl>
    ## 1 Prono~       3.08  6.17e-120        3.24   3.32e-121         3.16    4.10e-238
    ## 2 PSA_G~       0.320 1.01e-  2       -0.106  4.23e-  1         0.107   2.39e-  1
    ## 3 Prono~      -1.47  2.59e-  8       -2.35   1.84e- 17        -1.91    1.54e- 23

The estimate for the PSA\*Pronoun interaction is -2.34 for the he/she
biographies and -1.47 for the they biographies, which means that the
pronoun PSA reduced the relative difficulty of they/them more when
paired with the he/she biographies than with they biographies.
Connecting to the plot, the PSA-Neither difference is larger than
Both-Story difference.

### Producing they/them at least once

``` r
d %>% filter(P_Response=="they/them") %>%
  group_by(Condition) %>% 
  summarize(they=n_distinct(Participant)) %>%
  mutate(n=80) %>%
  mutate(prop=they/n)
```

    ## # A tibble: 4 x 4
    ##   Condition  they     n  prop
    ##   <fct>     <int> <dbl> <dbl>
    ## 1 both         38    80 0.475
    ## 2 neither      21    80 0.262
    ## 3 psa          46    80 0.575
    ## 4 story        28    80 0.35

Model with whether each participant produced they/them at least once as
the outcome variable. Higher with the gendered language PSA, no effect
of Biographies, vaguely trending interaction.

``` r
model_p_they <- glm(P_UseThey ~ PSA * Biographies, 
                d_they, family="binomial")
summary(model_p_they)
```

    ## 
    ## Call:
    ## glm(formula = P_UseThey ~ PSA * Biographies, family = "binomial", 
    ##     data = d_they)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -1.3082  -0.9282  -0.7804   1.2202   1.6356  
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                  -0.362464   0.117471  -3.086  0.00203 ** 
    ## PSA_GenLang                   0.927126   0.234941   3.946 7.94e-05 ***
    ## Biographies_They              0.005806   0.234941   0.025  0.98029    
    ## PSA_GenLang:Biographies_They -0.816340   0.469882  -1.737  0.08233 .  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 434.46  on 319  degrees of freedom
    ## Residual deviance: 415.50  on 316  degrees of freedom
    ## AIC: 423.5
    ## 
    ## Number of Fisher Scoring iterations: 4

# Memory Predicting Production

### Descriptive Stats

``` r
d %>% group_by(Pronoun, Condition, M_Acc) %>%
      summarise(m=mean(P_Acc))    
```

    ## # A tibble: 24 x 4
    ## # Groups:   Pronoun, Condition [12]
    ##    Pronoun Condition M_Acc     m
    ##    <fct>   <fct>     <int> <dbl>
    ##  1 he/him  both          0 0.738
    ##  2 he/him  both          1 0.867
    ##  3 he/him  neither       0 0.879
    ##  4 he/him  neither       1 0.927
    ##  5 he/him  psa           0 0.781
    ##  6 he/him  psa           1 0.816
    ##  7 he/him  story         0 0.812
    ##  8 he/him  story         1 0.892
    ##  9 she/her both          0 0.721
    ## 10 she/her both          1 0.857
    ## # ... with 14 more rows

Combining the two measures, there are 4 possible patterns: getting both
right, getting both wrong, getting just memory right, and getting just
production right.

``` r
mp_acc <- d %>% 
          mutate(BothRight=ifelse(M_Acc==1 & P_Acc==1, 1, 0)) %>%
          mutate(BothWrong=ifelse(M_Acc==0 & P_Acc==0, 1, 0)) %>%
          mutate(MemOnly=ifelse(M_Acc==1 & P_Acc==0, 1, 0)) %>%
          mutate(ProdOnly=ifelse(M_Acc==0 & P_Acc==1, 1, 0)) %>%
          pivot_longer(cols=c(BothRight, BothWrong, MemOnly, ProdOnly),
                       names_to="Combined_Accuracy") %>%
          group_by(Pronoun, Combined_Accuracy) %>%
          summarise(m=mean(value))
mp_acc
```

    ## # A tibble: 12 x 3
    ## # Groups:   Pronoun [3]
    ##    Pronoun   Combined_Accuracy      m
    ##    <fct>     <chr>              <dbl>
    ##  1 he/him    BothRight         0.691 
    ##  2 he/him    BothWrong         0.0430
    ##  3 he/him    MemOnly           0.0977
    ##  4 he/him    ProdOnly          0.169 
    ##  5 she/her   BothRight         0.695 
    ##  6 she/her   BothWrong         0.0531
    ##  7 she/her   MemOnly           0.117 
    ##  8 she/her   ProdOnly          0.134 
    ##  9 they/them BothRight         0.166 
    ## 10 they/them BothWrong         0.397 
    ## 11 they/them MemOnly           0.381 
    ## 12 they/them ProdOnly          0.0555

### Model

Maximal model has interactions between Pronoun (2 contrasts), Memory
Accuracy, PSA, and Biographies, then random intercepts by item.

``` r
model_mp_full <- P_Acc ~ Pronoun * PSA * Biographies * M_Acc + 
                (Pronoun|Participant) + (Pronoun|Name)

model_mp <- buildmer(model_mp_full, d, 
                     family="binomial", direction=c("order"))
```

    ## Determining predictor order

    ## Fitting via glm: P_Acc ~ 1

    ## Currently evaluating LRT for: Biographies, M_Acc, Pronoun, PSA

    ## Fitting via glm: P_Acc ~ 1 + Biographies

    ## Fitting via glm: P_Acc ~ 1 + M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun

    ## Fitting via glm: P_Acc ~ 1 + PSA

    ## Updating formula: P_Acc ~ 1 + Pronoun

    ## Currently evaluating LRT for: Biographies, M_Acc, PSA

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + PSA

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc

    ## Currently evaluating LRT for: Biographies, Pronoun:M_Acc, PSA

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + Pronoun:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA

    ## Currently evaluating LRT for: Biographies, Pronoun:M_Acc, Pronoun:PSA,
    ##     PSA:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + PSA:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA

    ## Currently evaluating LRT for: Biographies, Pronoun:M_Acc, PSA:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     PSA:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc

    ## Currently evaluating LRT for: Biographies, PSA:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + PSA:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + PSA:M_Acc

    ## Currently evaluating LRT for: Biographies, Pronoun:PSA:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:PSA:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:PSA:M_Acc

    ## Currently evaluating LRT for: Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies

    ## Currently evaluating LRT for: Biographies:M_Acc, Pronoun:Biographies,
    ##     PSA:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     Pronoun:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies

    ## Currently evaluating LRT for: Biographies:M_Acc, Pronoun:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + Pronoun:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + Biographies:M_Acc

    ## Currently evaluating LRT for: Pronoun:Biographies,
    ##     PSA:Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + Pronoun:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + PSA:Biographies:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + PSA:Biographies:M_Acc

    ## Currently evaluating LRT for: Pronoun:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies

    ## Currently evaluating LRT for: Pronoun:Biographies:M_Acc,
    ##     Pronoun:PSA:Biographies

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies

    ## Currently evaluating LRT for: Pronoun:Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     Pronoun:Biographies:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     Pronoun:Biographies:M_Acc

    ## Currently evaluating LRT for: Pronoun:PSA:Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     Pronoun:M_Acc:Biographies + Pronoun:PSA:Biographies:M_Acc

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     Pronoun:M_Acc:Biographies + Pronoun:PSA:Biographies:M_Acc

    ## Fitting via glm: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     Pronoun:M_Acc:Biographies + Pronoun:PSA:Biographies:M_Acc

    ## Currently evaluating LRT for: 1 | Name, 1 | Participant

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + M_Acc + PSA +
    ##     Pronoun:PSA + Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA +
    ##     Biographies + PSA:Biographies + M_Acc:Biographies +
    ##     M_Acc:PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + Pronoun:M_Acc:Biographies +
    ##     Pronoun:M_Acc:PSA:Biographies + (1 | Name)

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + M_Acc + PSA +
    ##     Pronoun:PSA + Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA +
    ##     Biographies + PSA:Biographies + M_Acc:Biographies +
    ##     M_Acc:PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + Pronoun:M_Acc:Biographies +
    ##     Pronoun:M_Acc:PSA:Biographies + (1 | Participant)

    ## Updating formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA +
    ##     Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies +
    ##     PSA:Biographies + M_Acc:Biographies + M_Acc:PSA:Biographies +
    ##     Pronoun:Biographies + Pronoun:PSA:Biographies +
    ##     Pronoun:M_Acc:Biographies + Pronoun:M_Acc:PSA:Biographies + (1 |
    ##     Name)

    ## Currently evaluating LRT for: Pronoun | Name, 1 | Participant

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + M_Acc + PSA +
    ##     Pronoun:PSA + Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA +
    ##     Biographies + PSA:Biographies + M_Acc:Biographies +
    ##     M_Acc:PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + Pronoun:M_Acc:Biographies +
    ##     Pronoun:M_Acc:PSA:Biographies + (1 + Pronoun | Name)

    ## boundary (singular) fit: see help('isSingular')

    ## Fitting via glmer, with ML: P_Acc ~ 1 + Pronoun + M_Acc + PSA +
    ##     Pronoun:PSA + Pronoun:M_Acc + M_Acc:PSA + Pronoun:M_Acc:PSA +
    ##     Biographies + PSA:Biographies + M_Acc:Biographies +
    ##     M_Acc:PSA:Biographies + Pronoun:Biographies +
    ##     Pronoun:PSA:Biographies + Pronoun:M_Acc:Biographies +
    ##     Pronoun:M_Acc:PSA:Biographies + (1 | Name) + (1 | Participant)

    ## Ending the ordering procedure due to having reached the maximal
    ##     feasible model - all higher models failed to converge. The types of
    ##     convergence failure are: Singular fit lme4 reports not having
    ##     converged (-1)

``` r
summary(model_mp)
```

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) (p-values based on Wald z-scores) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: P_Acc ~ 1 + Pronoun + M_Acc + PSA + Pronoun:PSA + Pronoun:M_Acc +  
    ##     M_Acc:PSA + Pronoun:M_Acc:PSA + Biographies + PSA:Biographies +  
    ##     M_Acc:Biographies + M_Acc:PSA:Biographies + Pronoun:Biographies +  
    ##     Pronoun:PSA:Biographies + Pronoun:M_Acc:Biographies + Pronoun:M_Acc:PSA:Biographies +  
    ##     (1 | Name)
    ##    Data: d
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##   3384.0   3540.3  -1667.0   3334.0     3815 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -3.6071 -0.4348  0.3461  0.4641  4.9870 
    ## 
    ## Random effects:
    ##  Groups Name        Variance Std.Dev.
    ##  Name   (Intercept) 0.003596 0.05996 
    ## Number of obs: 3840, groups:  Name, 12
    ## 
    ## Fixed effects:
    ##                                                 Estimate Std. Error  z value
    ## (Intercept)                                      0.10688    0.08987  1.18937
    ## Pronoun_T_HS                                     3.33124    0.18516 17.99081
    ## Pronoun_H_S                                     -0.46920    0.22284 -2.10556
    ## M_Acc                                            0.83019    0.10374  8.00239
    ## PSA_GenLang                                     -0.04541    0.17616 -0.25777
    ## Biographies_They                                 0.04027    0.17617  0.22858
    ## Pronoun_T_HS:PSA_GenLang                        -1.77795    0.36947 -4.81210
    ## Pronoun_H_S:PSA_GenLang                         -0.12067    0.44046 -0.27396
    ## Pronoun_T_HS:M_Acc                              -0.40183    0.21858 -1.83842
    ## Pronoun_H_S:M_Acc                                0.25503    0.25851  0.98651
    ## M_Acc:PSA_GenLang                                0.23980    0.20735  1.15653
    ## PSA_GenLang:Biographies_They                     0.16487    0.35233  0.46794
    ## M_Acc:Biographies_They                          -0.11428    0.20738 -0.55105
    ## Pronoun_T_HS:Biographies_They                   -0.71912    0.36942 -1.94664
    ## Pronoun_H_S:Biographies_They                     0.36914    0.44050  0.83800
    ## Pronoun_T_HS:M_Acc:PSA_GenLang                  -0.23686    0.43705 -0.54195
    ## Pronoun_H_S:M_Acc:PSA_GenLang                    0.48046    0.51606  0.93101
    ## M_Acc:PSA_GenLang:Biographies_They               0.42514    0.41472  1.02512
    ## Pronoun_T_HS:PSA_GenLang:Biographies_They        1.96223    0.74028  2.65066
    ## Pronoun_H_S:PSA_GenLang:Biographies_They         1.04670    0.88132  1.18765
    ## Pronoun_T_HS:M_Acc:Biographies_They              0.83020    0.43694  1.90005
    ## Pronoun_H_S:M_Acc:Biographies_They              -0.38998    0.51604 -0.75571
    ## Pronoun_T_HS:M_Acc:PSA_GenLang:Biographies_They -1.52764    0.87578 -1.74431
    ## Pronoun_H_S:M_Acc:PSA_GenLang:Biographies_They  -1.20247    1.03266 -1.16445
    ##                                                 Pr(>|z|) Pr(>|t|)    
    ## (Intercept)                                        0.234  0.23429    
    ## Pronoun_T_HS                                       0.000  < 2e-16 ***
    ## Pronoun_H_S                                        0.035  0.03524 *  
    ## M_Acc                                              0.000 1.22e-15 ***
    ## PSA_GenLang                                        0.797  0.79659    
    ## Biographies_They                                   0.819  0.81919    
    ## Pronoun_T_HS:PSA_GenLang                           0.000 1.49e-06 ***
    ## Pronoun_H_S:PSA_GenLang                            0.784  0.78411    
    ## Pronoun_T_HS:M_Acc                                 0.066  0.06600 .  
    ## Pronoun_H_S:M_Acc                                  0.324  0.32388    
    ## M_Acc:PSA_GenLang                                  0.247  0.24746    
    ## PSA_GenLang:Biographies_They                       0.640  0.63983    
    ## M_Acc:Biographies_They                             0.582  0.58160    
    ## Pronoun_T_HS:Biographies_They                      0.052  0.05158 .  
    ## Pronoun_H_S:Biographies_They                       0.402  0.40203    
    ## Pronoun_T_HS:M_Acc:PSA_GenLang                     0.588  0.58785    
    ## Pronoun_H_S:M_Acc:PSA_GenLang                      0.352  0.35185    
    ## M_Acc:PSA_GenLang:Biographies_They                 0.305  0.30531    
    ## Pronoun_T_HS:PSA_GenLang:Biographies_They          0.008  0.00803 ** 
    ## Pronoun_H_S:PSA_GenLang:Biographies_They           0.235  0.23497    
    ## Pronoun_T_HS:M_Acc:Biographies_They                0.057  0.05743 .  
    ## Pronoun_H_S:M_Acc:Biographies_They                 0.450  0.44982    
    ## Pronoun_T_HS:M_Acc:PSA_GenLang:Biographies_They    0.081  0.08111 .  
    ## Pronoun_H_S:M_Acc:PSA_GenLang:Biographies_They     0.244  0.24424    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    ## 
    ## Correlation matrix not shown by default, as p = 24 > 12.
    ## Use print(x, correlation=TRUE)  or
    ##     vcov(x)        if you need it
