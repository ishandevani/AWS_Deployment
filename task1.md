# Task 1: Deploy Frontend and Backend on a Single EC2 Instance

## Objective

To deploy an HTML/CSS frontend and a Python Flask backend on a single Amazon EC2 instance using Nginx.

## Technologies Used

- Amazon EC2
- Ubuntu Linux
- HTML
- CSS
- Python
- Flask
- Nginx

## What I Did

1. Created an Ubuntu EC2 instance.
2. Connected to the instance using SSH.
3. Installed Python, pip, and Nginx.
4. Copied the frontend and backend project files to the instance.
5. Configured the Flask backend to run on port 5000.
6. Configured Nginx to serve the HTML and CSS files.
7. Configured Nginx to forward API requests to Flask.
8. Tested the website in a browser.

## Step 1: Connect to EC2

Set the correct permissions for your private key and connect to the EC2 instance.

```bash
chmod 400 assignment-key.pem
ssh -i assignment-key.pem ubuntu@<EC2_PUBLIC_IP>
```

Replace `<EC2_PUBLIC_IP>` with the public IP address of your EC2 instance.

## Step 2: Install Required Packages

Update the package list and install the required packages.

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv nginx git
```

Check the installations:

```bash
python3 --version
nginx -v
```

## Step 3: Copy the Project Files

### Option 1: Clone the Git Repository

If the project is stored in GitHub, clone the repository.

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_PROJECT_DIRECTORY>
```

### Option 2: Copy Files from Your Local Computer

Run this command from your local computer, where your project folders and private key are available.

```bash
scp -i assignment-key.pem -r frontend backend ubuntu@<EC2_PUBLIC_IP>:/home/ubuntu/
```

### Example Project Structure

```text
frontend/
├── index.html
└── style.css

backend/
├── app.py
└── requirements.txt
```

The actual filenames may differ depending on the project.

## Step 4: Set Up the Flask Backend

Navigate to the backend directory.

```bash
cd ~/backend
```

Create a Python virtual environment.

```bash
python3 -m venv venv
```

Activate the virtual environment.

```bash
source venv/bin/activate
```

Install the Python dependencies.

```bash
pip install -r requirements.txt
```

### Run the Flask Application

Start the Flask backend on port 5000.

```bash
python -m flask --app app run --host 127.0.0.1 --port 5000
```

This command assumes the Flask application is in `app.py` and the Flask object is named `app`.

The backend is now listening on `127.0.0.1:5000`, making it accessible locally on the EC2 instance.

**Note:** This command runs Flask in development mode. For a persistent production deployment, Gunicorn and a systemd service can be configured.

## Step 5: Copy Frontend Files to Nginx

Create a directory for the website.

```bash
sudo mkdir -p /var/www/fullstack-app
```

Copy the HTML and CSS files.

```bash
sudo cp -r ~/frontend/* /var/www/fullstack-app/
```

Set the ownership.

```bash
sudo chown -R www-data:www-data /var/www/fullstack-app
```

Verify that the files are present.

```bash
ls -la /var/www/fullstack-app
```

## Step 6: Configure Nginx

Open the Nginx configuration file.

```bash
sudo nano /etc/nginx/sites-available/fullstack-app
```

Add the following configuration.

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/fullstack-app;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:5000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Note:** Adjust the `/api/` route and proxy configuration according to the routes defined in the Flask application.

With the trailing slash in `proxy_pass`, Nginx removes the matching `/api/` prefix when forwarding the request. For example, `/api/users` is forwarded to Flask as `/users`.

### Enable the Nginx Site

Create a symbolic link to enable the configuration.

```bash
sudo ln -s /etc/nginx/sites-available/fullstack-app /etc/nginx/sites-enabled/fullstack-app
```

If the default Nginx site conflicts with this configuration, disable it only after confirming it is not needed.

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Test the Nginx configuration.

```bash
sudo nginx -t
```

If the configuration test succeeds, enable Nginx and reload the service.

```bash
sudo systemctl enable nginx
sudo systemctl reload nginx
```

Check the Nginx service status.

```bash
sudo systemctl status nginx
```

## Step 7: Configure the EC2 Security Group

Configure the inbound rules for the EC2 instance.

| Protocol | Port | Source |
|---|---:|---|
| SSH | 22 | Your trusted IP address |
| HTTP | 80 | `0.0.0.0/0` if public access is required |

Port 5000 does not need to be publicly accessible because Nginx communicates with Flask locally.

**Security note:** Restrict SSH access to your trusted IP address instead of allowing access from everywhere.

## Step 8: Test the Application

### Check Nginx

Test the configuration.

```bash
sudo nginx -t
```

Check the service status.

```bash
sudo systemctl status nginx
```

### Test the Frontend

Run the following command on the EC2 instance.

```bash
curl -I http://127.0.0.1
```

A successful response should normally return an HTTP status such as `200 OK`.

### Test the Flask Backend

If the Flask application has a valid route at `/`, test it with:

```bash
curl http://127.0.0.1:5000/
```

If the application uses a different route, replace `/` with that route.

### Open the Application in a Browser

Open the following address in your browser.

```text
http://<EC2_PUBLIC_IP>
```

Replace `<EC2_PUBLIC_IP>` with your EC2 instance's public IP address.

Verify that:

- The HTML page loads correctly.
- The CSS styling works.
- The frontend can send requests to the Flask backend.
- The backend processes requests successfully.
- Any submitted data is handled according to the application's implementation.

## Result

The HTML/CSS frontend is served by Nginx, and Nginx forwards the configured API requests to the Flask backend running on the same EC2 instance.

This deployment uses a single EC2 instance for both frontend and backend services.

## Conclusion

In this task, I deployed an HTML/CSS frontend and a Python Flask backend on a single Ubuntu EC2 instance. I used Nginx to serve the frontend and forward API requests to the backend. Finally, I tested the services and verified that the application was accessible through the EC2 public IP address.