### **Phase 1: The Foundation (VPC)**
*Goal: Build the network where everything lives.*

**1. Create the VPC**
*   **Go to:** VPC Dashboard -> **Create VPC**.
*   **Resources to create:** Select **VPC and more**.
*   **Name tag auto-generation:** `xyzvpc`.
*   **IPv4 CIDR block:** `10.0.0.0/16`.
*   **Number of Availability Zones (AZs):** **2** (Crucial for High Availability).
*   **Number of public subnets:** **2**.
*   **Number of private subnets:** **2**.
*   **NAT gateways:** **None**.
*   **VPC endpoints:** **S3 Gateway** (Check this box. It lets servers download the code from S3).
*   **Click:** **Create VPC**.
*   **Wait:** Until it says "Success".
*   **📸 SCREENSHOT 1:** The "Your VPCs" page showing `xyzvpc` with Status "Available".

---

### **Phase 2: The Monolith (Proof of Concept)**
*Goal: Get the app running on one server to prove it works.*

**2. Launch the POC Server**
*   **Go to:** EC2 Dashboard -> **Launch Instances**.
*   **Name:** `Web-POC`.
*   **OS:** Ubuntu Server 24.04 LTS.
*   **Instance Type:** `t3.micro`.
*   **Key pair:** `vockey`.
*   **Network settings:**
    *   **VPC:** Select `xyzvpc`.
    *   **Subnet:** Select `xyzvpc-subnet-public1...` (**Public**).
    *   **Auto-assign Public IP:** **Enable**.
*   **Firewall (Security groups):**
    *   **Create security group**. Name: `POC-SG`.
    *   **Inbound Rules:**
        1.  **Type:** HTTP | **Source:** Anywhere (`0.0.0.0/0`).
        2.  **Type:** SSH | **Source:** My IP.
        3.  **Type:** MYSQL/Aurora | **Source:** `10.0.0.0/16` (This allows internal access for later migration).
*   **Advanced details -> IAM instance profile:** `LabInstanceProfile`.
*   **User data:** Copy code from `SolutionCodePOC` link (Phase 2 script).
*   **Click:** **Launch instance**.

**3. Test the POC**
*   **Go to:** EC2 Instances. Wait for `Web-POC` to be "Running".
*   **Action:** Copy **Public IPv4 address**. Paste in browser.
*   **Action:** Add a student named "Test User 1".
*   **📸 SCREENSHOT 2:** The website showing the list of students.

---

### **Phase 3: The Decoupling (Real Architecture)**
*Goal: Separate the Database from the Web Server.*

**4. Create the Database (RDS)**
*   **Go to:** RDS Dashboard -> **Subnet groups** -> **Create DB subnet group**.
    *   Name: `db-private-subnet-group`.
    *   VPC: `xyzvpc`.
    *   Add subnets: Select AZs (`us-east-1a`, `us-east-1b`) and select the two **Private** subnets.
    *   Click Create.
*   **Go to:** Databases -> **Create database**.
    *   Method: Standard. Engine: **MySQL**. Template: **Free tier**.
    *   DB identifier: `student-db`.
    *   Master username: `nodeapp`. Password: `student12`.
    *   Instance: `db.t3.micro`.
    *   Storage: 20 GiB.
    *   **Connectivity:**
        *   VPC: `xyzvpc`.
        *   Public access: **No**.
        *   VPC security group: Create new `DB-SG`.
    *   **Additional config:** Initial database name: `STUDENTS` (**Case sensitive**). Backup: Enable.
    *   **Click:** **Create database**.

**5. Create the Cloud9 Admin Machine**
*   **Go to:** Cloud9 -> **Create environment**.
*   **Name:** `Admin-Workstation`.
*   **Instance type:** `t3.micro`. Platform: Ubuntu 22.04.
*   **Network settings:**
    *   **Connection type:** **Secure Shell (SSH)** (CRITICAL).
    *   VPC: `xyzvpc`. Subnet: **Public**.
*   **Click:** **Create**.

**6. Migrate Data & Set Secrets**
*   **Fix DB Security Group:** Go to EC2 -> Security Groups -> `DB-SG`. Edit Inbound Rules. Add/Edit rule: **Type:** MYSQL/Aurora, **Source:** `10.0.0.0/16`. Save.
*   **Open Cloud9 IDE.**
*   **Action 1: Store Secret.**
    *   Run this (replace `[YOUR_RDS_ENDPOINT]`):
        ```bash
        aws secretsmanager create-secret --name Mydbsecret --secret-string '{"user":"nodeapp","password":"student12","host":"[YOUR_RDS_ENDPOINT]","db":"STUDENTS"}'
        ```
    *   **📸 SCREENSHOT 3:** The Secrets Manager console showing `Mydbsecret`.
*   **Action 2: Migrate Data.**
    *   Run in Cloud9 (replace IPs/Endpoint):
        ```bash
        mysqldump -h [WEB_POC_PRIVATE_IP] -u nodeapp -p --databases STUDENTS > data.sql
        # Password: student12
        mysql -h [YOUR_RDS_ENDPOINT] -u nodeapp -p STUDENTS < data.sql
        # Password: student12
        ```

