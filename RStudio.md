m# RStudio in the cloud

## Starting RStudio

- Go to [console.cloud.google.com](https://console.cloud.google.com/)

![Console main page](img/Console/Console%20main%20page.png)

- At the top is a search bar. Let's search for RStudio Server.

![RStudio search](img/RStudio/RStudio%20search.png)

- Click on `RStudio Workbench Standard for GCP`.
- Click `Launch` on the next page.

![RStudio deploy](img/RStudio/RStudio%20deploy.png)

- Change the `Deployment name` to something unique (like your name)
- Change the zone to anything in `us-central1-a`.
- Accept the terms and click deploy.
- After a while you will see a page like this:

![RStudio deployed](img/RStudio/RStudio%20deployed.png)

- Copy the password, and click on the `Site address` to go to the page.
- Login using the credentials provided (`rstudio-user` and the password you copied)

![RStudio running](img/RStudio/RStudio%20running.png)

- You now have RStudio running in the cloud!
- Let's install `tidyverse` since it doesn't ship with it for some reason
```R
install.packages('tidyverse')
```

## Getting data

- Click `Terminal`
- Here we have our `gsutil` program already installed and it is already set up with permissions
- Let's see what files we can use:
  - `gsutil ls`
- Let's grab the phenotype files:
  - `gsutil cp gs://neurogap_phenos_genos/NeuroGAP-P_Release5_*.csv .`
- Read in this file:
```R
library(tidyverse)
data = read_csv('NeuroGAP-P_Release5_AllSites.csv')
theme_set(theme_classic())
```
## Exploring the data 
- What are all these fields?
```
View(data)
```
- Someone wrote a data dictionary!
```
data_dict = read_csv('NeuroGAP-P_Release5_DataDict.csv')
View(data_dict)
```
- `age_at_iview` looks straightforward. let's plot it!
```
ggplot(data) + aes(x = age_at_iview) + geom_histogram()
```
- Same with `is_case`. Let's count it
```
data %>% group_by(is_case) %>% summarise(n=n())
```
- `count(...)` is a shortcut for `group_by(...) %>% summarize(n=n())`
```
data %>% count(is_case)
```
- Can you look at this by country too?
- Hint: Try writing a command that splits by both `is_case` and `study_country`

## Digging into the phenotype:
- Kessler Psychological Distress Scale (K10) - kten_total
```
ggplot(data) + aes(x = kten_total) + geom_histogram()
```
- Can you plot this by country to see if there are any systematic difference in phenotype definition?
  - Hint: Try using the `fill` aesthetic. Or better yet, `facet_grid` or `facet_wrap`

- Some demographic information:
```
data %>% count(study_country, lang_self_1)
data %>% count(study_country, lang_self_1) %>%
  group_by(study_country) %>%
  slice_max(order_by = n, n = 3)
```

- As with many datasets you'll get, many of the fields are coded
  - We wrote this function to extract information from the data_dict.
  - We won't go into detail on it but if you copy it and execute this cell, we can use the function later.
```
library(data.table)
extract_from_field = function(data_dict, field, new_field_name = 'new_field') {
  output = data_dict %>%
    filter(`Field Name` == field) %>%
    select(`Coded Values OR Formula`)
  return(tibble(fread(output[[1]])) %>% rename(!!field := 'V1',
                                               !!new_field_name := 'V2'))
}
```
- Now let's use it!
```
ethnicities = extract_from_field(data_dict, 'ethnicity_1', 'ethnicity') 
languages = extract_from_field(data_dict, 'lang_self_1', 'language')
countries = extract_from_field(data_dict, 'study_country', 'country')
sites = extract_from_field(data_dict, 'study_site', 'site')
```

- Combine these in to get the annotations in
```
data %>% left_join(countries) %>% left_join(languages) %>%
  count(country, language) %>%
  group_by(country) %>%
  slice_max(order_by = n, n = 3)
```

## Day 4

## Consent languages
- Maybe consent language is a bit more usable:
  - Let's confirm that everyone was consented in a language that they self-reported speaking
```
data %>%
  count(consent_lang %in% c(lang_self_1, lang_self_2, lang_self_3))
```
- What were the top consent languages by country?
```
consent_languages = extract_from_field(data_dict, 'consent_lang', 'language')
data %>%
  left_join(countries) %>% left_join(consent_languages) %>%
  count(country, language) %>%
  group_by(country) %>% slice_max(order_by = n, n = 10)
```
- What is the breakdown by cases and controls:
```
data %>% left_join(countries) %>% left_join(consent_languages) %>%
  count(country, language, case=if_else(is_case == 1, 'case', 'control'))
```
- Hmm, it's a bit annoying to have cases and controls on separate lines.
- Let's pivot this table to make it wide. Now if I can remember how to use pivot_wider:
```
?pivot_wider
```
- `names_from` and `values_from` are the 2 important fields
```
data %>% left_join(countries) %>% left_join(consent_languages) %>%
  count(country, language, case=if_else(is_case == 1, 'case', 'control')) %>%
  pivot_wider(names_from=case, values_from=n)
```
- Those `NA`s will get in the way. Let's make them zero:
```
data %>% left_join(countries) %>% left_join(consent_languages) %>%
  count(country, language, case=if_else(is_case == 1, 'case', 'control')) %>%
  pivot_wider(names_from=case, values_from=n, values_fill = 0)
```
- There are a few languages+countries with low case counts. Let's get rid of those:
```
data %>% left_join(countries) %>% left_join(consent_languages) %>%
  count(country, language, case=if_else(is_case == 1, 'case', 'control')) %>%
  pivot_wider(names_from=case, values_from=n, values_fill = 0) %>%
  filter(case > 10)
```
- Let's get the proportion of cases by language+country
```
data %>% left_join(countries) %>% left_join(consent_languages) %>%
  count(country, language, case=if_else(is_case == 1, 'case', 'control')) %>%
  pivot_wider(names_from=case, values_from=n, values_fill = 0) %>%
  filter(case > 10) %>%
  mutate(proportion_case = case / (case + control))
```
## Day 4 live-coding
### 
```
```
### 
```
```
### 
```
```
### Are only females pregnant?
```
data %>%
  count(msex, is_pregnant)
```
### 
```
data %>% filter(is_pregnant == 1 & msex == 0 & is_case == 0) %>% summarise(mean(kten_total))
```
### Initial exploration of all substance use phenotypes with case status
```
data %>%
  select_at(vars(is_case, msex, starts_with('assist') & !ends_with('amt'))) %>% 
  lm(is_case ~ ., data=.) %>% summary
```
### Depression question by pilot samples
```
neurogap_pca_data = read_table('neurogap.mds')
data %>%
  mutate(pilot_sample = subj_id %in% neurogap_pca_data$IID) -> data
data %>% count(pilot_sample)

kten1gmap = extract_from_field(data_dict, 'kten_q1g', 'kten_question1g')
data %>% left_join(kten1gmap) %>%
  count(pilot_sample = if_else(pilot_sample, 'Pilot', 'Rest'), kten_q1g, kten_question1g) %>%
  group_by(pilot_sample) %>%
  mutate(proportion = n/sum(n)) %>% 
  pivot_wider(names_from = pilot_sample, values_from = c(n, proportion))

```
### Consents: UBACC total score by site
```
data %>%
  left_join(countries) %>%
  group_by(country) %>%
  summarize(mean_ubacc = mean(ubacc_score_t1, na.rm=T))

data %>%
  left_join(countries) %>%
  left_join(consent_languages) %>%
  group_by(country, language) %>%
  summarize(mean_ubacc = mean(ubacc_score_t1, na.rm=T),
            n=n()) %>%
  filter(n > 10) %>%
  arrange(desc(mean_ubacc))
```
### Pivot consent by education status
```
education = extract_from_field(data_dict, 'education', 'education_level')
data %>%
  left_join(countries) %>%
  left_join(education) %>%
  lm(ubacc_score_t1 ~ education, data=.) %>%
  summary

data %>%
  left_join(countries) %>%
  left_join(education) %>%
  ggplot + aes(x = education, y = ubacc_score_t1, group = education) +
  geom_boxplot()

```
### Check by case status
```
data %>%
  left_join(countries) %>%
  left_join(education) %>%
  group_by(is_case) %>%
  summarize(mean(ubacc_score_t1, na.rm=T))
 
data %>%
  left_join(countries) %>%
  left_join(education) %>%
  ggplot + aes(x = ubacc_score_t1, fill = is_case == 1) +
  geom_histogram(alpha=0.5, position='dodge')
```
### Get the number of tries it took to pass the UBACC
```
data %>%
  mutate(n_trials = case_when(ubacc_score_t1 >= 20 ~ 1,
                              ubacc_score_t2 >= 20 ~ 2,
                              ubacc_score_t3 >= 20 ~ 3,
                              ubacc_score_t4 >= 15 ~ 4)) %>%
  count(n_trials)
```
### HIV status concordance (self-report vs CIDI)
```
data %>%
  count(chart=hiv_positive, self_report=cidi_q15)
```
### Look for discordance in HIV status by country
```
data %>% left_join(countries) %>%
  filter(hiv_positive == 1 & cidi_q15 == 0) %>%
  count(country)
```
### Look for missingness by country
```
data %>% left_join(countries) %>%
  count(country, missing_chart=is.na(hiv_positive),
        missing_self_report=is.na(cidi_q15))

```
### BPD mini vs chart
```
data %>% left_join(countries) %>%
  count(mini_BPD1_Past, chart_review_bipolar=psychosis_primary == 2)
```
### Pairwise correlation of variables
```
library(corrplot)
data %>% select_if(is.numeric) %>% cor() -> corr_matrix
corrplot(corr_matrix[colSums(!is.na(corr_matrix)) > 1, colSums(!is.na(corr_matrix)) > 1])
```

## PCA data
- Let's get your data (and some metadata)
```
gsutil cp gs://neurogap_phenos_genos/pca/neurogap.mds .
gsutil cp gs://neurogap_phenos_genos/pca/[YOURNAME].mds .
gsutil cp gs://neurogap_phenos_genos/gnomad_meta_hgdp_tgp_v1.txt .
```
- Read it in
```
pca_data = read_table("[YOURNAME].mds")
ggplot(pca_data) + aes(x = C1, y = C2) + geom_point() +
  xlab('PC1') + ylab('PC2')
```
- Plot this colored by population
```
ggplot(pca_data) + aes(x = C1, y = C2, color = FID) + geom_point() +
  xlab('PC1') + ylab('PC2')
```
- So many populations! We should collapse them a bit. We have a metadata file for the 1000 Genomes and HGDP datasets
```
metadata = read_tsv('gnomad_meta_hgdp_tgp_v1.txt')
pca_data %>%
  left_join(metadata %>% select(project_meta.sample_id, hgdp_tgp_meta.Genetic.region), by=c('IID' = 'project_meta.sample_id')) %>%
  mutate(pop=if_else(!is.na(hgdp_tgp_meta.Genetic.region), hgdp_tgp_meta.Genetic.region, FID))  %>%
  ggplot + aes(x = C1, y = C2, color = pop) + geom_point()
  
# Shapes too
pca_data %>%
  left_join(metadata %>% select(project_meta.sample_id, hgdp_tgp_meta.Genetic.region), by=c('IID' = 'project_meta.sample_id')) %>%
  mutate(pop=if_else(!is.na(hgdp_tgp_meta.Genetic.region), hgdp_tgp_meta.Genetic.region, FID), neurogap_pop=if_else(is.na(hgdp_tgp_meta.Genetic.region), FID, ''))  %>%
  ggplot + aes(x = C1, y = C2, color = pop, shape=neurogap_pop) + geom_point()
```
- Let's look just at NeuroGAP:
```
neurogap_pca_data = read_table('neurogap.mds')
neurogap_pca_data %>%
  ggplot + aes(x = C1, y = C2, color=FID) + geom_point() +
  xlab('PC1') + ylab('PC2')
```
- Color this by population
```
neurogap_pca_data %>%
  ggplot + aes(x = C1, y = C2, color=FID) + geom_point() +
  scale_color_brewer(palette = 'Set1') +
  xlab('PC1') + ylab('PC2')
```
## Stopping RStudio

- Go back to the [RStudio page](https://console.cloud.google.com/marketplace/product/rstudio-launcher-public/rstudio-server-pro-standard-for-gcp?q=search&referrer=search&project=gingeriimak). You can also search for it again.
- Click `View past deployments`
- Find your server, click on the checkbox, and click Delete
- Click `Delete All`

![RStudio stop](img/RStudio/RStudio%20stop.png)
