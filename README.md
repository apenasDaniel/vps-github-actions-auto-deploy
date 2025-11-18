

# Automating Website Deployment Using GitHub Actions

This guide provides step-by-step instructions to set up automated deployment for a static website (HTML, CSS, JS) hosted on a VPS using GitHub Actions. By the end of this guide, every push to your repository will trigger a workflow that automatically deploys your changes to the VPS.

## Prerequisites
- A VPS running Ubuntu with an accessible SSH port.
- A website hosted on your VPS at `/var/www/html/html-css-js`.
- A GitHub repository containing the website's code.
- Basic knowledge of SSH and command-line operations.

## Steps Overview
1. **Generate SSH Keys**: Create an SSH key pair for secure communication between GitHub Actions and your VPS.
2. **Copy SSH Key to VPS**: Add the public key to the VPS to allow passwordless access.
3. **Add SSH Private Key and Server Details to GitHub Secrets**: Add sensitive data to GitHub Secrets to keep your server secure.
4. **Create GitHub Actions Workflow**: Set up a workflow to automate the deployment process.
5. **Push Changes and Test Deployment**: Push the workflow file and verify automatic deployment.

## Step-by-Step Guide

### 1. Generate SSH Keys
First, create an SSH key pair on your local machine to facilitate secure communication between GitHub Actions and your VPS.

```bash
ssh-keygen -t ed25519 -C "<email@example.com>"
```

- Enter a filename for the key `github-actions-vps`
- Leave the passphrase empty by pressing Enter twice.

### 2. Copy SSH Key to VPS
Copy the public SSH key (`github-actions-vps.pub`) to the VPS to allow GitHub Actions to connect.

```bash
ssh-copy-id -i ~/.ssh/github-actions-vps.pub -p 22 danieloliveiradev@<YOUR_VPS_IP>
```

- Replace `<YOUR_VPS_IP>` with the IP address of your VPS.
- If using a different SSH port, make sure to specify it using the `-p` flag (as shown with `22`).

### 3. Add SSH Private Key and Server Details to GitHub Secrets
**Add the SSH private key to GitHub so the workflow can use it.**

1. Open your GitHub repository.
2. Go to **Settings > Secrets and variables > Actions**.
3. Click on **New repository secret**.
4. Name the secret `DEPLOY_SSH_KEY`.
5. Paste the content of `~/.ssh/github-actions-vps` (the private key) into the value field.
6. Click **Add secret**.

**Repeat this process for your web server details**

`WEB_HOST_IP_ADDRESS` - The IP address of your VPS.

`WEB_HOST_USERNAME` - The username for the user you login to the server with.

`WEB_HOST_DIRECTORY` - The directory from which your website gets served (`/var/www/html` is usually the default location).

### 4. Create GitHub Actions Workflow
Create a workflow file to automate deployment on push.

1. In the root of your project, create a directory named `.github/workflows`.
2. Inside the `workflows` folder, create a file named `deploy.yml`.
3. Add the following content to `deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Copy files via SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.DEPLOY_SSH_KEY }}
          HOST_IP_ADDRESS: ${{ secrets.WEB_HOST_IP_ADDRESS }}
          HOST_USERNAME: ${{ secrets.WEB_HOST_USERNAME }}
          HOST_DIRECTORY: ${{ secrets.WEB_HOST_DIRECTORY }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          rsync -avz --delete ./public/ $HOST_USERNAME@$HOST_IP_ADDRESS:$HOST_DIRECTORY -e "ssh -p 22 -o StrictHostKeyChecking=no"
```

### 5. Push Changes and Test Deployment
Push your changes to GitHub to trigger the workflow.

```bash
git add .
git commit -m "Add GitHub Actions workflow for automated deployment"
git push origin main
```

1. Navigate to the **Actions** tab in your GitHub repository.
2. Verify that the "Deploy" workflow runs successfully.

### 6. Verify on the VPS
After the workflow completes, SSH into your VPS to verify that the files have been updated.

```bash
ssh <username>@<VPS_IP_ADDRESS>
cd <HOST_DIRECTORY>
ls -la
```

You should see that the files have been updated with the latest changes.

### Troubleshooting
- **Permission Denied:** If you encounter a `Permission denied` error during the workflow execution, ensure that the SSH key has been correctly added to the `authorized_keys` on the VPS.
- **File Permissions:** Ensure that the user on the VPS has the correct permissions for the project directory. You can change the ownership and permissions using:
  
  ```bash
  sudo chown -R <username>:<username> /var/www/html
  sudo chmod -R 755 /var/www/html
  ```

### Conclusion
You've successfully set up automatic deployment for your website using GitHub Actions. Every time you push to the specified branch (e.g., `main`), the changes will be deployed to your VPS automatically.

---

This documentation should help you or anyone else repeat the process in the future. Feel free to adjust it based on any specific nuances of your setup!
