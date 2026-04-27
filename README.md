# EC2 Deployment Guide (MERN Stack)

This guide documents the steps taken to deploy this MERN stack application on an AWS EC2 instance running Ubuntu. It covers everything from connecting to the instance to keeping the backend running continuously using PM2.

## Prerequisites
* An AWS EC2 Instance running **Ubuntu 22.04 LTS or 24.04 LTS**.
* Security Group configured to allow inbound traffic on **Port 22 (SSH)** and **Port 5000 (Custom TCP)**.
* Your `.pem` key pair downloaded to your local machine.

---

## Step 1: Connect to the EC2 Instance

If you are using **Windows**, you must first fix the permissions of your `.pem` key file before SSH will accept it. Open PowerShell in the folder where your key is downloaded and run:

```powershell
# Remove default inherited permissions
icacls.exe your-key.pem /inheritance:r

# Grant Read access only to your Windows user
icacls.exe your-key.pem /grant:r "$($env:USERNAME):(R)"
Once permissions are fixed, connect to your instance:

Bash
ssh -i "your-key.pem" ubuntu@<your-ec2-public-ip-or-dns>
Step 2: Install Required Dependencies
Important: Vite and its underlying bundlers require Node.js v20.19+ or v22+. We will install Node.js v22 to ensure the frontend builds correctly without CustomEvent or missing native binding errors.

Update the system and install Node.js v22, Git, and PM2:

Bash
# 1. Update package lists
sudo apt update && sudo apt upgrade -y

# 2. Download and install Node.js v22
curl -fsSL [https://deb.nodesource.com/setup_22.x](https://deb.nodesource.com/setup_22.x) | sudo -E bash -
sudo apt install -y nodejs

# 3. Verify Node version (Should output v22.x.x)
node -v

# 4. Install Git
sudo apt install -y git

# 5. Install PM2 globally (Process Manager for Node.js)
sudo npm install -g pm2
Step 3: Clone the Repository
Bring the project code onto the EC2 instance:

Bash
git clone <your-github-repo-url>
cd <your-repository-folder-name>
Step 4: Build the Frontend
Because the backend is configured to serve the static frontend files in production, we need to build the React/Vite app first.

Bash
# Navigate to the frontend directory
cd frontend

# Clean cache and install dependencies fresh (prevents OS binary conflicts)
npm cache clean --force
npm install

# Build the project (generates the /dist folder)
npm run build
Step 5: Configure the Backend
Navigate to the backend directory and set up the environment variables:

Bash
# Navigate back to the root, then into the backend
cd ../backend

# Install backend dependencies
npm install

# Create the environment variables file
nano .env
Paste your configuration into the .env file:

Code snippet
NODE_ENV=production
PORT=5000
MONGO_URI=your_mongodb_connection_string
(Save and exit by pressing Ctrl+O, Enter, then Ctrl+X)

Step 6: Start the Server with PM2
To keep the Node.js application running in the background even after you close the SSH terminal, use PM2:

Bash
# Start the server
pm2 start server.js --name "ecommerce-app"

# Configure PM2 to auto-restart the app if the EC2 instance reboots
pm2 startup ubuntu
(Run the specific command PM2 outputs to your console, it will look something like sudo env PATH=$PATH:/usr/bin...)

Bash
# Save the current PM2 process list
pm2 save
