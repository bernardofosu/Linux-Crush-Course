
# 🔐 Creating and Using an SSH Key Pair for Terraform on AWS

You can create a new SSH key pair named **`terraform-demo`** using the following command on your terminal (Linux/macOS/WSL):

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/terraform-demo
```

---

## 📝 What This Does:

- `-t rsa`: Uses RSA encryption.
- `-b 4096`: Generates a 4096-bit key (stronger security).
- `-f ~/.ssh/terraform-demo`: Saves the private key to `~/.ssh/terraform-demo` and the public key to `~/.ssh/terraform-demo.pub`.

---

## 📌 After Creation:

- **Private Key**: `~/.ssh/terraform-demo` _(keep it secure!)_
- **Public Key**: `~/.ssh/terraform-demo.pub`

---

## ✅ Use in Terraform

To use this key in Terraform, load the public key with:

```hcl
resource "aws_key_pair" "ec2_key" {
  key_name   = "terraform-demo"
  public_key = file("~/.ssh/terraform-demo.pub")
}
```

---

## 📎 Optional: Rename the Private Key to `.pem` Format

You can rename the private key to match AWS’s convention using:

```bash
mv ~/.ssh/terraform-demo ~/.ssh/terraform-demo.pem
```

### ✅ What This Changes

- The key **contents stay the same** — it’s still a valid private key.
- Only the **filename changes** to follow AWS `.pem` style.

### 🔒 Permissions

Ensure proper permissions:

```bash
chmod 400 ~/.ssh/terraform-demo.pem
```

### 🖥️ SSH Access

Use the renamed key when connecting to EC2:

```bash
ssh -i ~/.ssh/terraform-demo.pem ec2-user@<ec2-public-ip>
```

---

## 🔁 Reminder for Terraform

Renaming the private key to `.pem` **does not affect** Terraform usage. Terraform only needs the **public key**:

```hcl
resource "aws_key_pair" "ec2_key" {
  key_name   = "terraform-demo"
  public_key = file("~/.ssh/terraform-demo.pub")
}
```

Let me know if you want help with:

- Storing the key as a variable 🗂️
- Provisioning EC2 with the key 🔧
- Automating access setup in a Terraform module 🧰
