# End-to-End MLOps Capstone: MLflow, DVC, Flask, Docker, CI/CD, EKS, Prometheus, and Grafana

A production-style machine learning project template that demonstrates how to build, track, version, package, deploy, and monitor an ML system from experimentation to cloud deployment.

## Overview

This project follows a full MLOps workflow:

- Structured project setup using Cookiecutter Data Science
- Experiment tracking with **DagsHub + MLflow**
- Data and pipeline versioning with **DVC**
- Modular ML code in `src/`
- Model serving with **Flask**
- Containerization with **Docker**
- CI/CD with **GitHub Actions**
- Image publishing to **Amazon ECR**
- Deployment to **Amazon EKS**
- Monitoring with **Prometheus** and **Grafana**

## Architecture

```text
Local Development
    |
    v
GitHub Repository
    |
    +--> DagsHub Repo Connection
    |        |
    |        +--> MLflow Experiment Tracking
    |
    +--> GitHub Actions CI/CD
             |
             +--> Build Docker Image
             +--> Push to Amazon ECR
             +--> Deploy to Amazon EKS
                              |
                              +--> Flask Inference App
                              +--> LoadBalancer Service
                                              |
                                              +--> Prometheus Scraping
                                              +--> Grafana Dashboards
```

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── ci.yaml
├── data/
│   ├── external/
│   ├── interim/
│   ├── processed/
│   └── raw/
├── flask_app/
├── local_s3/
├── notebooks/
├── reports/
│   └── figures/
├── src/
│   ├── logger/
│   ├── model/
│   ├── data_ingestion.py
│   ├── data_preprocessing.py
│   ├── feature_engineering.py
│   ├── model_building.py
│   ├── model_evaluation.py
│   └── register_model.py
├── tests/
├── scripts/
├── dvc.yaml
├── params.yaml
├── requirements.txt
├── Dockerfile
└── README.md
```

## Tech Stack

- **Python 3.10**
- **Conda**
- **Cookiecutter Data Science**
- **MLflow**
- **DagsHub**
- **DVC**
- **Flask**
- **Docker**
- **GitHub Actions**
- **AWS S3**
- **AWS ECR**
- **AWS EKS**
- **Prometheus**
- **Grafana**

## Setup Instructions

### 1. Create the repository

Create a new GitHub repository and clone it locally:

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### 2. Create and activate environment

```bash
conda create -n atlas python=3.10 -y
conda activate atlas
```

### 3. Install Cookiecutter and generate project template

```bash
pip install cookiecutter
cookiecutter -c v1 https://github.com/drivendataorg/cookiecutter-data-science
```

> Note: Cookiecutter Data Science v1 is still available, although the maintainers recommend v2 for newer projects.

### 4. Rename module folder

Rename:

```text
src/models  -->  src/model
```

### 5. Initial Git commit

```bash
git add .
git commit -m "Initial project structure setup"
git push origin main
```

## Experiment Tracking with DagsHub + MLflow

DagsHub provides a hosted MLflow-compatible experiment tracking server for connected repositories.

### 1. Connect repository to DagsHub

1. Go to [DagsHub Dashboard](https://dagshub.com/dashboard)
2. Create a new repo
3. Connect your GitHub repository
4. Open the repository on DagsHub
5. Copy:
   - Experiment tracking URL
   - Repository owner name
   - Repository name
   - MLflow setup snippet

### 2. Install dependencies

```bash
pip install dagshub mlflow
```

### 3. Example MLflow setup

```python
import dagshub
import mlflow

dagshub.init(repo_owner="<repo-owner>", repo_name="<repo-name>")
mlflow.autolog()

with mlflow.start_run():
    # training code here
    pass
