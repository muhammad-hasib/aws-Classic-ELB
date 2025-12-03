## Steps to create an Internet-facing Classic ELB for 2 servers

1. **Preconditions**  
   - Have two EC2 instances running (in same Region/VPC), each with a web server (e.g. Apache/Nginx) installed and accessible on the server port (e.g. port 80). :contentReference[oaicite:0]{index=0}  
   - Ensure the instance security groups allow inbound traffic from the load balancer on the listener (and health-check) port. :contentReference[oaicite:1]{index=1}

2. **Open AWS Console → EC2 → Load Balancers → Create Load Balancer → Classic Load Balancer** :contentReference[oaicite:2]{index=2}

3. **Basic configuration**  
   - Set a **unique name** (max 32 alphanumeric/hyphen characters; cannot start or end with hyphen).  
   - Set **Scheme = Internet-facing** (so ELB gets public DNS / public IPs). :contentReference[oaicite:3]{index=3}

4. **Network mapping**  
   - Select the same **VPC** where your EC2 instances exist. :contentReference[oaicite:4]{index=4}  
   - Choose **at least two Availability Zones (AZs)** — pick a public subnet in each AZ (if your instances are in public subnets). This helps with availability and fault-tolerance. :contentReference[oaicite:5]{index=5}

5. **Security group for ELB**  
   - Choose an existing security group, or **create a new one**.  
   - Add an **inbound rule**: allow traffic from `0.0.0.0/0` (Internet) on the **listener port** (e.g. TCP port 80 for HTTP or 443 for HTTPS). This is **mandatory** for Internet-facing ELB.
   - <img width="1577" height="524" alt="image" src="https://github.com/user-attachments/assets/aaea9847-a113-415d-9720-0f24bd7f68f8" />
   - Ensure outbound rules allow return traffic to backend instances / health-check traffic (recommended).

6. **Configure listener(s)**  
   - Define a listener, e.g.:  
     - Client → ELB: HTTP on port 80  
     - ELB → backend instances: HTTP on port 80  
   - You can add more listeners (e.g. HTTPS, other ports) if required. :contentReference[oaicite:6]{index=6}

7. **Health check configuration**  
   - Set health-check protocol (e.g. HTTP), port (e.g. 80), and ping path (e.g. `/index.html` or `/`) so ELB can monitor instance health. Only healthy instances receive traffic. :contentReference[oaicite:7]{index=7}

8. **Register your 2 EC2 instances with the ELB**  
   - Select the two running EC2 instances (from same VPC/AZs) under “Available instances” and register them. :contentReference[oaicite:8]{index=8}  
   - Wait until instances pass health check — then ELB will distribute incoming traffic across them.

9. **Review & Create**  
   - Review all configuration settings: name, scheme, VPC/subnets, security group, listener, health-check, registered instances.  
   - Click **Create Load Balancer** to finish. :contentReference[oaicite:9]{index=9}
  
10. **Post creation checks**
   - Once the Classic ELB is created make sure to check status of the instances.
   - <img width="1829" height="404" alt="image" src="https://github.com/user-attachments/assets/0e5fbe13-bc30-4a5c-a1d4-789b02be5065" />

10. **Test the setup**  
    - Copy the ELB’s public DNS name (something like `my-load-balancer-1234567890.region.elb.amazonaws.com`) from EC2 console.
    - Paste it in a web browser (from Internet). If configured correctly, you should see your server’s default web page. This confirms ELB is working. :contentReference[oaicite:10]{index=10}
    - <img width="1869" height="1061" alt="image" src="https://github.com/user-attachments/assets/b96db19d-8e99-473b-9545-e3f163c4f1e7" />
    - If any of the server goes down, you can check under instance status which shows health of available instances: <img width="1817" height="536" alt="image" src="https://github.com/user-attachments/assets/33082010-2256-4815-b8b2-c75340f0113e" />


