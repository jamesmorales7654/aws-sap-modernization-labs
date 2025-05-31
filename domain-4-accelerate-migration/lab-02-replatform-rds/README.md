# Lab 02 – Replatform: EC2 + Amazon RDS (MySQL)

## 🧠 Objective
Replatform a legacy web application by separating the web and database layers:
- Migrate the app to EC2 with Apache + PHP
- Migrate the database to Amazon RDS (MySQL)
- Connect the app securely to the new managed data layer

---

## ☁️ AWS Services Used
- **Amazon EC2** (Ubuntu) – App tier
- **Amazon RDS** (MySQL) – Data tier
- **Security Groups** – Tier-to-tier traffic rules
- **VPC/Subnets** – Default sandbox VPC and public subnet

---

## ✅ Steps

### 1. Launch EC2 Instance (App Server)

```bash
aws ec2 run-instances \
  --image-id ami-053b0d53c279acc90 \
  --count 1 \
  --instance-type t2.micro \
  --key-name lab-key-v2 \
  --security-group-ids sg-000a30dc74333e639 \
  --subnet-id subnet-xxxxxxxxxxxxxxxxx \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab2-app-server}]'
Security group allows:

SSH (22) from 0.0.0.0/0

HTTP (80) from 0.0.0.0/0

MySQL (3306) from itself (SG ID)

2. Install App Stack (on EC2)
bash
Copy
Edit
sudo apt update
sudo apt install -y apache2 php libapache2-mod-php php-mysql mysql-client
sudo systemctl start apache2
sudo systemctl enable apache2
3. Launch Amazon RDS (MySQL)
bash
Copy
Edit
aws rds create-db-instance \
  --db-instance-identifier lab2-mysql \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password password \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-000a30dc74333e639 \
  --publicly-accessible \
  --backup-retention-period 0
Wait until the instance is available.

4. Connect EC2 to RDS (Verify)
bash
Copy
Edit
mysql -h <rds-endpoint> -u admin -p
# password: password
sql
Copy
Edit
CREATE DATABASE appdb;
USE appdb;
CREATE TABLE visitors (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    visit_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO visitors (name) VALUES ('first-visitor');
5. Deploy PHP App to EC2
bash
Copy
Edit
cd /var/www/html
sudo vim index.php
Paste this:

php
Copy
Edit
<?php
$servername = "lab2-mysql.cfmewq46kcz8.us-east-1.rds.amazonaws.com";
$username = "admin";
$password = "password";
$dbname = "appdb";

$conn = new mysqli($servername, $username, $password, $dbname);
if ($conn->connect_error) {
  die("Connection failed: " . $conn->connect_error);
}
echo "✅ Connected to RDS MySQL successfully.<br>";

$result = $conn->query("SELECT * FROM visitors");
if ($result->num_rows > 0) {
  while($row = $result->fetch_assoc()) {
    echo "Visitor: " . $row["name"] . " at " . $row["visit_time"] . "<br>";
  }
} else {
  echo "No visitors found.";
}
$conn->close();
?>
6. View in Browser
pgsql
Copy
Edit
http://<ec2-public-ip>/index.php
You should see:

vbnet
Copy
Edit
✅ Connected to RDS MySQL successfully.
Visitor: first-visitor at ...
🧹 Cleanup
bash
Copy
Edit
aws ec2 stop-instances --instance-ids <your-ec2-id>
aws rds delete-db-instance --db-instance-identifier lab2-mysql --skip-final-snapshot
�� Architect Notes
Replatforming lifts the app from legacy infra while offloading ops to AWS.

EC2 + RDS is the simplest 2-tier cloud-native pattern.

Security groups were used to enforce tier-to-tier access via port 3306.

No refactoring was needed — only reconfiguration.

✅ Outcome
A real-world, working cloud-native replatformed app: stateless app layer on EC2 and managed DB layer on RDS — with secure integration and minimized operational overhead.
