#  3-Tier Web Application Deployment on AWS  
### Manual DataBase  Server + Relational DataBase + Application Load Balancer+ Auto Scaling
### This project deploys a Java-based 3-tier web application on AWS, featuring a secure and scalable setup using NGINX (Jump Server), php (App Server), and MySQL (RDS). It includes Auto Scaling, Application Load Balancer (ALB), and custom VPC networking to ensure high availability, performance, and secure inter-tier communication

--- <img width="1024" height="1024" alt="VPC Architecture with Multi-AZ Setup" src="https://github.com/user-attachments/assets/693f975a-f3e8-4d8a-ae7a-5d40b58ab542" />


##  Project Overview

This project deploys a Java-based 3-tier web application on AWS with:

- **Presentation Layer** → NGINX on Jump Server (Public Subnet)
- **Application Layer** → php ( EC2 Auto Scaling Group)
- **Data Layer** → Manual DB Server EC2 (MariaDB client) connects to RDS (MySQL)

---

##  Architecture Summary

| Layer         | Component                                    | Subnet            |
|---------------|----------------------------------------------|-------------------|
| Presentation  | Jump Server (NGINX)                          | Public Subnet     |
| Application   |  EC2 in Auto Scaling Group                   | Private Subnet 1  |
| Data          | DB EC2 + RDS (MySQL)                         | Private Subnet 2  |

---

##  Security Groups

| Name      | Inbound Rules               | Source               |
|-----------|-----------------------------|----------------------|
| Jump SG   | TCP 22 (SSH), 80 (HTTP)     | Your IP              |
| App SG    | TCP 80                      | ALB SG               |
| ALB SG    | TCP 80                      | 0.0.0.0/0            |
| DB SG     | TCP 3306 (MySQL)            | App SG               |

---

##  Networking (VPC Setup)

1. **VPC**: `10.0.0.0/16`
2. **Subnets**:
   - Public Subnet: `10.0.0.0/20`
   - Private Subnet 1 (App): `10.0.16.0/20`
   - Private Subnet 2 (DB): `10.0.32.0/20`
   <img width="1545" height="714" alt="Screenshot 2025-09-15 051627" src="https://github.com/user-attachments/assets/c57b5913-cab0-4148-99e5-702831750f14" />

3. **Internet Gateway**: Attach to VPC
4. **NAT Gateway**: Create in Public Subnet (use Elastic IP)
5. **Route Tables**:
   - Public Route Table → `0.0.0.0/0 → IGW`
   - Private Route Table → `0.0.0.0/0 → NAT`

---

##  RDS Setup (MySQL)

1. Go to **RDS > Create Database**
<img width="1553" height="248" alt="Screenshot 2025-09-15 051550" src="https://github.com/user-attachments/assets/d7b50ed8-c2f3-46b1-a9e0-37382d01f43e" />

2. Choose **MySQL**
3. DB Name: `Nakhawa`
4. Username: `root`, Password: `rajivnakhawa`
5. Subnet Group: Private Subnet 2
6. Attach **DB SG**

Save the ***RDS Endpoint***.
<img width="1919" height="776" alt="Screenshot 2025-07-04 153339" src="https://github.com/user-attachments/assets/d76b40e2-01cc-445b-92b2-ed496469aa95" />


---

## 🖥️ Manual DB Server (EC2) Setup

1. Launch an EC2 in **Private Subnet 2**
2. Attach **DB SG**
3. Connect via **Jump Server**:
   ```bash
   ssh -i key.pem ec2-user@<DB-Private-IP>
   ```
4. Install MariaDB client:
#### bash
```
 sudo yum install mariadb105 -y
```
5. Connect to RDS (use ***RDS Endpoint***):

bash
```
sudo mysql -h <RDS-ENDPOINT> -u admin -p
```
Inside MySQL:
```
CREATE DATABASE Nakhawa;
USE Nakhawa;

CREATE TABLE users(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    password VARCHAR(10),
);
```
<img width="1011" height="300" alt="Screenshot 2025-09-15 052005" src="https://github.com/user-attachments/assets/026e79e6-fca4-4928-b433-0a30cf3eaeaa" />

## **Launch Template (App EC2)**
1.Go to EC2 > Launch Templates > Create
Use this User Data:
```
```
## ***Target Group + ALB***
**1. Create Target Group :**

- Type: Instance

- Protocol: HTTP

- Port: 80

- Health Check Path: /

**2. Create Application Load Balancer (ALB) :**

- Type: Internet-facing

- Listener: Port 80 → Forward to Target Group

- Attach ALB SG

**Save the ALB DNS Name**

## **Auto Scaling Group (ASG)**
- Go to EC2 > Auto Scaling Group > Create

- Use Launch Template from earlier

- Subnet: Private Subnet 1

- Attach to Target Group

- Scaling Policy:

              *  Desired: 2

              * Min: 1

              * Max: 4

              *  Target CPU > 20%

 ## Jump Server Setup (Public)
 1. Launch EC2 in Public Subnet

 2. Attach Jump SG

 3. SSH into it:

bash
```
ssh -i key.pem ec2-user@<Jump-Public-IP>
```
4. Install NGINX:
#### bash
```
   sudo yum install nginx -y
   sudo nano /etc/nginx/nginx.conf
```
<img width="1889" height="86" alt="sc5" src="https://github.com/user-attachments/assets/61693883-afce-4911-9d97-1d73b25e2f5f" />


5. In location / block:
#### bash
```
location / {
    proxy_pass http://<ALB-DNS-Name>
}
```
<img width="1650" height="866" alt="image" src="https://github.com/user-attachments/assets/fde24bab-0ef4-46b9-bba4-01a322515bb6" />

6. Save and restart NGINX:
#### bash
```
sudo systemctl restart nginx
```
### Tomcat DB Connection Configuration
On your application EC2 instances:

1. Edit context file:
#### bash
```
s
```

2.Add inside <Context>:

3. Save and restart php-fpm:

 ### Test the Application
 1. Open a web browser and navigate to the ALB DNS name.
 2. You should see the Tomcat application running on the ALB DNS name.

 ### OUTPUT
 <img width="683" height="526" alt="Screenshot 2025-09-15 051810" src="https://github.com/user-attachments/assets/56affa8e-d9f3-458b-949c-de3b82e24b20" />
<img width="980" height="102" alt="Screenshot 2025-09-15 051947" src="https://github.com/user-attachments/assets/43d1154e-5ae5-4fe4-9cf0-bc6a1b10499c" />


 <img width="1011" height="300" alt="Screenshot 2025-09-15 052005" src="https://github.com/user-attachments/assets/58f4a954-2b0f-4d7e-bfd9-46295d09c889" />

 
 ## Summary
 **This project demonstrates a 3-Tier Architecture implementation using a public Web Tier, a private Application Tier built with PHP, and a private Database Tier powered by MySQL. The Web Tier provides a simple HTML and JavaScript interface where users can interact with the system. When a user performs an action, the request is sent to the Application Tier, which acts as the middleware and contains the business logic. The PHP backend processes the request, connects to the MySQL Database Tier, retrieves or modifies data, and returns the results as a JSON response. The Database Tier is kept private for security, ensuring it can only be accessed through the Application Tier. This separation of concerns enhances security, scalability, and maintainability, allowing each tier to be managed and scaled independently. The project provides a clear example of how enterprise applications are typically structured to handle user interaction, business logic, and data storage in a modular and secure way.**
