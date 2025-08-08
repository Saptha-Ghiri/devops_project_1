# 🚀 Project 1 – Auto-Deploy HTML Page to AWS EC2 via GitHub Actions

This guide walks through **deploying an HTML page to an EC2 instance automatically** when you push changes to your GitHub repo.

We use:
- **Apache** to serve the HTML page
- **SSH Keys** for secure communication
- **Two trust relationships**:
  1. **EC2 → GitHub** (Deploy Key)
  2. **GitHub Actions → EC2** (Private Key in GitHub Secrets)

---

## 📌 1. Launch EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**
2. Select:
   - **Ubuntu 22.04 LTS**
   - **t2.micro** (Free Tier)
   - Create a new key pair (`aws_key.pem`) → Download it
   - Open Security Group ports:
     - SSH (22) → Your IP
     - HTTP (80) → Anywhere
3. Launch and connect:
   ```bash
   ssh -i aws_key.pem ubuntu@<EC2-PUBLIC-IP>
📌 2. Install Apache on EC2
bash
Copy
Edit
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
Check in browser:
http://<EC2-PUBLIC-IP> → You should see the Apache default page.

📌 3. Create Project Folder on EC2
bash
Copy
Edit
cd ~
mkdir devops
cd devops
📌 4. Trust Relationship #1 – EC2 → GitHub (Deploy Key)
a. Generate SSH Keypair on EC2
bash
Copy
Edit
ssh-keygen -t rsa -b 4096 -C "github-deploy-key"
# Press Enter for all prompts
Files generated:

swift
Copy
Edit
/home/ubuntu/.ssh/id_rsa      (private key)
/home/ubuntu/.ssh/id_rsa.pub  (public key)
b. Add Public Key to GitHub
Go to GitHub repo → Settings → Deploy Keys

Add a new key, paste content of:

bash
Copy
Edit
cat ~/.ssh/id_rsa.pub
Check Allow write access (optional)

c. Configure Git on EC2
bash
Copy
Edit
git init
git remote add origin git@github.com:<username>/<repo>.git
📌 5. Trust Relationship #2 – GitHub Actions → EC2
a. Generate SSH Keypair for GitHub Actions
On your local machine or EC2:

bash
Copy
Edit
ssh-keygen -t rsa -b 4096 -C "ec2-access-key"
Files generated:

vbnet
Copy
Edit
id_rsa     (private key)
id_rsa.pub (public key)
b. Add Public Key to EC2
bash
Copy
Edit
echo "<public-key-content>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
c. Add Private Key to GitHub Secrets
Go to Repo → Settings → Secrets and variables → Actions

Create new secret:

Name: EC2_SSH_KEY

Value: Paste entire private key (id_rsa)

📌 6. Create GitHub Actions Workflow
In your repo, create:
.github/workflows/deploy.yml

yaml
Copy
Edit
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
📌 7. First Manual Deployment
Before automation, run this once on EC2:

bash
Copy
Edit
cd ~/devops
git pull origin main
sudo cp index.html /var/www/html/index.html
Visit:

cpp
Copy
Edit
http://<EC2-PUBLIC-IP>
📌 8. Test Automation
Edit index.html in your repo

Commit & push to main

GitHub Actions will:

SSH into EC2

Pull latest changes

Copy index.html to Apache’s root

📊 Trust Relationship Diagram
1. EC2 → GitHub (Deploy Key)
scss
Copy
Edit
[EC2] --public key--> [GitHub Deploy Keys]
   │
   └--private key stays on EC2 (~/.ssh/id_rsa)
2. GitHub Actions → EC2 (Secrets)
scss
Copy
Edit
[GitHub Actions] --private key--> (GitHub Secret: EC2_SSH_KEY)
   │
   └--public key--> [EC2: ~/.ssh/authorized_keys]
✅ Summary
Apache serves HTML file from /var/www/html/

EC2 can pull from GitHub via Deploy Key

GitHub Actions can SSH into EC2 via secret private key

Fully automated deployment

Happy Deploying 🚀

yaml
Copy
Edit

---

Do you want me to **also create a diagram image** for the trust relationships so it can be added to this README? That would make it visually clear. ​:contentReference[oaicite:0]{index=0}​