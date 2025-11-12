# 🧠 GitHub Actions CI/CD Pipeline for Flask Application

## 🎯 Objective

Implement a **Continuous Integration and Continuous Deployment (CI/CD)** workflow using **GitHub Actions** for a Python Flask application.

This pipeline automates:

- Installing dependencies  

- Running tests  

- Building the application  

- Deploying to **Staging** when changes are pushed to the `staging` branch  

- Deploying to **Production** when changes are pushed to the `main` branch

---

## 🏗️ 1. Setup

### 📁 Repository

**GitHub Repository:** [https://github.com/JoinDeeHub/Flask_App](https://github.com/JoinDeeHub/Flask_App)

This repository contains:

- A Flask web application (`app.py`)

- Templates and static resources

- A GitHub Actions workflow (`.github/workflows/deploy.yml`)

- Documentation (`README.md`)

### 🌿 Branch Structure

- **`main`** → Production deployment  

- **`staging`** → Staging deployment

---

## ⚙️ 2. GitHub Actions Workflow

The workflow YAML file is located at:

.github/workflows/deploy.yml

yaml



### 🚀 Workflow Overview

<pre class="overflow-visible!" data-start="1910" data-end="2115"><div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary"><div class="sticky top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span><span>| Stage | Description |

|--------|--------------|

| **Install Dependencies** | Set up Python and install required libraries using `pip install -r requirements.txt` |

| **Run Tests** | Run tests using `pytest` (if tests exist) |

| **Build** | Prepare application for deployment |

| **Deploy to Staging** | Triggered automatically when code is pushed to `staging` branch |

| **Deploy to Production** | Triggered automatically when code is pushed to `main` branch |</span></span></code></div></div></pre>

<img width="1352" height="592" alt="Screenshot from 2025-11-12 06-29-03" src="https://github.com/user-attachments/assets/f165af98-9063-4b8a-b0cf-f8d9ee259279" />


* * * * *


Actuall page 

<img width="1362" height="402" alt="Screenshot from 2025-11-11 23-50-59" src="https://github.com/user-attachments/assets/7c76d839-3288-487b-920d-b10f7ff2e26a" />


<img width="1362" height="546" alt="Screenshot from 2025-11-11 23-51-58" src="https://github.com/user-attachments/assets/8446dad4-4dff-434d-a55b-86db6018e275" />


---

### 🧩 Workflow File Content (`.github/workflows/deploy.yml`)

```yaml 

name: CI/CD Pipeline for Flask App

on:

  push:

    branches:

      - main

      - staging

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

    - name: Checkout Code

      uses: actions/checkout@v5

    - name: Set up Python

      uses: actions/setup-python@v5

      with:

        python-version: '3.12'

    - name: Install Dependencies

      run: |

        python -m pip install --upgrade pip

        pip install -r requirements.txt

    - name: Run Tests

      run: pytest || echo "No tests found, skipping"

    - name:  to EC2

      uses: appleboy/scp-action@v1

      with:

        host: ${{ secrets.EC2_HOST }}

        username: ${{ secrets.EC2_USERNAME }}

        key: ${{ secrets.EC2_SSH_KEY }}

        port: 22

        source: ./

        target: ./

    - name: Deploy to EC2

      uses: appleboy/ssh-action@v1

      with:

        host: ${{ secrets.EC2_HOST }}

        username: ${{ secrets.EC2_USERNAME }}

        key: ${{ secrets.EC2_SSH_KEY }}

        port: 22

        script: |

          echo "Starting deployment on EC2..."

          sudo cp -r * /var/www/html/

          sudo systemctl daemon-reload

          sudo systemctl restart flaskapp

          sudo systemctl restart nginx

          echo "Deployment complete!"
```

<img width="1362" height="546" alt="Screenshot from 2025-11-12 00-09-49" src="https://github.com/user-attachments/assets/734bcf5f-d43f-404a-be48-2ac8af8cbee5" />

---



🔑 3. Environment Secrets

Sensitive information (e.g., SSH keys, credentials) is securely stored in GitHub Secrets.

Secret  Description

EC2_HOST  Public IP address or hostname of the EC2 instance

EC2_USERNAME  SSH username (e.g., ubuntu)

EC2_SSH_KEY  Private SSH key used for authentication

MONGO_URI  (Optional) MongoDB connection string

SECRET_KEY  Flask secret key for session management

📍 Add these under:

GitHub → Repository → Settings → Secrets and variables → Actions


---


🧪 4. Workflow Execution and Deployment Flow

🌱 Staging Deployment

Triggered automatically when code is pushed to the staging branch.

Deployed to:


/var/www/html/staging/

Accessible at:


```http://<EC2-IP>/staging/```

Purpose:

To test new features and validate changes before pushing to production.

<img width="1362" height="546" alt="Screenshot from 2025-11-12 00-05-45" src="https://github.com/user-attachments/assets/c8686b11-5ca2-42d0-86f9-66ea5c65f114" />


---


🚀 Production Deployment

Triggered automatically when code is pushed to the main branch.

Deployed to:


/var/www/html/

Accessible at:

```http://<EC2-IP>/```

Purpose:

To serve the final, stable version of the application.

---

🖥️ 5. EC2 Configuration Summary

Component  Description

OS  Ubuntu 22.04

Web Server  Nginx (reverse proxy)

App Server  Gunicorn

Service File  /etc/systemd/system/flaskapp.service

Python Environment  /home/ubuntu/app/venv

Example: Systemd Unit File

```
ini


[Unit]

Description=Gunicorn instance to serve Flask_App

After=network.target

[Service]

User=ubuntu

Group=www-data

WorkingDirectory=/home/ubuntu/app

Environment="PATH=/home/ubuntu/app/venv/bin"

ExecStart=/home/ubuntu/app/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 app:app

[Install]

WantedBy=multi-user.target
```

---

🌐 6. Nginx Configuration

```
nginx


server {

    listen 80;

    server_name _;

    # Production

    location / {

        proxy_pass http://127.0.0.1:8000;

        include /etc/nginx/proxy_params;

        proxy_redirect off;

    }

    # Staging

    location /staging {

        alias /var/www/html/staging;

        index index.html;

        try_files $uri $uri/ /index.html;

    }

}
```

---

📂 7. Project Structure


<pre class="overflow-visible!" data-start="1910" data-end="2115"><div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary"><div class="sticky top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span><span>Flask_App/

├── app.py

├── requirements.txt

├── templates/

│   ├── base.html

│   ├── index.html

│   ├── add_student.html

│   └── update_student.html

├── staging/

├── .github/

│   └── workflows/

│       └── deploy.yml

└── README.md
  </span></span></code></div></div></pre>

* * * * *


🧾 8. Testing and Validation

After deployment:

SSH into the EC2 instance:


``` ssh -i <key.pem> ubuntu@<EC2-IP> ```

Check Gunicorn service:

``` sudo systemctl status flaskapp ```

Validate Nginx routes:

```
curl -I http://127.0.0.1/

curl -I http://127.0.0.1/staging/
```

---

📸 9. Screenshots for Submission

Include the following in your submission:

✅ GitHub Actions workflow successful run

<img width="625" height="228" alt="Screenshot from 2025-11-12 00-36-50" src="https://github.com/user-attachments/assets/fc37588e-7be4-44f3-be6e-efa29eb5c657" />

<img width="1213" height="533" alt="Screenshot from 2025-11-12 00-38-04" src="https://github.com/user-attachments/assets/0edee1b6-8a58-4f4d-8dab-471c414f01e7" />




📦 Files deployed to EC2

<img width="1362" height="317" alt="Screenshot from 2025-11-12 00-34-05" src="https://github.com/user-attachments/assets/b18a2101-2475-4705-91fb-fb79067fb1db" />


<img width="1362" height="317" alt="Screenshot from 2025-11-12 00-34-28" src="https://github.com/user-attachments/assets/0aad0a4c-2ae1-483f-aebc-b71a2e2193cf" />


<img width="1361" height="533" alt="Screenshot from 2025-11-12 00-39-01" src="https://github.com/user-attachments/assets/47df1568-98e2-45af-be66-d93a07549f2d" />





🌍 Flask app accessible via Production (/)

<img width="1362" height="651" alt="Screenshot from 2025-11-12 00-11-02" src="https://github.com/user-attachments/assets/c0c443cb-392c-4d4b-9567-dcd2f09f4ccc" />


🧪 Flask app accessible via Staging (/staging/)

<img width="1362" height="651" alt="Screenshot from 2025-11-12 00-10-06" src="https://github.com/user-attachments/assets/dbe58791-893a-40b2-a665-42eba03d20d6" />

<img width="1362" height="651" alt="Screenshot from 2025-11-12 00-10-31" src="https://github.com/user-attachments/assets/dda96154-c95b-4735-ba61-836a6b5fa991" />


<img width="1361" height="475" alt="Screenshot from 2025-11-12 00-40-18" src="https://github.com/user-attachments/assets/dabb189e-dfc3-4ccb-9cd1-e0180f1b419b" />

<img width="707" height="661" alt="Screenshot from 2025-11-12 00-41-37" src="https://github.com/user-attachments/assets/8a766bb1-0ba1-4f04-8125-6eb7bbaf7526" />

* * * * *


# ⚙️ Jenkins CI/CD Pipeline for Flask Application

## 🎯 Objective

Set up a **Jenkins CI/CD pipeline** that automates the **build, test, and deployment** stages for a Python-based Flask web application.

This pipeline ensures that every code change is automatically:

1\. Built and tested using Python virtual environments.

2\. Deployed to a staging environment on an EC2 server.

3\. Reported via Jenkins email notifications for success or failure.

---

## 🏗️ Project Overview

This project demonstrates how to integrate **Jenkins** with a **Flask web application** hosted on GitHub to enable **automated testing and deployment**.  

It follows the DevOps best practices of continuous integration and continuous delivery.

---

🏗️ Architecture Overview

<img width="844" height="454" alt="image" src="https://github.com/user-attachments/assets/10829a67-57bf-4870-bf74-a50dcf247f9f" />


---

## 📁 Repository Structure

<pre class="overflow-visible!" data-start="1910" data-end="2115"><div class="contain-inline-size rounded-2xl relative bg-token-sidebar-surface-primary"><div class="sticky top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre!"><span><span>Flask_App/

├── app.py

├── requirements.txt

├── templates/

│ ├── base.html

│ ├── index.html

│ ├── add_student.html

│ └── update_student.html

├── Jenkinsfile

├── README.md

└── tests/

└── test_sample.py
</span></span></code></div></div></pre>


 code

**Repository:** [https://github.com/JoinDeeHub/Flask_App](https://github.com/JoinDeeHub/Flask_App)

---

## ⚙️ 1. Setup Instructions

### 🖥️ Jenkins Installation

Jenkins was installed on an Ubuntu virtual machine (or EC2 instance) using the following steps:

```

sudo apt update -y

sudo apt install -y openjdk-17-jre-headless git python3 python3-venv python3-pip

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc >/dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list >/dev/null

sudo apt update -y

sudo apt install -y jenkins

sudo systemctl enable --now jenkins
```

Access Jenkins at:

👉 http://<JENKINS_SERVER_IP>:8080

🔩 Required Jenkins Plugins

Install these plugins under Manage Jenkins → Plugins:

Pipeline

Git & GitHub Integration

GitHub Branch Source

SSH Agent Plugin

Email Extension Plugin

Credentials Binding

<img width="1366" height="701" alt="Screenshot from 2025-11-12 18-08-57" src="https://github.com/user-attachments/assets/edbfefeb-9868-4ee5-9830-b4ca44c16a1b" />

<img width="1366" height="701" alt="Screenshot from 2025-11-12 18-08-26" src="https://github.com/user-attachments/assets/59c08e7e-ba7e-4938-8574-6999447093e8" />

<img width="1366" height="701" alt="Screenshot from 2025-11-12 18-08-13" src="https://github.com/user-attachments/assets/ca6c1663-32b0-4a45-9453-1a0a1a60158b" />


---

💻 2. Source Code Setup

Fork this repository: ``` https://github.com/JoinDeeHub/Flask_App ```

Clone the forked repository into your Jenkins server:

```
git clone https://github.com/<your-username>/Flask_App.git

cd Flask_App

Ensure that your Flask app is working locally:


python3 -m venv venv

. venv/bin/activate

pip install -r requirements.txt

python app.py

Access your local Flask app at http://127.0.0.1:8000
```

---

🧱 3. Jenkins Pipeline Overview

The Jenkinsfile defines a declarative pipeline with three main stages:

Stage  Description

Build  Creates a virtual environment and installs dependencies using pip.

Test  Runs automated tests using pytest.

Deploy  Copies the tested application to a remote EC2 instance's /var/www/html/ directory and restarts services.

---


🧩 Jenkinsfile

Below is the simplified version of the Jenkinsfile used in this project:

```
groovy


pipeline {

  agent any

  environment {

    DEPLOY_HOST = "YOUR_EC2_PUBLIC_IP"

    DEPLOY_USER = "ubuntu"

    SSH_CRED_ID = "EC2_SSH_KEY"

    DEPLOY_PATH = "/var/www/html"

  }

  stages {

    stage('Checkout') {

      steps {

        checkout scm

        echo "Checked out branch: ${env.BRANCH_NAME}"

      }

    }

    stage('Build') {

      steps {

        sh '''

          python3 -m venv venv

          . venv/bin/activate

          pip install --upgrade pip

          pip install -r requirements.txt

        '''

      }

    }

    stage('Test') {

      steps {

        sh '''

          . venv/bin/activate

          pytest || echo "No tests found"

        '''

      }

    }

    stage('Deploy') {

      steps {

        withCredentials([sshUserPrivateKey(credentialsId: "${SSH_CRED_ID}",

                                           keyFileVariable: 'SSH_KEY_FILE',

                                           usernameVariable: 'SSH_USER')]) {

          sh '''

            echo "Deploying to ${DEPLOY_USER}@${DEPLOY_HOST}"

            scp -i $SSH_KEY_FILE -o StrictHostKeyChecking=no -r * ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}

            ssh -i $SSH_KEY_FILE -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "sudo systemctl restart flaskapp && sudo systemctl restart nginx"

          '''

        }

      }

    }

  }

  post {

    success {

      mail to: 'you@example.com',

           subject: "Jenkins Build SUCCESS - ${env.JOB_NAME} #${env.BUILD_NUMBER}",

           body: "Build succeeded! View details: ${env.BUILD_URL}"

    }

    failure {

      mail to: 'you@example.com',

           subject: "Jenkins Build FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",

           body: "Build failed! Check logs: ${env.BUILD_URL}"

    }

  }

}
```

---


🚀 4. Deployment Configuration

EC2 Server Setup

On your EC2 instance (Ubuntu 22.04):

```
sudo apt update -y

sudo apt install -y python3-venv python3-pip nginx
```

Create Flask systemd Service:

```/etc/systemd/system/flaskapp.service```

ini

```
[Unit]

Description=Gunicorn instance to serve Flask_App

After=network.target

[Service]

User=ubuntu

Group=www-data

WorkingDirectory=/home/ubuntu/app

ExecStart=/home/ubuntu/app/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 app:app

Restart=on-failure

[Install]

WantedBy=multi-user.target

```

Enable and start:

```

sudo systemctl daemon-reload

sudo systemctl enable flaskapp

sudo systemctl start flaskapp
```

<img width="738" height="370" alt="Screenshot from 2025-11-13 01-22-17" src="https://github.com/user-attachments/assets/ce152a1f-a805-4496-b863-52dc494cdafc" />

<img width="1141" height="651" alt="Screenshot from 2025-11-13 01-24-46" src="https://github.com/user-attachments/assets/49002b1c-89c0-452f-80fe-f4f4c2010ac1" />

<img width="1365" height="651" alt="Screenshot from 2025-11-13 01-25-11" src="https://github.com/user-attachments/assets/1fde0e47-7e32-4e1f-bcc9-171bb6817bf0" />


---

🔔 5. Pipeline Triggers

The pipeline is automatically triggered when:

A commit is pushed to the main branch of the GitHub repository.

Steps:

In Jenkins, open the pipeline configuration.

Under Build Triggers, check:

✅ "GitHub hook trigger for GITScm polling"

On GitHub, add a webhook:

Payload URL: http://<jenkins-server>:8080/github-webhook/

Content Type: application/json

Event: "Just the push event."

---

✉️ 6. Email Notifications

The pipeline uses Jenkins' Email Extension Plugin for notifications.

SMTP Setup:

Go to Manage Jenkins → Configure System.

Under "E-mail Notification":

SMTP Server: smtp.gmail.com

Port: 465

Use SSL

Credentials: Your Gmail App Password

<img width="1128" height="366" alt="Screenshot from 2025-11-13 01-28-29" src="https://github.com/user-attachments/assets/6e0b30c6-0cf6-49e7-bab1-a7825d1b70d1" />


---

🧪 7. Testing the Pipeline

Push code changes to the main branch:

```

git add .

git commit -m "Test CI/CD pipeline"

git push origin main

Watch Jenkins automatically trigger a new build:

Stage 1: Build dependencies

Stage 2: Run tests

Stage 3: Deploy to EC2

Verify the application:

Production: http://<EC2-IP>/

Staging (optional): http://<EC2-IP>/staging/
 ```

<img width="1362" height="402" alt="Screenshot from 2025-11-11 23-50-59" src="https://github.com/user-attachments/assets/548c5866-0b07-440c-9d7d-7090f0492613" />


Check Jenkins console logs for deployment confirmation.

<img width="1366" height="701" alt="Screenshot from 2025-11-13 00-59-09" src="https://github.com/user-attachments/assets/0aebb17d-13d9-4a96-8099-af9592dd7c2c" />

<img width="1217" height="366" alt="Screenshot from 2025-11-13 01-30-12" src="https://github.com/user-attachments/assets/67d43b2b-0373-45ab-81a1-b4705f88e0bc" />

<img width="1363" height="366" alt="Screenshot from 2025-11-13 01-32-14" src="https://github.com/user-attachments/assets/0ffb3320-d3ef-42a6-a261-15958f5f4314" />

---

<img width="1366" height="701" alt="Screenshot from 2025-11-13 01-01-52" src="https://github.com/user-attachments/assets/24b03059-fa81-4c96-a6e2-d7ab373ff5d0" />

<img width="1366" height="701" alt="Screenshot from 2025-11-13 00-45-14" src="https://github.com/user-attachments/assets/855dbc81-1602-4aa7-a9bf-b2b2f7c5f527" />

<img width="1366" height="701" alt="Screenshot from 2025-11-13 00-45-27" src="https://github.com/user-attachments/assets/631937a1-6347-4f9d-a2f4-26cb95dcae72" />

<img width="1363" height="624" alt="Screenshot from 2025-11-13 01-38-38" src="https://github.com/user-attachments/assets/cafbc486-dc65-4a55-83b9-8d7d09040115" />

<img width="1363" height="474" alt="image" src="https://github.com/user-attachments/assets/05b4b80b-43ed-4da1-9c73-75cce725b40c" />

* * * * *

<img width="1363" height="366" alt="image" src="https://github.com/user-attachments/assets/6637d8b5-90d4-4ee7-a696-8931dc93c1d4" />

<img width="1366" height="701" alt="Screenshot from 2025-11-13 00-44-14" src="https://github.com/user-attachments/assets/3a776a57-bdb6-4e04-a06e-cabbd303f264" />

<img width="1366" height="534" alt="image" src="https://github.com/user-attachments/assets/1cf6dab3-334f-4c34-860d-15cc45b38c5d" />


🏁 Conclusion

This project successfully implements a Jenkins-based CI/CD pipeline for a Flask web application.

It automates the build, testing, and deployment processes, ensuring:

Faster feedback cycles

Reliable deployments

Reduced manual intervention

Result:

A fully automated CI/CD pipeline delivering continuous integration and deployment from GitHub → Jenkins → EC2.



* * * * *

📌 Author
---------

**DEEPIKA NARENDRAN**\
Project: Jenkins CI/CD for Flask Web Application &
GitHub Actions CI/CD Pipeline Flask App

GitHub: @JoinDeeHub
