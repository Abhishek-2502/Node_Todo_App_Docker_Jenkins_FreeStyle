# Jenkins freestyle CI/CD Pipeline with Docker on AWS EC2

This guide provides step-by-step instructions to set up Jenkins freestyle CI/CD Pipeline on an AWS EC2 instance which involves configuration of SSH keys, integratation with GitHub, and deployment of Node.js application using Docker and Jenkins automation.

## Prerequisites
- An AWS EC2 instance (Ubuntu recommended)
- GitHub repository with application code

---

## 1. Install Jenkins and Docker on EC2
Follow the guide to install and set up Jenkins and Docker on your EC2 instance: [Jenkins_Docker_Setup_AWS](https://github.com/Abhishek-2502/Jenkins_Docker_Setup_AWS)

## 2. Configure SSH Keys
Generate and configure SSH keys for secure authentication on EC2 instance.

Command to generate secret key:
```bash
sudo ssh-keygen -t rsa -b 4096 -f ~/.ssh/github-deploy
```

Command to give permissions:
```bash
sudo chmod 600 ~/.ssh/authorized_keys
```

Command to see public key:
```bash
sudo cat ~/.ssh/github-deploy.pub  
```

Command for passwordless SSH login or deploy keys for GitHub, CI/CD, or remote servers.
```bash
sudo cat ~/.ssh/github-deploy.pub >> ~/.ssh/authorized_keys 
```

Command to see private key:
```bash
sudo cat ~/.ssh/github-deploy 
```

## 3. Configure GitHub SSH Key
1. Go to **GitHub → Settings → SSH and GPG keys → New SSH key**.
2. Enter a title (e.g., `jenkins-master`).
3. Key Type: `Authentication Key`.
4. Paste the public key in key section.

## 4. Create a Jenkins Project
1. Open Jenkins at `http://your_public_ip:8080`.
2. Click **New Item** and select **Freestyle Project**.
3. Enter project name (e.g., `node-todo-app`).
4. Click **OK**.

## 5. Configure Project
- **Description**: `Node todo app with freestyle pipeline`.
- **GitHub Project**: Add repository URL.
- **Source Code Management**: Select **Git**.
- **Repository URL**: Enter GitHub repo link.
- **Credentials**:
  - **Domain**: Global credentials (unrestricted)
  - **Kind**: SSH Username with private key
  - **Scope**: Global (Jenkins, nodes, items, all child items, etc)
  - **ID**: `github-jenkins`
  - **Description**: `This is for Jenkins and GitHub integration`
  - **Username**: `ubuntu` (EC2 username)
  - **Private Key**: Paste private key from `-----BEGIN OPENSSH PRIVATE KEY-----` to `-----END OPENSSH PRIVATE KEY-----`.
- **Branch**: Select the required branch.
- Leave other things as it is.

## 6. Build and Deploy Application
1. Click **Save**.
2. Click **Build Now**.
3. Check **Console Output** for execution status.
4. Navigate to Jenkins workspace path: `/var/lib/jenkins/workspace/node-todo-app`.
5. **Allow port 8000** in Security Group (Custom TCP, Anywhere-IPv4) for accessing node-todo-app.

## 7. Run Node.js App on EC2 (For Testing Purpose, can skip)

### Method 1
```bash
sudo apt-get update
```

```bash
sudo apt install nodejs -y
```

```bash
sudo apt install npm -y
```

```bash
sudo npm install -y
```

```bash
node app.js
```

### Method 2 (NVM)
NVM (Node Version Manager) allows you to install and manage multiple Node.js versions.

1. Install NVM:

```bash
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
source ~/.bashrc  # Load NVM
```

2. Install Node.js (Install latest LTS version) :

```bash
nvm install --lts
```

3. Verify installation:

```bash
node -v
npm -v
```

4. List available Node.js versions:

```bash
nvm ls-remote
```

5. Switch between installed versions:

```bash
nvm install 18
```


Verify the application at `http://your_public_ip:8000`.

---

## 8. Deploy with Docker (For Testing Purpose, can skip)

Build Docker container
```bash
docker build . -t node-todo-app:latest
```

Run Docker container
```bash
docker run -d --name node-todo-app-con -p 8000:8000 node-todo-app
```

See running Docker Container
```bash
docker ps
```

Stop Docker container
```bash
docker stop node-todo-app-con
```

Remove Docker container
```bash
docker rm node-todo-app-con
```
This will run the code through Docker while building container.

## 9. Automate Deployment via Jenkins
Add the following commands in Jenkins **Build Steps → Execute Shell**:

```bash
# Build a docker image
docker build . -t node-todo-app:latest

# Stop and remove the existing container if it exists
docker rm -f node-todo-app-con 2>/dev/null || true

# Run a new container
docker run -d --name node-todo-app-con -p 8000:8000 node-todo-app
```

---

## 10. Automate Build with Webhooks
1. Install **GitHub Integration Plugin** in Jenkins from **Manage Jenkins → Plugins → Available Plugins**.
2. Select **restart Jenkins when installation is complete and no jobs are running**.
3. Go to **GitHub Repo → Settings → Webhooks → Add Webhook**.
4. Set Payload URL: `http://your_public_ip:8080/github-webhook/`.
5. Content Type: `application/json`.
6. Select **Just the push event**.
7. Save and verify webhook with a green tick.
8. In **Jenkins → Project → Configure**:
   - Select **GitHub hook trigger for GITScm polling** under **Triggers**.

Now, any code push to GitHub will trigger an automatic build and deployment via Jenkins. If doesn't work then build manually from jenkins for one time.

#### NOTE: your_public_ip changes when you stop the EC2 instance. So update Payload URL accordingly

---

## Conclusion
You have successfully set up Jenkins on EC2, integrated it with GitHub using SSH keys, configured CI/CD with Docker, and automated deployments using webhooks.

Test your application at `http://your_public_ip:8000` and ensure the pipeline is working as expected.

## Author
Abhishek Rajput
