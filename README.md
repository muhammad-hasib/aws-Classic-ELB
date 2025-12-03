# ELB + Static-Site Test Setup

## 🌐 Project Overview
This repository demonstrates a test setup where two static websites are hosted on separate servers (or instances), and traffic is distributed between them using a load balancer.  
Use-case: Testing load balancing / high-availability / failover for simple static sites.

## 🧰 What’s Inside
- **site1/** — HTML/CSS for static site 1  
- **site2/** — (Optional) HTML/CSS for static site 2  
- **nginx-configs/** — Nginx server-block configs used on each server to serve static files  
- **aws-elb-setup/** — Notes / steps / scripts (if any) used to setup the load balancer and register backend servers (for ELB)  
- **README.md** — This documentation  

## 🚀 Setup & Usage (Manual Web-Server + Nginx + ELB)
1. Launch two Ubuntu (or other) instances.  
2. Copy `site1/` (and `site2/`) contents to servers’ web root (e.g. `/var/www/site1`).  
3. Copy appropriate Nginx config from `nginx-configs/`, enable it, disable default site, reload Nginx.  
4. Verify by accessing each server’s public IP — you should see your static site.  
5. On AWS (or your cloud), create a load balancer, register both servers (instances) as targets.  
6. Configure listener(s), health-checks, security-groups, and route traffic to backend servers (as per ELB documentation).  
7. Access load balancer DNS (or configured domain) — traffic will be distributed between your two backend servers serving the static sites.  

## 📚 ELB Setup (AWS) — Key Steps
- Create a load balancer (e.g. Classsic Load Balancer).  
- Define subnets, security-groups, and set up listener (HTTP port 80 / HTTPS as needed).  
- Create a target group, register both server instances as targets.  
- Configure health check settings.  
- After target status becomes healthy, requests to the load balancer will route to either of the servers.  

## ✅ Why This Setup
- Demonstrates basic load-balancing with static content  
- Easy to replicate — only static files + Nginx + ELB  
- Helps in testing distribution, failover, and basic HA behaviour  

## 📝 Notes & Tips
- Don’t commit private keys, credentials, or sensitive info to GitHub.  
- Use relative paths and environment-agnostic configs where possible.  
- Document any custom port, SSL, firewall or DNS configuration you apply.  


