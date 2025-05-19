# Flask Web App with Docker and CI/CD to AWS ECS

This project demonstrates how to build a simple Python Flask web application, containerize it using Docker, and deploy it to **Amazon ECS (Elastic Container Service)** using a CI/CD pipeline with **GitHub Actions**.

---

## 🚀 Features

- Simple Flask web server (`app.py`)
- Dockerized application
- GitHub Actions CI/CD pipeline:
  - Builds and pushes Docker image to Docker Hub
  - Updates ECS Task Definition and deploys to ECS
- Uses AWS Application Load Balancer (ALB) for routing

---

## 🧱 Project Structure

my-flask-app/
│
├── app.py # Main Flask application
├── Dockerfile # Dockerfile to build the container
├── requirements.txt # Python dependencies
├── .github/
│ └── workflows/
│ └── deploy-to-ecs.yml # GitHub Actions workflow for CI/CD
└── README.md # Project documentation


---

## 🔧 Prerequisites

- Docker installed locally
- Python 3.9+
- AWS account with:
  - ECS Cluster and Service (Fargate)
  - ALB and Target Group
  - IAM user with ECS, ECR, and EC2 permissions
- GitHub repository with following secrets configured:

| Secret Name             | Description                           |
|-------------------------|---------------------------------------|
| `DOCKERHUB_USERNAME`    | Your Docker Hub username              |
| `DOCKERHUB_PASSWORD`    | Your Docker Hub password or PAT       |
| `AWS_ACCESS_KEY_ID`     | AWS access key                        |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key                        |
| `AWS_REGION`            | AWS region (e.g., `us-east-1`)        |
| `ECS_CLUSTER_NAME`      | Name of the ECS cluster               |
| `ECS_SERVICE_NAME`      | Name of the ECS service               |
| `ECS_TASK_FAMILY`       | Task definition family name           |
| `CONTAINER_NAME`        | Container name in the task definition |

---

## 🐍 Run Locally

```bash
# Create virtual environment (optional)
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run the Flask app
python app.py

Visit http://localhost:5000
🐳 Docker
Build and run the container locally

docker build -t my-flask-app .
docker run -d -p 5000:5000 my-flask-app

🚀 CI/CD Pipeline with GitHub Actions
Workflow Summary

    Triggers on push to main branch

    Builds and pushes Docker image to Docker Hub

    Updates ECS Task Definition

    Deploys new revision to ECS service

File: .github/workflows/deploy-to-ecs.yml

name: Build and Deploy to Amazon ECS

on:
  push:
    branches:
      - main

env:
  IMAGE_NAME: ${{ secrets.DOCKERHUB_USERNAME }}/my-flask-app:latest

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKERHUB_PASSWORD }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

      - name: Build Docker image
        run: docker build -t $IMAGE_NAME .

      - name: Push Docker image
        run: docker push $IMAGE_NAME

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Get current task definition
        id: task-def
        run: |
          aws ecs describe-task-definition \
            --task-definition ${{ secrets.ECS_TASK_FAMILY }} \
            --query taskDefinition > task-def.json

      - name: Replace image in task definition
        run: |
          IMAGE="${{ env.IMAGE_NAME }}"
          jq '.containerDefinitions[0].image = "'$IMAGE'"' task-def.json > new-task-def.json

      - name: Register new task definition
        run: |
          TASK_DEF_ARN=$(aws ecs register-task-definition \
            --cli-input-json file://new-task-def.json \
            --query 'taskDefinition.taskDefinitionArn' \
            --output text)
          echo "TASK_DEF_ARN=$TASK_DEF_ARN" >> $GITHUB_ENV

      - name: Update ECS Service
        run: |
          aws ecs update-service \
            --cluster ${{ secrets.ECS_CLUSTER_NAME }} \
            --service ${{ secrets.ECS_SERVICE_NAME }} \
            --task-definition $TASK_DEF_ARN

🌐 Access Your App

Once deployed to ECS with an Application Load Balancer (ALB), use the ALB DNS name (e.g. http://my-load-balancer-123456789.us-east-1.elb.amazonaws.com) to access the app.
🧼 Cleanup

To avoid costs:

    Stop/delete ECS services and clusters

    Deregister task definitions

    Delete ALB and Target Group

📚 Resources

    Flask Documentation

    Docker Docs

    AWS ECS Docs

    GitHub Actions Docs

