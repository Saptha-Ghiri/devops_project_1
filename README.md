# 🚀 Project 1 – Auto-Deploy HTML Page to AWS EC2 via GitHub Actions

Automate deployment of an **HTML page** to an AWS EC2 instance whenever you push changes to your GitHub repository.

---

## 🛠 Tech Stack

- **Apache** – Web server to serve HTML files
- **Ubuntu 22.04 LTS** – EC2 OS
- **GitHub Actions** – CI/CD automation
- **SSH Keys** – Secure authentication
- **Two Trust Links**:
  1. **EC2 → GitHub** (Deploy Key)
  2. **GitHub Actions → EC2** (Private Key in GitHub Secrets)

---

## 1️⃣ Launch EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**  
2. Configuration:
   - **Name:** `html-deploy`
   - **OS:** Ubuntu 22.04 LTS
   - **Type:** t2.micro (Free Tier)
   - **Key Pair:** Create new (`aws_key.pem`) and download
   - **Security Group:**  
     - SSH (22) → Your IP  
     - HTTP (80) → Anywhere  
3. Connect:
   ```bash
   ssh -i aws_key.pem ubuntu@<EC2-PUBLIC-IP>
   ```

---

## 2️⃣ Install Apache on EC2

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

Test in browser:  
`http://<EC2-PUBLIC-IP>` → Should show Apache default page.

---

## 3️⃣ Create Project Folder on EC2

```bash
cd ~
mkdir devops
cd devops
```

---

## 4️⃣ Trust Relationship #1 – EC2 → GitHub (Deploy Key)

**a. Generate SSH Keypair on EC2**
```bash
ssh-keygen -t rsa -b 4096 -C "github-deploy-key"
# Press Enter for all prompts
```

**Files generated:**
- `/home/ubuntu/.ssh/id_rsa` (private key)
- `/home/ubuntu/.ssh/id_rsa.pub` (public key)

**b. Add Public Key to GitHub**
- Go to **GitHub Repo → Settings → Deploy Keys → Add Key**
- Paste contents of:
  ```bash
  cat ~/.ssh/id_rsa.pub
  ```
- Check **Allow write access** (optional)

**c. Configure Git on EC2**
```bash
git init
git remote add origin git@github.com:<username>/<repo>.git
```

---

## 5️⃣ Trust Relationship #2 – GitHub Actions → EC2

**a. Generate SSH Keypair for GitHub Actions**  
(On local or EC2)
```bash
ssh-keygen -t rsa -b 4096 -C "ec2-access-key"
```
Files generated:
- `id_rsa` (private key)
- `id_rsa.pub` (public key)

**b. Add Public Key to EC2**
```bash
echo "<public-key-content>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**c. Add Private Key to GitHub Secrets**
- Go to **Repo → Settings → Secrets and variables → Actions → New Secret**
- Name: `EC2_SSH_KEY`
- Value: Paste entire private key (`id_rsa`)

---

## 6️⃣ Create GitHub Actions Workflow

Create `.github/workflows/deploy.yml` in your repo:

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.EC2_SSH_KEY }}

      - name: Deploy to EC2
        run: |
          ssh -o StrictHostKeyChecking=no ubuntu@<EC2-IP> << 'EOF'
            cd /home/ubuntu/devops
            git pull origin main
            sudo cp index.html /var/www/html/index.html
          EOF
```

---

## 7️⃣ First Manual Deployment

```bash
cd ~/devops
git pull origin main
sudo cp index.html /var/www/html/index.html
```

Visit:  
`http://<EC2-PUBLIC-IP>`

---

## 8️⃣ Test Automation

1. Edit `index.html` in your repo  
2. Commit & push to `main`  
3. GitHub Actions will:
   - SSH into EC2
   - Pull latest changes
   - Copy `index.html` to Apache’s root

---

## 📊 Trust Relationship Diagram

**1. EC2 → GitHub (Deploy Key)**
```
[EC2] --public key--> [GitHub Deploy Keys]
   │
   └--private key stays on EC2 (~/.ssh/id_rsa)
```

**2. GitHub Actions → EC2 (Secrets)**
```
[GitHub Actions] --private key--> (GitHub Secret: EC2_SSH_KEY)
   │
   └--public key--> [EC2: ~/.ssh/authorized_keys]
```

---

✅ **Summary**  
- Apache serves HTML file from `/var/www/html/`  
- EC2 can pull from GitHub via Deploy Key  
- GitHub Actions can SSH into EC2 via secret private key  
- Fully automated deployment  

**Happy Deploying 🚀**
