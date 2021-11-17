# RStudio in the cloud

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
data %>%
  left_join(ethnicities) %>%
  left_join(languages) %>%
  count(ethnicity, language) %>%
  ggplot + aes(x = ethnicity, y = language, fill = n) +
  geom_tile()

data %>% left_join(sites) %>% left_join(languages) %>%
  count(country, language) %>%
  group_by(country) %>%
  slice_max(order_by = n, n = 3)
```

## PCA data
- Let's get your data
```
gsutil cp gs://neurogap_phenos_genos/pca/[YOURNAME]_neurogap.mds .
gsutil cp gs://neurogap_phenos_genos/pca/[YOURNAME].mds .
```
- Read it in
```
pca_data = read_tsv("[YOURNAME].mds")
ggplot(pca_data) + aes(x = C1, y = C2) + geom_point()
```
- Plot this colored by population
```
ggplot(pca_data) + aes(x = C1, y = C2, color = FID) + geom_point()
```
- So many populations!
- Let's look just at NeuroGAP:
```
neurogap_pca_data = read_table('neurogap.mds')
neurogap_pca_data %>%
  ggplot + aes(x = C1, y = C2, color=FID) + geom_point() 
```
- Color this by population

```bash
neurogap_pca_data %>%
  ggplot + aes(x = C1, y = C2, color=FID) + geom_point() +
  scale_color_brewer(palette = 'Set1') +
  theme_bw() +
  xlab('PC1') + ylab('PC2')
```
## Stopping RStudio

- Go back to the [RStudio page](https://console.cloud.google.com/marketplace/product/rstudio-launcher-public/rstudio-server-pro-standard-for-gcp?q=search&referrer=search&project=gingeriimak). You can also search for it again.
- Click `View past deployments`
- Find your server, click on the checkbox, and click Delete
- Click `Delete All`

![RStudio stop](img/RStudio/RStudio%20stop.png)
