# Google Cloud Console

## Walkthrough

- Go to [console.cloud.google.com](https://console.cloud.google.com/)

![Console main page](img/Console/Console%20main%20page.png)

- Select your project (gingeriimak)

![Console project selection](img/Console/Console%20project%20selection.png)

- You may need to go into `ALL` to find it

![Console project selection 2](img/Console/Console%20project%20selection%202.png)

- Click the navigation menu (aka the "hamburger menu" as it has 3 lines that look like a hamburger)
  - Pinning is a great way to save the things you come back to often

![Console navigation](img/Console/Console%20navigation.png)

- The most commonly used features are `Compute Engine` and `Cloud Storage`

![Console compute engine](img/Console/Console%20compute%20engine.png)

- `Compute Engine` is the service that allows you to run any number of "computers"
  - Technically, they're not individual computers, but "virtual machines" (VMs)
  - These simulate the idea of a computer: a CPU, some memory, some disk space
  - When you start a VM, Google Cloud reserves these things for you. When you delete it, they go back into the pool.
  - The power of the Cloud is that you can create one, ten, or thousands of these VMs and run lots of things in parallel.
- `VM instances` show you what you have currently running, and you can create, stop, and delete them.
  - Make sure you delete your instances once you're done using them!

- The Cloud Storage Browser allows you to browse files within that project.

![Console storage browser](img/Console/Console%20storage%20browser.png)

- The top level of the structure are known as "buckets" which hold a collection of files. Bucket names are *globally* unique (since we now have `neurogap_phenos_genos`, no one else in the world can create that bucket).

![Console storage browser 2](img/Console/Console%20storage%20browser%202.png)

- Clicking on a file shows information about the file, and an ultra-useful "copy gsutil path to clipboard" button:
  - We'll use this path frequently when we use the command line to manipulate or download files, or load them into RStudio. 

![Console storage file details](img/Console/Console%20storage%20file%20details.png)
