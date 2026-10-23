# Deploy-a-Docker-Container-on-AWS
# Deploy a Docker Container on AWS: Dockerfile → Amazon ECR → ECS Fargate

A beginner-friendly, hands-on project that takes a simple Node.js app from a **Dockerfile** to a **running container on AWS**, with no servers to manage.


<img width="1600" height="900" alt="docker-ecr-fargate-cover" src="https://github.com/user-attachments/assets/0b09e9fd-c97a-4c1e-821e-e5d7980dea46" />



## Architecture

```
Dockerfile  ──►  Amazon ECR  ──►  Amazon ECS (Fargate)  ──►  Public IP :3000
 (build)        (store image)      (run container)
```

| Service | Role |
|---|---|
| **Docker** | Packages the app into an image |
| **Amazon ECR** | Stores the image in a private registry |
| **Amazon ECS + Fargate** | Runs the container serverlessly |

## Project Structure

```
.
├── app.js                # Express app (port 3000, /health endpoint)
├── package.json
├── Dockerfile            # node:20-alpine image
├── .dockerignore
├── deploy.sh             # Build + push image to ECR in one command
├── task-definition.json  # Sample Fargate task definition (CLI use)
└── README.md
```

## Prerequisites

- An AWS account (Free Tier is fine for learning)
- [Docker](https://docs.docker.com/get-docker/)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) configured (`aws configure`)

## Quick Start

### 1. Run locally

```bash
docker build -t my-docker-app:v1 .
docker run -d -p 3000:3000 my-docker-app:v1
```

Open http://localhost:3000. You should see: **Hello from Docker running on AWS Fargate!**

> **Apple Silicon tip:** build with `--platform linux/amd64` so the image runs on standard Fargate.

### 2. Push to Amazon ECR

Use the helper script (creates the repo if needed, logs in, builds, tags, pushes):

```bash
./deploy.sh v1
```

Or do it manually (replace `<account-id>`):

```bash
aws ecr create-repository --repository-name my-docker-app --region ap-south-1

aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-south-1.amazonaws.com

docker tag my-docker-app:v1 <account-id>.dkr.ecr.ap-south-1.amazonaws.com/my-docker-app:v1
docker push <account-id>.dkr.ecr.ap-south-1.amazonaws.com/my-docker-app:v1
```

### 3. Run it on ECS Fargate (console)

1. **ECS → Clusters → Create cluster** → name `docker-demo-cluster`, keep **Fargate** selected
2. **Task definitions → Create new** → Fargate, **0.25 vCPU / 0.5 GB**, execution role `ecsTaskExecutionRole`, container port `3000`, image URI from ECR
3. **Cluster → Services → Create** → Launch type **Fargate**, desired tasks `1`, public subnet, **Public IP on**, security group allowing **inbound TCP 3000**
4. Open the task, copy its **Public IP**, and visit `http://<public-ip>:3000`

### Optional: use the CLI instead of the console

Edit the placeholders in `task-definition.json`, create the log group, then register the task:

```bash
aws logs create-log-group --log-group-name /ecs/my-docker-app-task --region ap-south-1
aws ecs register-task-definition --cli-input-json file://task-definition.json --region ap-south-1
```

## Troubleshooting

| Problem | Likely cause |
|---|---|
| Task stops immediately | Check CloudWatch logs and confirm `CMD` in the Dockerfile |
| Can't pull the image | Execution role missing ECR permissions, or wrong image URI |
| Page doesn't load | Security group not allowing port 3000, or Public IP disabled |
| `exec format error` | Image built for the wrong CPU architecture (use `--platform linux/amd64`) |

## Best Practices

- Use versioned tags (`v1`, `v2`), not `latest`, in production
- Turn on **ECR image scanning**
- Use **least-privilege IAM roles** for tasks
- Put an **Application Load Balancer** in front for real workloads (`/health` is ready for health checks)
- Send logs to **CloudWatch** and alarm on failures

## Clean Up (avoid charges)

1. Set the ECS service desired count to `0`, then delete the service
2. Delete the cluster
3. Delete the ECR repository (or image)
4. Delete any demo security groups

## Read the Full Article

The step-by-step walkthrough is on my dev.to profile: https://dev.to/vishal_patil_aws

## Author

**Vishal Patil**

- LinkedIn: https://www.linkedin.com/in/vishalpatil-devops-aws/
- dev.to: https://dev.to/vishal_patil_aws
- GitHub: https://github.com/VishalPatil-DevOps-AWS

#vishalcloudjourney #AWS #Docker #ECS #Fargate #ECR #DevOps

## License

[MIT](LICENSE)
