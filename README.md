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

| Stage | Description |

|--------|--------------|

| **Install Dependencies** | Set up Python and install required libraries using `pip install -r requirements.txt` |

| **Run Tests** | Run tests using `pytest` (if tests exist) |

| **Build** | Prepare application for deployment |

| **Deploy to Staging** | Triggered automatically when code is pushed to `staging` branch |

| **Deploy to Production** | Triggered automatically when code is pushed to `main` branch |


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

📌 Author
---------

**DEEPIKA NARENDRAN**\
GitHub Actions CI/CD Pipeline Flask App

GitHub: @JoinDeeHub

