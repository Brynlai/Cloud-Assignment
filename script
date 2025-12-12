Here is a blunt, efficient script designed to hit every "Excellent" criteria in your rubric within 10 minutes.

**Preparation:**
*   Have all the tabs listed below open in your browser *before* you start recording.
*   Do not wait for pages to load. Click the tab, speak the line, move on.

---

### **Introduction (30 Seconds)**
**[Open Slide: Title Slide with Group Members]**
"This is [Your Name] from Group [X]. We built a highly available, scalable, 3-tier web application on AWS for student records. Here is the proof."

**[Open Slide: Architecture Diagram]**
"This is our architecture. We use a custom VPC with 2 Availability Zones.
*   **Public Subnets** host the Application Load Balancer and Auto Scaling Web Servers.
*   **Private Subnets** host the RDS Database.
*   We use **Secrets Manager** for security and **S3** for code artifacts."

---

### **1. Functional (1 Minute)**
**[Open Tab: The Website (ALB DNS URL)]**
"First, Functionality. The site is live via the Load Balancer DNS."

**[Action: Click 'Add Student'. Type 'Demo User'. Click Submit.]**
"I can add a student. It appears immediately."

**[Action: Click 'Edit' on 'Demo User'. Change name to 'Demo User Edited'. Click Submit.]**
"I can modify records."

**[Action: Click 'Delete' on 'Demo User'.]**
"I can delete records. All CRUD operations work with zero delay. The data migrated successfully from our initial POC to this final RDS instance."

---

### **2. Highly Available & Network (1.5 Minutes)**
**[Open Tab: VPC Dashboard -> Resource Map]**
"For High Availability, we deployed across **two** Availability Zones: `us-east-1a` and `us-east-1b`. You can see Public and Private subnets in both zones."

**[Open Tab: EC2 Instances (Filtered by Running)]**
"Our web servers are spread across these two zones. If one zone fails, the other handles traffic."

**[Open Tab: RDS Databases -> Connectivity Tab]**
"Our Database is in the **Private Subnets**, completely isolated from the internet. We also enabled **Automated Backups** for recovery."

**[Open Tab: Secrets Manager]**
"We do not hardcode passwords. The database credentials are stored here in **AWS Secrets Manager** and retrieved dynamically by the app."

---

### **3. Load Balancing (1 Minute)**
**[Open Tab: EC2 -> Load Balancers -> Listeners Tab]**
"Traffic is managed by an **Application Load Balancer**. It listens on Port 80."

**[Open Tab: EC2 -> Target Groups -> Targets Tab]**
"The ALB routes traffic evenly to our Target Group. You can see our instances here, marked **Healthy** across multiple zones. This ensures no single instance is overloaded."

---

### **4. Scalability & High Performance (2 Minutes)**
**[Open Tab: EC2 -> Auto Scaling Groups -> Automatic Scaling Tab]**
"For Scalability, we use an **Auto Scaling Group**.
*   **Min Capacity:** 2 (to handle baseline traffic).
*   **Max Capacity:** 4 (to handle spikes).
*   **Policy:** Target Tracking on **CPU Utilization** set to 50%."

**[Open Tab: Cloud9 Terminal (Show the load test result)]**
"We stress-tested the system with **10,000 requests** at **200 requests per second**.
*   **Error Rate:** Less than 1%.
*   **Response Time:** Fast.
*   The system remained functional under maximum load."

**[Open Tab: Auto Scaling Group -> Activity History Tab]**
"Here is the proof of scaling. You can see the ASG detected the high CPU load and successfully **launched a new EC2 instance** to handle the traffic. It scaled out in a timely manner."

---

### **5. Security (2 Minutes)**
**[Open Tab: EC2 -> Security Groups -> Web-SG]**
"For Security, we followed strict least-privilege access.
*   **Web Server SG:** Only accepts HTTP traffic from the **Load Balancer's Security Group**. It accepts SSH only from our admin IP. Port 22 is NOT open to the world."

**[Open Tab: EC2 -> Security Groups -> DB-SG]**
"**Database SG:** Only accepts traffic on Port 3306 from the **Web Server Security Group** and our internal Admin tools. It is blocked from the public internet."

**[Open Tab: EC2 -> Security Groups -> ALB-SG]**
"**Load Balancer SG:** Only this group allows HTTP traffic from the open internet (`0.0.0.0/0`)."

---

### **6. Cost Optimization (1 Minute)**
**[Open Tab: Cost Estimate PDF/Screenshot]**
"Finally, Cost Optimization.
*   **Compute:** We used `t3.micro` instances, which are sized perfectly for this workload.
*   **Database:** We used `db.t3.micro` free tier.
*   **Scaling:** We configured the ASG to scale in when traffic drops, so we don't pay for idle servers.
*   **Storage:** We used GP2/GP3 volumes appropriate for the data size."

**[Switch to Conclusion Slide]**
"To summarize: We built a secure, multi-AZ, auto-scaling architecture that survived a heavy load test while keeping costs low. Thank you."
