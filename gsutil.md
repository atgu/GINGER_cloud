# gsutil

## Overview

- Google cloud provides command line utilities to access their services
- If you have not installed `gcloud` yet, go [here](https://cloud.google.com/sdk/docs/install)


- The two main utilities are:
  - `gcloud` - Google Cloud
  - `gsutil` - Google (Cloud) Storage utilities
  
### Running new commands
- We'll walk through some basics of these utilities, but if you get stuck, try running command itself on its own
- This is advice for `gcloud`, but also for every other command you may encounter.
- If you get stuck/don't know what to do:
  - Step 1: `gcloud` (just run the command itself)
    - Usually it will tell you what to do
  - If that doesn't work, step 2: `gcloud -h` (`-h` is the universal command for HALP)
    - Sometimes `-h` doesn't work and you can try `--help` (not usually needed, but actually the case for `gcloud`)
    - Exceptions: basic unix commands that don't have `-h`, or it is a flag for the program 
      - Most common example: `ls` where `-h` means "human readable"
      - For these you can try `man ls` (manual for ls)
  - If that doesn't work, step 3: https://google.com: search for what you want to do: `gsutil copy file`


### Using gsutil
- We will mostly use `gsutil` to move files around
- The main commands are similar to Unix:
  - `gsutil ls`: List a directory or buckets
```bash
~ $ gsutil ls
gs://neurogap_phenos_genos/
~ $ gsutil ls gs://neurogap_phenos_genos/
gs://neurogap_phenos_genos/NeuroGAP-P_Release5_AllSites.csv
gs://neurogap_phenos_genos/NeuroGAP-P_Release5_AllSites.pdf
gs://neurogap_phenos_genos/NeuroGAP-P_Release5_DataDict.csv
gs://neurogap_phenos_genos/NeuroGAP_pilot_clean.bed
gs://neurogap_phenos_genos/NeuroGAP_pilot_clean.bim
gs://neurogap_phenos_genos/NeuroGAP_pilot_clean.fam
gs://neurogap_phenos_genos/NeuroGAP_pilot_clean_grch38.bed
gs://neurogap_phenos_genos/NeuroGAP_pilot_clean_grch38.bim
gs://neurogap_phenos_genos/NeuroGAP_pilot_clean_grch38.fam
gs://neurogap_phenos_genos/gnomad_meta_hgdp_tgp_v1.txt
gs://neurogap_phenos_genos/hgdp_tgp_UPDATED.bed
gs://neurogap_phenos_genos/hgdp_tgp_UPDATED.bim
gs://neurogap_phenos_genos/hgdp_tgp_UPDATED.fam
gs://neurogap_phenos_genos/file_fixes/
```
  - `gsutil mv`: Move a file, or rename it
  - `gsutil cp`: Copy a file (and also possibly rename it)

### Examples:
- Copy a file to a different folder: `gsutil cp gs://bucket/file.txt gs://bucket/folder/`
- Move a file to a different bucket: `gsutil mv gs://bucket/file.txt gs://bucket2/`
- Copy a file from the cloud to your local machine: `gsutil cp gs://bucket/file.txt .`
- Copy a file from your local machine to the cloud: `gsutil cp myfile.txt gs://bucket/`
- Rename a file: `gsutil mv gs://bucket/file.txt gs://bucket/file2.txt`

### gcloud

- `gcloud` allows you to interface with all other google services, particularly VMs (virtual machines)
- `gcloud compute instances list`: look at what instances are currently running
- `gcloud compute instnaces stop VM_NAME`: stop the VM named `VM_NAME`
- `gcloud compute ssh VM_NAME`: connect into a VM you have running