```

### 4. Run notebooks or experiments

Run the experiment notebooks/scripts and verify that metrics, params, and artifacts appear in DagsHub.

```bash
git add .
git commit -m "Add MLflow experiment tracking"
git push origin main
```

## DVC Pipeline Setup

DVC supports local remotes and cloud remotes such as Amazon S3, and the default remote can be set with `dvc remote add -d`.[web:5][web:6]

### 1. Initialize DVC

```bash
dvc init
```

### 2. Create temporary local remote

```bash
mkdir local_s3
dvc remote add -d mylocal local_s3
```

### 3. Add pipeline code

Implement the following inside `src/`:

- `logger/`
- `data_ingestion.py`
- `data_preprocessing.py`
- `feature_engineering.py`
- `model_building.py`
- `model_evaluation.py`
- `register_model.py`

### 4. Add pipeline config files

Create:

- `dvc.yaml`
- `params.yaml`

### 5. Run pipeline

```bash
dvc repro
dvc status
```

### 6. Commit pipeline

```bash
git add .
git commit -m "Add DVC pipeline"
git push origin main
```

## DVC Remote Storage on S3

DVC supports S3 remotes, and its documentation notes that S3 support may require installing the `dvc[s3]` extra before adding the remote.[web:4][web:6]

### 1. Create AWS resources

Create:

- An IAM user
- An S3 bucket

Keep the AWS credentials secure.

### 2. Install AWS and DVC S3 support

```bash
pip install "dvc[s3]" awscli
```

### 3. Optional remote management

```bash
dvc remote list
dvc remote remove <name>
```

### 4. Configure AWS CLI

```bash
aws configure
```

### 5. Add S3 as DVC remote

```bash
dvc remote add -d myremote s3://<bucket-name>
```

### 6. Push tracked data

```bash
dvc push
```

## Flask Inference App

### 1. Create app directory

```bash
mkdir flask_app
```

Add the Flask serving files and supporting assets inside `flask_app/`.

### 2. Install Flask

```bash
pip install flask
```

### 3. Run the app

```bash
python app.py
```

### 4. Freeze dependencies

```bash
pip freeze > requirements.txt
```

## CI/CD with GitHub Actions

### 1. Add workflow

Create:

```text
.github/workflows/ci.yaml
```

### 2. Add test and utility directories

Create:

```text
tests/
scripts/
```

### 3. Add DagsHub token

Generate a DagsHub access token from your repository settings and store it in GitHub Secrets.

**Recommended GitHub Secrets / Variables:**

- `CAPSTONE_TEST`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`

> Never hardcode secrets directly in the repository.

## Dockerization

### 1. Generate app-specific requirements

```bash
pip install pipreqs
cd flask_app
pipreqs . --force
cd ..
```

### 2. Add Dockerfile

Create a `Dockerfile` in the project root.

### 3. Build image

```bash
docker build -t capstone-app:latest .
```

### 4. Run container

Without env variable:

```bash
docker run -p 8888:5000 capstone-app:latest
```

With env variable:

```bash
docker run -p 8888:5000 -e CAPSTONE_TEST=<your-token> capstone-app:latest
```

### 5. Optional Docker Hub push

```bash
docker tag capstone-app:latest <dockerhub-username>/capstone-app:latest
docker push <dockerhub-username>/capstone-app:latest
```

## AWS ECR Setup