---

### **Phase 4: The Scale Up (High Availability)**
*Goal: Make it scale and handle traffic.*

**7. Create the Final Web Server Template**
*   **Go to:** EC2 -> Launch Templates -> **Create launch template**.
*   **Name:** `Web-Template`.
*   **OS:** Ubuntu 24.04. Instance: `t3.micro`. Key: `vockey`.
*   **Network settings:**
    *   **Subnet:** Don't include.
    *   **Firewall:** Create security group `Web-SG`. Rules: HTTP (`0.0.0.0/0`), SSH (`0.0.0.0/0`).
    *   **Advanced network configuration:** **Auto-assign public IP: Enable** (CRITICAL).
*   **Advanced details:** **IAM instance profile:** `LabInstanceProfile`.
*   **User data:** (Use the Secure Script):
    ```bash
    #!/bin/bash -xe
    dnf update -y
    dnf install -y gcc-c++ make
    curl -sL https://rpm.nodesource.com/setup_16.x | bash -
    dnf install -y nodejs
    mkdir /var/www
    cd /var/www
    wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCARCH-2/lab-task-sol-code/RDSPOC.zip
    unzip RDSPOC.zip
    cd RDSPOC
    npm install
    export SECRET_NAME=Mydbsecret
    export REGION=us-east-1
    export APP_PORT=80
    node index.js
    ```
*   **Click:** **Create launch template**.

**8. Create the Load Balancer (ALB)**
*   **Go to:** EC2 -> Load Balancers -> **Create load balancer**.
*   **Type:** Application Load Balancer. Name: `Web-ALB`. Scheme: Internet-facing.
*   **Network:** `xyzvpc`. Select **both AZs**. Select **PUBLIC** subnets.
*   **Security groups:** Create `ALB-SG`. Allow HTTP (`0.0.0.0/0`).
*   **Listeners:** HTTP 80 -> Create target group -> `Web-TG` -> Instances.
*   **Click:** **Create load balancer**.
*   **📸 SCREENSHOT 4:** The Load Balancer description page showing "Active".

**9. Create Auto Scaling Group (ASG)**
*   **Go to:** EC2 -> Auto Scaling Groups -> **Create**.
*   **Name:** `Web-ASG`. Template: `Web-Template`.
*   **Network:** `xyzvpc`. Select two **PUBLIC** subnets.
*   **Load balancing:** Attach to existing -> `Web-TG`.
*   **Group size:** Desired: 2, Min: 2, Max: 4.
*   **Scaling policies:** Target tracking -> CPU -> 50.
*   **Click:** **Create**.
*   **Wait:** Wait for 2 instances to launch and be Healthy.
*   **📸 SCREENSHOT 5:** The ASG page showing 2 instances launched.

**10. Final Security Lockdown**
*   **Go to:** Security Groups.
*   **Select `Web-SG`:** Edit Inbound Rules. Change HTTP Source to **Custom -> `ALB-SG`**.
*   **Select `DB-SG`:** Ensure MySQL Source allows `10.0.0.0/16`.
*   **📸 SCREENSHOT 6:** The `Web-SG` inbound rules page.

---

### **Phase 5: The Test**
*Goal: Prove it works.*

**11. Verify Functionality**
*   Paste ALB DNS in browser. Add a student.
*   **📸 SCREENSHOT 7:** The website loaded via the ALB URL.

**12. Stress Test**
*   **Go to:** Cloud9.
*   **Run:**
    ```bash
    sudo npm install -g loadtest
    loadtest -n 10000 -c 100 --rps 200 http://[YOUR_ALB_DNS_NAME]
    ```
*   **Go to:** EC2 -> Auto Scaling Groups -> **Activity** tab.
*   **📸 SCREENSHOT 8:** The Cloud9 terminal showing the load test results.
*   **📸 SCREENSHOT 9:** The ASG Activity history showing "Launching a new EC2 instance".

---

### **Phase 6: Cost Estimation**
*Goal: Show the money.*

**13. Create Cost Estimate**
*   Go to [AWS Pricing Calculator](https://calculator.aws/).
*   **Configure the following services:**
    1.  **Amazon EC2:**
        *   2 Instances, `t3.micro`, Linux, On-Demand.
    2.  **Amazon RDS for MySQL:**
        *   1 Instance, `db.t3.micro`, Single-AZ.
        *   Storage: 20 GB.
    3.  **Elastic Load Balancing:**
        *   Application Load Balancer (1 unit).
        *   Processed bytes: 1 GB/month (Estimate).
    4.  **AWS Secrets Manager:**
        *   1 Secret.
    5.  **Amazon S3:** (This is the one you asked about).
        *   Standard Storage.
        *   Storage: 1 GB (For logs/code artifacts).
*   **Click:** **Add to estimate** for each.
*   **Click:** **View summary**.
*   **📸 SCREENSHOT 10:** The Cost Estimate Summary page showing the total monthly cost.
