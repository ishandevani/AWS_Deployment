# Task 3: Docker Deployment Using Amazon ECR, ECS, and VPC

## Objective
Deploy the Flask backend and Express frontend as Docker containers using Amazon ECR and Amazon ECS Fargate.

## Configuration used
- ECS cluster: `task3-fullstack-cluster`
- ECR repositories: `task3-frontend`, `task3-backend`
- ECS services: `task3-frontend-service`, `task3-backend-service`
- Task definition families: `frontend-task`, `backend-task`
- Service Connect namespace: `task3-namespace`
- Frontend port: `3000`
- Backend port: `5000`
- Frontend environment variable: `BACKEND_URL=http://backend:5000`
- Local image names: `sdfanfskdnf/fullstack-frontend:1.0`, `sdfanfskdnf/fullstack-backend:1.0`

## What we do
1. Build Docker images.
2. Create ECR repositories and push the images.
3. Create an ECS Fargate cluster and task definitions.
4. Deploy backend and frontend ECS services.
5. Configure VPC, subnets, security groups, and Service Connect.
6. Test the frontend and its connection to Flask.
7. Clean up unused resources after capturing screenshots.

## 1. Build Docker images

From the project directory containing `frontend/` and `backend/`:

```bash
docker build -t sdfanfskdnf/fullstack-backend:1.0 ./backend
docker build -t sdfanfskdnf/fullstack-frontend:1.0 ./frontend
docker images
```

If using Docker Compose:

```bash
docker compose build
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

## 2. Create ECR repositories

In AWS Console, open **Amazon ECR → Private repositories** and create:
- `task3-backend`
- `task3-frontend`

Copy each repository URI and confirm the correct AWS Region.

## 3. Log in and push to ECR

Replace `<AWS_REGION>` and `<AWS_ACCOUNT_ID>` with your own values. Run these commands on the machine that has the Docker images.

```bash
aws sts get-caller-identity
aws ecr get-login-password --region <AWS_REGION> | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com
```

Tag and push:

```bash
docker tag sdfanfskdnf/fullstack-backend:1.0 <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-backend:1.0
docker tag sdfanfskdnf/fullstack-frontend:1.0 <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-frontend:1.0

docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-backend:1.0
docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/task3-frontend:1.0
```

Verify the `1.0` image tag in both ECR repositories.

## 4. Create ECS cluster and task definitions

In Amazon ECS:
1. Create the Fargate cluster `task3-fullstack-cluster`.
2. Create task definition family `backend-task` using the `task3-backend:1.0` ECR image and container port `5000/TCP`.
3. Create task definition family `frontend-task` using the `task3-frontend:1.0` ECR image and container port `3000/TCP`.
4. Add `BACKEND_URL=http://backend:5000` to the frontend container environment.
5. Ensure the task execution role can pull ECR images and send logs to CloudWatch.
6. Configure CloudWatch logging if needed.

Do not place AWS access keys inside container environment variables.

## 5. VPC and security groups

Choose suitable VPC subnets and task networking. For assignment testing:
- Frontend security group: allow TCP `3000` from your public IP (`/32`).
- Backend security group: allow TCP `5000` from the frontend security group only.
- Avoid exposing backend port `5000` to the whole internet.

Tasks need outbound connectivity to ECR and CloudWatch, through NAT or suitable VPC endpoints.

## 6. Deploy backend service

In **ECS → Clusters → `task3-fullstack-cluster`**:
1. Create `task3-backend-service` from `backend-task`.
2. Choose Fargate and the configured VPC, subnets, and backend security group.
3. Set desired task count to `1`.
4. Configure Service Connect using namespace `task3-namespace`.
5. Configure backend discovery name `backend` and port mapping to container port `5000`.
6. Wait until the service and task are stable/running.

## 7. Deploy frontend service

1. Create `task3-frontend-service` from `frontend-task`.
2. Use the same ECS cluster and compatible VPC networking.
3. Enable Service Connect with namespace `task3-namespace` as appropriate for the frontend client.
4. Ensure the frontend security group permits TCP `3000` from your IP.
5. Confirm `BACKEND_URL=http://backend:5000` in the frontend task definition.
6. Wait until the service is stable.

If the frontend cannot reach Flask, inspect ECS service events, task logs, environment variables, port mappings, Service Connect names, and security groups.

## 8. Test

Open the frontend service's public IP/DNS with port `3000` if there is no load balancer. Submit a test user and verify the response. Check ECS task status, service events, and CloudWatch logs if there is a problem.

**Data storage note:** The Flask code used in this assignment stored the latest user in memory rather than a persistent database. A task/process restart may clear that data.

## 9. Cleanup

After screenshots, delete resources that are no longer needed:
1. Delete `task3-frontend-service` and wait for its tasks to stop.
2. Delete `task3-backend-service` and wait for its tasks to stop.
3. Delete ECS cluster `task3-fullstack-cluster`.
4. Delete ECR repositories if you no longer need the images.
5. Delete Service Connect namespace `task3-namespace` if unused.
6. Check for load balancers, NAT gateways, Elastic IPs/public IPv4 addresses, VPC endpoints, CloudWatch log groups, and custom security groups/VPCs before removing them.
7. Check Billing/Cost Explorer for any remaining resources or charges.

Deleting ECS services does not automatically remove every AWS resource. NAT gateways, load balancers, public IPv4 addresses, and other resources may still incur charges.

## Result
The Express frontend and Flask backend were deployed as separate ECS Fargate services. ECR stored the images, and Service Connect enabled internal service discovery between them.

## Screenshots
- ECR repositories and image tag `1.0`
- ECS cluster, services, and running tasks
- Task definitions and container ports
- Service Connect namespace/configuration
- Frontend environment variable
- Application test
- Cleanup evidence if required
