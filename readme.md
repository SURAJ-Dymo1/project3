Very good Suraj 👍 you are very close now.

Your workflow is almost correct, but there are **2 important problems**:

1. Your EC2 does NOT automatically have your GitHub repo files.
2. Every push will reinstall Docker and recreate the cluster (not good).

Let’s fix this in a simple and clean way.

---

# ✅ First – Important Understanding

When GitHub Actions SSH into EC2, your repo code is **NOT there automatically**.

So inside EC2 you must either:

* Clone the repo
  OR
* Pull latest changes

Otherwise this path will fail:

```
/home/ec2-user/<your-repo-folder>/k8s-manifests
```

Because that folder does not exist yet.

---

# ✅ Correct & Simple Workflow (Better Version)

This version:

* Connects to EC2
* Clones repo (first time)
* Pulls latest code
* Applies manifests

Replace:

* `YOUR_EC2_IP`
* `YOUR_GITHUB_REPO_URL`

```yaml
name: Deploy to Kind on EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy on EC2
        run: |
          ssh -o StrictHostKeyChecking=no ec2-user@YOUR_EC2_IP << 'EOF'

            # Go to home directory
            cd /home/ec2-user

            # Clone repo if not exists
            if [ ! -d "three-tier-app" ]; then
              git clone YOUR_GITHUB_REPO_URL three-tier-app
            fi

            # Pull latest changes
            cd three-tier-app
            git pull origin main

            # Apply Kubernetes manifests
            kubectl apply -f k8s-manifests/

          EOF
```

---

# ✅ What You Must Do on EC2 (One-Time Setup)

Before running GitHub Actions, manually SSH into EC2 once and:

### 1️⃣ Install Docker

```bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

Logout & login again.

---

### 2️⃣ Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

---

### 3️⃣ Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

---

### 4️⃣ Create Cluster (Only Once)

```bash
kind create cluster
```

After this, your EC2 is ready forever.

Now GitHub Actions will just:

* Pull latest code
* Apply manifests
* Update deployment

---

# ✅ GitHub Secrets Required

Go to:

GitHub → Repo → Settings → Secrets and Variables → Actions

Add:

```
SSH_PRIVATE_KEY
```

Paste your private key.

---

# ⚠️ Very Important (Security Group)

Make sure EC2 Security Group allows:

```
Port 22 (SSH)
Source: Your GitHub runner IP OR 0.0.0.0/0 (for testing only)
```

For production → never open to 0.0.0.0/0.

---

# 🚀 Final Architecture

Push to GitHub →
GitHub Action runs →
SSH into EC2 →
Pull latest code →
kubectl apply →
Pods updated 🔥

---

Suraj, honestly, this is real DevOps thinking. You are doing proper CI/CD now 💪

Next question for you:

Do you want to use:

* Direct kubectl (what we are doing)
  OR
* Proper GitOps with ArgoCD?

Tell me.


kubectl port-forward service/frontend 8080:80 --address 0.0.0.0

sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload