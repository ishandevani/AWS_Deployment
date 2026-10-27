# AWS Full-Stack Application Deployment Assignment

This repository documents three approaches for deploying a Node.js/Express frontend and Python/Flask backend.

## Tasks

| File | Task | Main tools/services |
|---|---|---|
| [task1.md](task1.md) | Frontend and backend on one EC2 instance | EC2, Ubuntu, Flask, Nginx |
| [task2.md](task2.md) | Frontend and backend on separate EC2 instances | EC2, VPC, security groups, Flask, Express |
| [task3.md](task3.md) | Container deployment | Docker, Amazon ECR, Amazon ECS Fargate, VPC, Service Connect, CloudWatch |

## Application details

- Frontend: Node.js/Express, port `3000`
- Backend: Python/Flask, port `5000`
- Local images: `sdfanfskdnf/fullstack-frontend:1.0`, `sdfanfskdnf/fullstack-backend:1.0`
- ECR repositories: `task3-frontend`, `task3-backend`
- ECS cluster: `task3-fullstack-cluster`
- ECS services: `task3-frontend-service`, `task3-backend-service`
- Service Connect namespace: `task3-namespace`
- Frontend backend URL: `BACKEND_URL=http://backend:5000`

## Main commands

### Connect to EC2

```bash
chmod 400 assignment-key.pem
ssh -i assignment-key.pem ubuntu@<EC2_PUBLIC_IP>
```

### Install packages on Ubuntu

```bash
sudo apt update
sudo apt install -y git python3 python3-pip python3-venv nodejs npm nginx
```

### Run Flask

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m flask --app app run --host 127.0.0.1 --port 5000
```

For separate-instance networking, only bind Flask to `0.0.0.0:5000` if required and restrict access using the backend security group.

### Run Express

```bash
cd frontend
npm install
node server.js
```

Use `npm start` if that is the start script defined by the project.

### Build Docker images

```bash
docker build -t sdfanfskdnf/fullstack-backend:1.0 ./backend
docker build -t sdfanfskdnf/fullstack-frontend:1.0 ./frontend
docker images
```

### Push images to ECR

Replace the AWS account ID and Region placeholders:

```bash
aws sts get-caller-identity
aws ecr get-login-password --region <AWS_REGION> | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com

docker tag sdfanfskdnf/fullstack-backend:1.0 <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-backend:1.0
docker tag sdfanfskdnf/fullstack-frontend:1.0 <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-frontend:1.0

docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-backend:1.0
docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-frontend:1.0
```

### Docker Compose (local testing, if configured)

```bash
docker compose build
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

## Security reminders

- Restrict SSH access to a trusted IP.
- Prefer private IP communication between frontend and backend.
- Do not expose backend port `5000` to the entire internet.
- Configure ECS security groups and Service Connect names/ports consistently.
- Never commit `.pem` files, AWS credentials, passwords, or other secrets to GitHub.

## Data storage note

The Flask implementation used in this assignment stores the latest user in memory rather than a persistent database. A process or task restart can clear that data. Use a database or another durable store if persistent data is required.

## AWS cleanup and cost

After capturing screenshots, delete unused ECS services, the ECS cluster, ECR repositories if no longer required, and other resources created for the assignment. Also check NAT gateways, load balancers, Elastic IPs/public IPv4 addresses, VPC endpoints, CloudWatch log groups, and custom networking resources. Some resources can continue to incur charges even after ECS services are deleted.