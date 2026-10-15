
# Exercise 9: Jenkins Multi-Stage Pipeline — Deploying a Python Application

## Objective

Build a Jenkins **Pipeline** job (as opposed to a Freestyle job) that performs multiple stages — Build, Test, Deploy, Run Application, and a post-deploy Test — for a sample Flask application, demonstrating a realistic multi-stage CI/CD workflow.

## Scenario

Automate the CI/CD pipeline for a Python Flask application through five stages:

1. **Build** — create a Python virtual environment and install dependencies.
2. **Test** — run unit tests.
3. **Deploy** — copy the app to a mock deployment directory.
4. **Run Application** — start the deployed app in the background.
5. **Test Application** — verify the application using the test script.

## Step 1: Flask Application

### `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, Jenkins Multi-Stage Pipeline!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### `requirements.txt`

```text
flask==2.1.2
Werkzeug==2.1.2
```

Werkzeug is pinned to a compatible version to prevent the import error encountered during the pipeline test.

### `test_app.py`

```python
import unittest
from app import app

class TestApp(unittest.TestCase):
    def test_home(self):
        tester = app.test_client()
        response = tester.get("/")
        print(response.data.decode("utf-8"))
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data.decode("utf-8"), "Hello, Jenkins Multi-Stage Pipeline!")

if __name__ == "__main__":
    unittest.main()
```

## Step 2: Jenkinsfile

The declarative Jenkinsfile defines five stages and uses a Python virtual environment to install and run dependencies.

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
                sh 'python3 -m venv venv'
                sh 'venv/bin/python -m pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'venv/bin/python -m pip install pytest'
                sh 'venv/bin/python -m pytest test_app.py'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    mkdir -p ${WORKSPACE}/python-app-deploy
                    cp ${WORKSPACE}/app.py ${WORKSPACE}/python-app-deploy/
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Running application...'
                sh '''
                    nohup ${WORKSPACE}/venv/bin/python ${WORKSPACE}/python-app-deploy/app.py > ${WORKSPACE}/python-app-deploy/app.log 2>&1 &
                    echo $! > ${WORKSPACE}/python-app-deploy/app.pid
                    sleep 2
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh 'venv/bin/python test_app.py'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
```

The pipeline uses five stages to automate dependency installation, testing, deployment, application startup, and post-deployment testing.

## Step 3: Push to the Existing Repository

Added the application files to the existing `devops-sample-code` GitHub repository used for Exercise 8.

Repository URL: https://github.com/heyfn/devops-sample-code

The files include:

- `app.py`
- `requirements.txt`
- `test_app.py`
- `Jenkinsfile`

The changes were committed and pushed to the `main` branch.

![Pipeline files pushed to GitHub](Images/jenkins-pipeline-files-pushed.png)

## Step 4: Create the Jenkins Pipeline Job

Created a Jenkins Pipeline job named `MultiStagePipeline`.

## Step 5: Configure Pipeline from SCM

Configured Jenkins to retrieve the Jenkinsfile directly from GitHub.

- **Definition:** Pipeline script from SCM
- **SCM:** Git
- **Repository URL:** `https://github.com/heyfn/devops-sample-code.git`
- **Branch Specifier:** `*/main`
- **Script Path:** `Jenkinsfile`

![Pipeline SCM configuration](Images/jenkins-pipeline-scm-config.png)

## Step 6: Install Python in the Jenkins Container

The official Jenkins Docker image does not include Python by default. Python 3, pip, the virtual environment package, and Flask were installed in the running Jenkins container.

The installation was verified by checking the Python and Flask versions.

![Python and Flask verified inside the Jenkins container](Images/jenkins-container-python-verify.png)

**Note on persistence:** Installing packages directly inside a running container changes its writable layer. If the container is removed and recreated, these system-level installations may need to be repeated. A custom Jenkins image can provide a more persistent setup.

## Step 7: Troubleshoot Dependency Compatibility

The first pipeline attempt failed during test collection because Flask `2.1.2` was incompatible with the newer Werkzeug version installed by pip.

The error was:

```text
ImportError: cannot import name 'url_quote' from 'werkzeug.urls'
```

The issue was resolved by pinning Werkzeug to version `2.1.2` in `requirements.txt`, alongside Flask `2.1.2`. The updated dependency file was committed and pushed to GitHub.

## Step 8: Re-run the Pipeline — Full Success

After correcting the dependency versions and updating the pipeline, the Jenkins job was executed again.

All five stages completed successfully:

- Build
- Test
- Deploy
- Run Application
- Test Application

The final test verified the expected application response:

```text
Hello, Jenkins Multi-Stage Pipeline!
```

The pipeline displayed the success message and finished with `SUCCESS`.

![Full pipeline run succeeding through all five stages](Images/jenkins-multistage-pipeline-success.png)

## Key Takeaways

- **Pipeline as Code:** A Jenkinsfile defines the build process alongside the application's source code.
- **Automated Testing:** Unit tests verify that the application behaves as expected.
- **Deployment:** Copying the application into a separate directory demonstrates a basic deployment workflow.
- **Dependency Management:** Pinning compatible package versions helps prevent dependency-related failures.
- **Container Environment:** Required runtimes and tools must be available in the Jenkins execution environment.
- **CI/CD Workflow:** A multi-stage pipeline automates building, testing, deploying, running, and verifying an application.

## Additional Enhancements (Not Implemented)

- Add a code-quality stage using `flake8` or `pylint`.
- Deploy the application into a dedicated Docker container.
- Add email or Slack notifications for pipeline success and failure.
- Build a custom Jenkins image with Python and required tools preinstalled.
