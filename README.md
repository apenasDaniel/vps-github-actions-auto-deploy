

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
3. **Create GitHub Actions Workflow**: Set up a workflow to automate the deployment process.
4. **Push Changes and Test Deployment**: Push the workflow file and verify automatic deployment.

## Step-by-Step Guide

### 1. Generate SSH Keys
First, create an SSH key pair on your local machine to facilitate secure communication between GitHub Actions and your VPS.

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/github-actions-vps
```

- When prompted, leave the passphrase empty by pressing Enter twice.

### 2. Copy SSH Key to VPS
Copy the public SSH key (`github-actions-vps.pub`) to the VPS to allow GitHub Actions to connect.

```bash
ssh-copy-id -i ~/.ssh/github-actions-vps.pub -p 2222 danieloliveiradev@YOUR_VPS_IP
```

- Replace `YOUR_VPS_IP` with the IP address of your VPS.
- If using a different SSH port, make sure to specify it using the `-p` flag (as shown with `2222`).

### 3. Add SSH Private Key to GitHub Secrets
Add the SSH private key to GitHub so the workflow can use it.

1. Open your GitHub repository.
2. Go to **Settings > Secrets and variables > Actions**.
3. Click on **New repository secret**.
4. Name the secret `VPS_SSH_KEY`.
5. Paste the content of `~/.ssh/github-actions-vps` (the private key) into the value field.
6. Click **Add secret**.

### 4. Create GitHub Actions Workflow
Create a workflow file to automate deployment on push.

1. In the root of your project, create a directory named `.github/workflows`.
2. Inside the `workflows` folder, create a file named `deploy.yml`.
3. Add the following content to `deploy.yml`:

```yaml
name: Deploy to VPS

on:
  push:
    branches:
      - main  # Adjust this to the branch you want to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Copy files via SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.VPS_SSH_KEY }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          rsync -avz --delete --exclude='.git*' ./ danieloliveiradev@YOUR_VPS_IP:/var/www/html/html-css-js -e "ssh -p 2222 -o StrictHostKeyChecking=no"
```

- Replace `YOUR_VPS_IP` with the IP address of your VPS.
- Adjust the path `/var/www/html/html-css-js` if your project directory is different.

### 5. Push Changes and Test Deployment
Push your changes to GitHub to trigger the workflow.

```bash
git add .
git commit -m "Add GitHub Actions workflow for automated deployment"
git push origin main
```

1. Navigate to the **Actions** tab in your GitHub repository.
2. Verify that the "Deploy to VPS" workflow runs successfully.

### 6. Verify on the VPS
After the workflow completes, SSH into your VPS to verify that the files have been updated.

```bash
ssh -p 2222 danieloliveiradev@YOUR_VPS_IP
cd /var/www/html/html-css-js
ls -la
```

You should see that the files have been updated with the latest changes.

### Troubleshooting
- **Permission Denied:** If you encounter a `Permission denied` error during the workflow execution, ensure that the SSH key has been correctly added to the `authorized_keys` on the VPS.
- **File Permissions:** Ensure that the user on the VPS has the correct permissions for the project directory. You can change the ownership and permissions using:
  
  ```bash
  sudo chown -R danieloliveiradev:danieloliveiradev /var/www/html/html-css-js
  sudo chmod -R 755 /var/www/html/html-css-js
  ```

### Conclusion
You've successfully set up automatic deployment for your website using GitHub Actions. Every time you push to the specified branch (e.g., `main`), the changes will be deployed to your VPS automatically.

---

This documentation should help you or anyone else repeat the process in the future. Feel free to adjust it based on any specific nuances of your setup!
