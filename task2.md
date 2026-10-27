# Task 2: Flask Backend and Express Frontend on Separate EC2 Instances

## Objective
Run the Express frontend and Flask backend on two separate EC2 instances.

## What we do
1. Launch two Ubuntu EC2 instances.
2. Install Python on the backend instance and Node.js on the frontend instance.
3. Run Flask on port 5000 and Express on port 3000.
4. Configure the frontend to call the backend using its private IP.
5. Restrict backend access with security groups.
6. Test the application from the browser.

> Replace placeholder IPs, paths, and repository details with the values from your actual deployment. Confirm details against your screenshots.

## 1. Connect to EC2

```bash
chmod 400 assignment-key.pem
ssh -i assignment-key.pem ubuntu@<FRONTEND_PUBLIC_IP>
ssh -i assignment-key.pem ubuntu@<BACKEND_PUBLIC_IP>
```

## 2. Backend EC2: install Python and run Flask

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv git
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_PROJECT_DIRECTORY>/backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m flask --app app run --host 0.0.0.0 --port 5000
```

Use the backend EC2 **private IP** for internal application traffic. Find addresses with:

```bash
hostname -I
```

Binding Flask to `0.0.0.0` permits network access; the backend security group must restrict port 5000 to the frontend security group.

## 3. Frontend EC2: install Node.js and run Express

```bash
sudo apt update
sudo apt install -y nodejs npm git
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_PROJECT_DIRECTORY>/frontend
npm install
node server.js
```

Or use the script in `package.json`:

```bash
npm start
```

Configure the frontend's backend URL to:

```text
http://<BACKEND_PRIVATE_IP>:5000
```

Use the environment variable/configuration method supported by your code.

## 4. Security groups

**Frontend security group**
- SSH `22`: your trusted IP only.
- Frontend port `3000`: your IP for testing, or required public access.

**Backend security group**
- SSH `22`: trusted administration path only.
- Flask port `5000`: source should be the frontend EC2 security group, not `0.0.0.0/0`.

Both instances need network connectivity inside the VPC. Prefer the backend private IP for application traffic.

## 5. Test

From the frontend instance:

```bash
curl http://<BACKEND_PRIVATE_IP>:5000/
curl -I http://127.0.0.1:3000
```

Use the actual API route if `/` is not a valid backend endpoint. Open:

```text
http://<FRONTEND_PUBLIC_IP>:3000
```

Submit a test user and verify the backend response.

## Result
The frontend and backend run on separate EC2 instances and communicate over the VPC using the backend's private IP.

## Screenshots
- Both EC2 instances running
- Flask backend terminal
- Express frontend terminal/browser
- Security group inbound rules
- Successful form submission/API response