### Required GitHub Secrets / Variables

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`

### IAM permission

Attach:

```text
AmazonEC2ContainerRegistryFullAccess
```

### Goal

Run the CI/CD pipeline until the Docker image is built and pushed to **Amazon ECR**.

## AWS CLI / kubectl / eksctl Setup

If `aws` resolves to a Python or Anaconda installation, remove the conflicting package and use the official AWS CLI v2 installation instead.

### PowerShell checks

```powershell
Get-Command aws
aws --version
kubectl version --client
eksctl version
```

### Remove conflicting Python AWS CLI

```bash
pip uninstall awscli
```

### Install kubectl

```powershell
Invoke-WebRequest -Uri "https://dl.k8s.io/release/v1.28.2/bin/windows/amd64/kubectl.exe" -OutFile "kubectl.exe"
Move-Item -Path .\kubectl.exe -Destination "C:\Windows\System32"
kubectl version --client
```

### Install eksctl

```powershell
Invoke-WebRequest -Uri "https://github.com/weaveworks/eksctl/releases/download/v0.158.0/eksctl_Windows_amd64.zip" -OutFile "eksctl.zip"
Expand-Archive -Path .\eksctl.zip -DestinationPath .
Move-Item -Path .\eksctl.exe -Destination "C:\Windows\System32\eksctl.exe"
eksctl version
```

## Amazon EKS Deployment

### 1. Create cluster

```bash
eksctl create cluster --name flask-app-cluster --region us-east-1 --nodegroup-name flask-app-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 1 --managed
```

### 2. Update kubeconfig

```bash
aws eks --region us-east-1 update-kubeconfig --name flask-app-cluster
```

### 3. Verify cluster

```bash
aws eks list-clusters
aws eks --region us-east-1 describe-cluster --name flask-app-cluster --query "cluster.status"
kubectl get nodes
kubectl get namespaces
kubectl get pods
kubectl get svc
```

### 4. Optional delete

```bash
eksctl delete cluster --name flask-app-cluster --region us-east-1
eksctl get cluster --region us-east-1
```

### 5. Deploy through CI/CD

Update:

- `ci.yaml`
- `deployment.yaml`
- `Dockerfile`

Also update the node security group inbound rules to allow the required app port.

### 6. Access service

```bash
kubectl get svc flask-app-service
curl http://<external-ip>:5000
```

## Monitoring with Prometheus

### EC2 instance recommendation

Launch an Ubuntu EC2 instance with:

- Instance type: `t3.medium`
- Disk: `20GB`
- Open ports:
  - `22` for SSH
  - `9090` for Prometheus

### Install Prometheus

```bash
sudo apt update && sudo apt upgrade -y
wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.46.0.linux-amd64.tar.gz
mv prometheus-2.46.0.linux-amd64 prometheus
sudo mv prometheus /etc/prometheus
sudo mv /etc/prometheus/prometheus /usr/local/bin/
```

### Configure Prometheus

Edit:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Example:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "flask-app"
    static_configs:
      - targets: ["<external-load-balancer-dns>:5000"]
```

Verify:

```bash
cat /etc/prometheus/prometheus.yml
which prometheus
/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml
```

## Monitoring with Grafana

### EC2 instance recommendation

Launch another Ubuntu EC2 instance with:

- Instance type: `t3.medium`
- Disk: `20GB`
- Open ports:
  - `22` for SSH
  - `3000` for Grafana

### Install Grafana

```bash
sudo apt update && sudo apt upgrade -y
wget https://dl.grafana.com/oss/release/grafana_10.1.5_amd64.deb
sudo apt install ./grafana_10.1.5_amd64.deb -y
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

### Open Grafana

```text
http://<ec2-public-ip>:3000
```

Default login:

```text
username: admin
password: admin
```

### Add Prometheus datasource

Use your Prometheus server URL, for example:

```text
http://<prometheus-ec2-public-ip>:9090
```

Then click **Save and Test** and start building dashboards.

## Common Commands

### Git

```bash
git add .
git commit -m "your message"
git push origin main
```

### DVC

```bash
dvc repro
dvc status
dvc push
dvc pull
dvc remote list
```

### Docker

```bash
docker build -t capstone-app:latest .
docker run -p 8888:5000 capstone-app:latest
```

### Kubernetes

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
kubectl get namespaces
```

## AWS Cleanup

To avoid unnecessary charges, clean up resources after testing.

### Kubernetes cleanup

```bash
kubectl delete deployment flask-app
kubectl delete service flask-app-service
kubectl delete secret capstone-secret
```

### EKS cleanup

```bash
eksctl delete cluster --name flask-app-cluster --region us-east-1
eksctl get cluster --region us-east-1
```

### Additional cleanup

- Delete unused ECR images
- Delete S3 artifacts if no longer needed
- Confirm CloudFormation stack deletion
- Verify that EC2 instances are terminated
- Confirm no billable services remain active

## Security Notes

- Do not commit AWS keys, DagsHub tokens, or `.env` values.
- Store secrets only in GitHub Secrets or your local environment.
- Use least-privilege IAM policies wherever possible.
- Rotate credentials after demos or team handoffs.

## Future Improvements

- Add unit and integration test coverage
- Add model performance regression checks in CI
- Add schema validation for incoming data
- Add canary or blue-green deployment
- Add structured logging and alerting
- Replace static Prometheus target config with service discovery

## Acknowledgements

This project structure is inspired by:

- Cookiecutter Data Science
- DagsHub MLflow integration
- DVC remote storage workflows
- AWS cloud-native deployment patterns

## License

This project is intended for educational, research, and portfolio use unless stated otherwise.