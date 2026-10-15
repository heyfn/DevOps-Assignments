\# Assignment 9: Jenkins Multi-Stage Pipeline



\## Objective



To create a Jenkins multi-stage pipeline that automatically tests, deploys, runs, and verifies a Python Flask application using a Jenkinsfile stored in GitHub.



\## Tools and Technologies



\- Jenkins

\- Docker

\- Git and GitHub

\- Python 3

\- Flask

\- pytest

\- Jenkins Pipeline (Declarative Syntax)



\## Application Overview



The application is a simple Flask web application that returns a welcome message when accessed through the root URL (`/`).



\*\*Expected output:\*\*



`Hello, Jenkins Multi-Stage Pipeline!`



\## Implementation Steps



\### 1. Push Application Files to GitHub



Created and pushed the following files to the `devops-sample-code` repository:



\- `app.py` — Flask application

\- `requirements.txt` — Python dependencies

\- `test\_app.py` — Application test case

\- `Jenkinsfile` — Pipeline configuration



!\[Application files pushed to GitHub](Images/jenkins-pipeline-files-pushed.png)



\### 2. Configure Jenkins Pipeline



Created the `MultiStagePipeline` job in Jenkins and configured it to retrieve the `Jenkinsfile` from the GitHub repository using Pipeline Script from SCM.



\- \*\*SCM:\*\* Git

\- \*\*Repository:\*\* `https://github.com/heyfn/devops-sample-code.git`

\- \*\*Branch:\*\* `\*/main`

\- \*\*Script Path:\*\* `Jenkinsfile`



!\[Jenkins pipeline SCM configuration](Images/jenkins-pipeline-scm-config.png)



\### 3. Configure Python in the Jenkins Container



Installed Python 3, pip, the virtual environment package, and Flask in the Jenkins container. Verified the Python and Flask installations.



!\[Python and Flask installation verification](Images/jenkins-container-python-verify.png)



\### 4. Execute the Multi-Stage Pipeline



The declarative Jenkins pipeline contains four stages:



1\. \*\*Test\*\* — Creates a Python virtual environment, installs dependencies and pytest, and runs the application test.

2\. \*\*Deploy\*\* — Installs the application dependencies in the virtual environment.

3\. \*\*Run Application\*\* — Starts the Flask application in the background.

4\. \*\*Test Application\*\* — Uses `curl` to verify that the application responds successfully on port `5000`.



!\[Successful Jenkins multi-stage pipeline](Images/jenkins-multistage-pipeline-success.png)



\## Result



The Jenkins multi-stage pipeline executed successfully. All four stages completed, and the application returned the expected response:



`Hello, Jenkins Multi-Stage Pipeline!`



\*\*Status: SUCCESS\*\*

