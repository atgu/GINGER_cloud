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
- Let's grab a phenotype file:
  - `gsutil cp gs://neurogap_phenos_genos/NeuroGAP-P_Release5_AllSites.csv .`
- Read in this file:
```R
library(tidyverse)
data = read_csv('NeuroGAP-P_Release5_AllSites.csv')
theme_set(theme_classic())
data %>% ggplot + aes(x = age_at_iview) + geom_histogram()
```

## Stopping RStudio

- Go back to the [RStudio page](https://console.cloud.google.com/marketplace/product/rstudio-launcher-public/rstudio-server-pro-standard-for-gcp?q=search&referrer=search&project=gingeriimak). You can also search for it again.
- Click `View past deployments`
- Find your server, click on the checkbox, and click Delete
- Click `Delete All`

![RStudio stop](img/RStudio/RStudio%20stop.png)