\# Exercise 8: Creating a "Hello World" Jenkins Job



\## Objective



Create a GitHub repository containing a simple shell script, then configure a Jenkins Freestyle job to clone that repository and execute the script — demonstrating a basic CI pipeline from source control to build execution.



\## Step 1: Create the GitHub Repository



Created a public GitHub repository named `devops-sample-code`.



Repository URL: https://github.com/heyfn/devops-sample-code



\## Step 2: Create the Script Locally



Created `hello-world.sh` with the following content:



```bash

\#!/bin/bash

echo "Hello, Jenkins!"

```



\## Step 3: Initialize Git and Commit



Initialized a local Git repository, staged `hello-world.sh`, and committed the file.



!\[Git status showing the staged script](Images/jenkins-hello-world-status.png)



\## Step 4: Push to GitHub



Pushed `hello-world.sh` to the `main` branch of the GitHub repository.



\## Step 5: Verify on GitHub



Verified that `hello-world.sh` is visible in the public repository.



Repository URL: https://github.com/heyfn/devops-sample-code



!\[hello-world.sh visible in GitHub](Images/jenkins-hello-world-github.png)



\## Step 6: Create the Jenkins Job



Created a Jenkins Freestyle project named `HelloWorld`.



\### Source Code Management — Git



Repository URL: `https://github.com/heyfn/devops-sample-code.git`



Branch: `\*/main`



!\[Jenkins Git SCM configuration](Images/jenkins-job-scm-config.png)



\### Build Steps — Execute shell



Configured the build step with the command:



```bash

sh hello-world.sh

```



!\[Jenkins Execute shell build step](Images/jenkins-job-build-step.png)



\## Step 7: Run the Jenkins Job



Ran the `HelloWorld` job using \*\*Build Now\*\* and checked the console output.



The script printed `Hello, Jenkins!`, and the build completed successfully.



!\[Jenkins console output showing a successful build](Images/jenkins-console-output.png)



\## Key Takeaways



\- GitHub stores the source code in a shared repository.

\- Jenkins retrieves the code through Git Source Code Management.

\- A Freestyle job can execute shell commands as a build step.

\- The console output confirms whether the build succeeded.



