

\# Exercise 5: Docker Security with AppArmor and Python



\## Objective

Understand how to secure Docker containers using AppArmor profiles, apply them via the Docker SDK for Python, and test restricted actions inside a container.



\## Scenario

A Python Flask web application is containerized with Docker. The goal is to restrict the container's access to sensitive directories and prevent unauthorized actions (e.g. executing binaries, reading system files) using an AppArmor security profile.



\## Platform Finding: AppArmor Is Not Available on This Setup



Before applying any profile, the environment was checked for AppArmor support:



```powershell

docker info | Select-String -Pattern "apparmor"

```



No AppArmor entries were found in the Docker information output.



!\[AppArmor not present in Docker info](Images/apparmor-not-supported.png)



Result: no AppArmor entries were found. This is a platform limitation of the current Docker Desktop setup. AppArmor is a Linux kernel LSM (Linux Security Module), and its availability depends on the Linux environment running the containers. Docker Desktop on Windows with WSL2 did not report AppArmor support in this environment.



\## Step 1: Flask Application



`app.py`



```python

from flask import Flask



app = Flask(\_\_name\_\_)



@app.route('/')

def home():

&#x20;   return "Hello, this is a secure Flask application running inside a Docker container!"



if \_\_name\_\_ == '\_\_main\_\_':

&#x20;   app.run(host='0.0.0.0', port=5000)

```



\## Step 2: Dockerfile



```dockerfile

FROM python:3.8-slim

WORKDIR /app

COPY . /app

RUN pip install flask

EXPOSE 5000

CMD \["python", "app.py"]

```



\## Step 3: Build the image



```powershell

docker build -t flask-apparmor .

```



!\[Docker build](Images/apparmor-docker-build.png)



\## Step 4: Attempt to run with an AppArmor security profile



```powershell

docker run --security-opt="apparmor=my-apparmor-profile" -p 5001:5000 -d --name flask-apparmor-test flask-apparmor

```



!\[Container started despite missing AppArmor backend](Images/apparmor-run-attempt.png)



Result: the container started successfully even though no AppArmor entries were reported by Docker. The successful startup does not prove that the named profile was enforced. The host environment's AppArmor support was not available in the Docker information output.



\## Step 5: Confirm the application works normally



```powershell

curl.exe http://localhost:5001/

```



Response:



`Hello, this is a secure Flask application running inside a Docker container!`



The following command was also attempted, as in the exercise:



```powershell

docker run -d --name flask-normal -p 5001:5000 flask-apparmor

```



This second container could not start because port `5001` was already allocated to `flask-apparmor-test`. The successful `curl.exe` response confirms that the application running in the first container was accessible.



!\[Flask app responding and port conflict](Images/apparmor-flask-working.png)



\## Step 6: Clean up



```powershell

docker stop flask-apparmor-test

docker rm flask-apparmor-test flask-normal

docker rmi flask-apparmor

```



The containers and image created for this exercise were removed after testing.



\## What a Working AppArmor Profile Would Do



For reference, on a Linux host with AppArmor available, the profile for this exercise would look like:



```text

\#include <tunables/global>



/usr/bin/python3 {

deny /etc/\*\* r,

deny /var/\*\* rw,

network inet stream,

/app/\*\* rwk,

deny /bin/\*\* rmix,

deny /usr/bin/\*\* rmix,

capability net\_bind\_service,

deny capability sys\_admin,

}

```



It would be loaded with:



```bash

sudo apparmor\_parser -r /etc/apparmor.d/my-apparmor-profile

```



It could then be applied with:



```bash

docker run --security-opt="apparmor=my-apparmor-profile" ...

```



On a supported Linux host, the profile's restrictions could be tested using the Docker SDK for Python (`docker.from\_env()`, `container.exec\_run(...)`), checking whether restricted actions such as `cat /etc/passwd` or executing `/bin/bash` return non-zero exit codes.



\## Q\&A



\*\*Q1: What is the purpose of using AppArmor with Docker containers?\*\*



A: AppArmor enforces mandatory access control policies that confine what a containerized application can access — files, network, binaries — adding a security layer beyond Docker's default isolation.



\*\*Q2: How do AppArmor profiles help secure a Docker container?\*\*



A: Profiles explicitly allow or deny access to specific paths, capabilities, and network actions, so even if an application is compromised, its blast radius is limited to what the profile permits.



\*\*Q3: Why is it important to restrict access to sensitive directories such as `/etc/` and `/var/`?\*\*



A: These directories hold system configuration and credentials. Unrestricted access would let a compromised container read or tamper with host-level settings and sensitive data.



\*\*Q4: What other capabilities can you restrict using AppArmor profiles?\*\*



A: Network binding, binary execution, write access to specific paths, and Linux capabilities such as `cap\_sys\_admin` (which grants broad administrative control).



\*\*Q5: How can you verify if an AppArmor profile is successfully applied to a Docker container?\*\*



A: Inspect the container (`docker inspect` or the Docker SDK's `client.api.inspect\_container()`) and check the `HostConfig.SecurityOpt` field, which lists the applied security options including the AppArmor profile name. However, seeing a security option in the container configuration does not by itself guarantee that the profile is being enforced; the host must support AppArmor.

```



\## Step 14: Verify the files



After saving and closing Notepad, return to PowerShell and run:



```powershell

Get-ChildItem .\\5-Docker-Security-AppArmor.md

```



Then verify the assignment folder:



```powershell

Get-ChildItem .\\apparmor-security-exercise

```



You should see `app.py` and `Dockerfile` in that folder.



\*\*Send me the output of these two commands.\*\* Next, we’ll check Git status and push only the Assignment 5 files and screenshots to your existing GitHub repository.

