# 🎮 2048 Game – CI/CD on AWS (CodePipeline → CodeBuild → ECR → ECS Fargate)

A **production-grade CI/CD pipeline** that builds, containerizes, and deploys the classic **2048 web game** to **AWS ECS Fargate**, with container images stored in **Amazon ECR** and builds triggered automatically from **GitHub pushes** using **AWS CodePipeline** and **CodeBuild**.

---

## 🌐 Live Demo
Deployed to an ECS Fargate task with a public IP (HTTP:80).

**Source Repository:**  
🔗 [https://github.com/Arjunpuram22/2048-game-ci-cd](https://github.com/Arjunpuram22/2048-game-ci-cd)

---

## 🧭 What I Built

- 🧱 **Containerized** a static Nginx site (the 2048 game) with a simple Dockerfile  
- 🚀 **Pushed** Docker images to **Amazon ECR**  
- ☁️ **Deployed** the container on **ECS Fargate** (serverless containers)  
- 🔁 **Automated** the full deployment lifecycle via **AWS CodePipeline**:
  - **Source:** GitHub repository (this one)  
  - **Build:** CodeBuild executes `buildspec.yml`, builds & pushes Docker image to ECR  
  - **Deploy:** CodePipeline deploys to ECS using `imagedefinitions.json`  
- 🔄 Every Git push to `main` branch automatically redeploys the latest version

---

## 🖼️ Architecture

![CI/CD Architecture – 2048 Game](architecture.png)
