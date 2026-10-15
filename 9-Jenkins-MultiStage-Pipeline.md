# Exercise 9: Jenkins Multi-Stage Pipeline — Deploying a Python Application

## Objective

Build a Jenkins Pipeline job that performs multiple stages — Build, Test, Deploy, Run Application, and a post-deploy Test — for a sample Flask application, demonstrating a realistic multi-stage CI/CD workflow.

## Scenario

Automate the CI/CD pipeline for a Python Flask application through five stages:

1. **Build** — create a Python virtual environment and install dependencies.
2. **Test** — run unit tests.
3. **Deploy** — copy the application to a mock deployment directory.
4. **Run Application** — start the deployed application in the background.
5. **Test Application** — verify the application using the test script.

## Step 1: Flask Application

### `app.py`

The Flask application returns a message when the root URL is accessed.

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

The declarative Jenkinsfile defines five stages. A Python virtual environment is used to install and run dependencies.

- **Build:** Create the virtual environment and install Flask.
- **Test:** Install pytest and execute the application test.
- **Deploy:** Copy `app.py` into the `python-app-deploy` directory.
- **Run Application:** Start the deployed Flask application in the background.
- **Test Application:** Execute the test script again and verify the application response.

The pipeline also includes post-build actions that display a success or failure message.

## Step 3: Push to the Existing Repository

The application files are stored in the existing `devops-sample-code` GitHub repository.

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

The Jenkins Docker image does not include Python by default. Python 3, pip, the virtual environment package, and Flask were installed in the running Jenkins container.

The installation was verified by checking the Python and Flask versions.

![Python and Flask verified inside the Jenkins container](Images/jenkins-container-python-verify.png)

**Note on persistence:** Installing packages directly inside a running container changes its writable layer. If the container is removed and recreated, these system-level installations may need to be repeated. A custom Jenkins image can provide a more persistent setup.

## Step 7: Troubleshoot Dependency Compatibility

The initial pipeline run failed