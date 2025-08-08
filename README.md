# 🚀 Automated Deployment to EC2 Using GitHub Actions

This guide shows you **two methods** to set up automated deployment from GitHub to your EC2 instance using SSH.

---

## ✅ Prerequisites

1. An EC2 instance running (Ubuntu preferred).
2. A GitHub repository with your deployment code.
3. Your EC2 security group must allow **SSH (port 22)** access.
4. Apache/Nginx installed on EC2 (optional but recommended).

---

## 🔐 Generate SSH Key Pair

On your local machine (or using Git Bash):

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- This creates two files:
  - Private key: `~/.ssh/id_rsa`
  - Public key: `~/.ssh/id_rsa.pub`

## 🔐 Add Public Key to EC2 (Method 1)

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

## 🔐 Store Private Key in GitHub Secrets

Go to your GitHub repository:

- **Settings → Secrets and variables → Actions → New repository secret**
  - Name: `EC2_SSH_KEY`
  - Value: _Paste the entire private key content from `id_rsa`_

---

# ✅ Method 1: Using `webfactory/ssh-agent` GitHub Action

```yaml
deploy:
  runs-on: ubuntu-latest

  steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.9.0
      with:
        ssh-private-key: ${{ secrets.EC2_SSH_KEY }}

    - name: Deploy to EC2
      run: |
        ssh -o StrictHostKeyChecking=no ubuntu@<your-ec2-ip> << 'EOF'
          cd /home/ubuntu/devops
          git pull origin main
          sudo cp index.html /var/www/html/index.html
        EOF
```

✅ **Note:** You must manually add the **public key** to the `~/.ssh/authorized_keys` file in your EC2.

---

# ✅ Method 2: Manually Writing SSH Setup Commands

```yaml
deploy:
  runs-on: ubuntu-latest

  steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Setup SSH key
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H <your-ec2-ip> >> ~/.ssh/known_hosts

    - name: SSH into EC2 and deploy
      run: |
        ssh ubuntu@<your-ec2-ip> << 'EOF'
          cd /home/ubuntu/devops
          git pull origin main
          sudo cp index.html /var/www/html/index.html
        EOF
```

✅ **Note:** This method includes manual steps to prepare the SSH connection during the workflow.

---

## 🧠 Key Differences

| Feature                        | Method 1 (webfactory/ssh-agent) | Method 2 (manual setup)         |
|-------------------------------|----------------------------------|---------------------------------|
| Uses external GitHub action   | ✅ Yes                          | ❌ No                           |
| Manual SSH key setup required | ✅ Yes                          | ✅ Yes (within workflow)        |
| Flexibility                   | ⚠️ Limited to the action's flow | ✅ Full control                 |
| Host key verification         | ❌ Skipped with `-o Strict...`   | ✅ Handled via `ssh-keyscan`    |

---

## 📌 Conclusion

- In **both methods**, you **must store your private key** (`id_rsa`) as a GitHub secret (`EC2_SSH_KEY`).
- In **Method 1**, you need to **manually add the public key** to EC2.
- In **Method 2**, the public key is used automatically via `ssh-keyscan`, but still assumes the key pair is accepted by the EC2 instance.

---
