2048 Game – CI/CD on AWS (CodePipeline → CodeBuild → ECR → ECS Fargate)

A production-style CI/CD pipeline that builds, containers, and deploys the classic 2048 web game to AWS ECS Fargate, with images stored in Amazon ECR and builds driven by AWS CodePipeline/CodeBuild from GitHub pushes.

Live demo: Deployed to an ECS task with a public IP (HTTP:80).
Source repo: https://github.com/Arjunpuram22/2048-game-ci-cd
🧭 What I built
	•	Containerized a static Nginx site (the 2048 game) with a simple Dockerfile.
	•	Pushed images to Amazon ECR.
	•	Ran the container on ECS Fargate (serverless containers).
	•	Created a full CI/CD pipeline:
	•	Source: GitHub repo (this one)
	•	Build: CodeBuild executes buildspec.yml, builds & pushes Docker image to ECR
	•	Deploy: CodePipeline deploys to ECS service using imagedefinitions.json
	•	Every git push to main redeploys the site automatically.
🖼️ Architecture
  GitHub (main branch)
        │
        ▼
AWS CodePipeline ──────────────►  Stage 1: Source (GitHub)
        │
        ├──────────────────────►  Stage 2: Build (CodeBuild, Docker build/push to ECR)
        │
        └──────────────────────►  Stage 3: Deploy (ECS Fargate Service -> Task -> Public IP / ALB)
📂 Repo structure
  .
├── Dockerfile
├── buildspec.yml
├── index.html        # 2048 UI (edited to prove CI/CD)
├── js/               # game logic
├── style/            # CSS
└── meta/             # icons
✅ Prerequisites
	•	AWS account & CLI configured (aws configure) for your target region (e.g., us-east-1)
	•	Docker Desktop installed locally (for local build tests)
	•	GitHub repository (this repo)
	•	An ECR repository (e.g., 2048-game-repo)
	•	An ECS cluster & Fargate service that will run this image (container port 80)

Security note: In public docs, avoid publishing your account ID or public IPs if you don’t want to.
🧱 Dockerfile (Nginx)
# Use AWS public ECR nginx image to avoid Docker Hub rate limits
FROM public.ecr.aws/nginx/nginx:latest

# Copy the static site into nginx web root
COPY . /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
🧪 CodeBuild – buildspec.yml

This file drives the build & push to ECR and creates imagedefinitions.json for ECS deploy.

Replace <ACCOUNT_ID> and region as needed. Keep the ECS container name consistent with your task definition’s container name.
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

  build:
    commands:
      - echo Building the Docker image...
      - docker build -t 2048-game .
      - echo Tagging the Docker image...
      - docker tag 2048-game:latest <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/2048-game-repo:latest

  post_build:
    commands:
      - echo Pushing the Docker image to Amazon ECR...
      - docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/2048-game-repo:latest
      - echo Creating imagedefinitions.json for ECS deployment...
      - printf '[{"name":"2048-container","imageUri":"%s"}]' "<ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/2048-game-repo:latest" > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json

🐳 ECS task definition (key fields)
	•	Launch type: Fargate
	•	Network mode: awsvpc
	•	Container:
	•	Image: <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/2048-game-repo:latest
	•	Port mappings: container 80/tcp
	•	Service: 1 desired task, assign public IP = ENABLED, public subnet(s)

Security group inbound rule: allow TCP 80 from 0.0.0.0/0 (for a public demo).

🚀 CodePipeline (end-to-end)
	•	Source: GitHub (main branch)
	•	Build: CodeBuild project (privileged mode enabled for Docker)
	•	Deploy: ECS deploy action (cluster + service), artifact file = imagedefinitions.json

Execution mode: Superseded (if a new commit arrives, cancel the older run and deploy the newest).

Artifacts bucket: S3 bucket you own (e.g., 2048-game-artifacts-<region>-<random>).

🔄 Proving CI/CD (what I did)
	1.	Edited index.html (changed the title/heading, e.g., “2048 by ”).
	2.	Committed & pushed to GitHub main.
	3.	Observed CodePipeline run Source → Build → Deploy all green.
	4.	Verified the ECS task restarted with the latest image and the change was visible at the public IP on port 80.
