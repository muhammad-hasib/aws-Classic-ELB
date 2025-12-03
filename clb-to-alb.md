Migrating a Classic ELB to ALB:
Steps:
# Migrating from Classic Load Balancer → Application Load Balancer (ALB)

## 🎯 Why migrate to ALB

- ALB works at the application layer (HTTP/HTTPS) and supports advanced routing features: path- or host-based routing, header/method based rules, content-based routing etc. :contentReference[oaicite:2]{index=2}  
- ALB offers better scaling, supports registering targets by instance, IP, or even Lambda, and integrates well with modern architectures (microservices, containers, etc.). :contentReference[oaicite:3]{index=3}  
- For newer workloads and future-proofing, ALB is recommended over the older Classic Load Balancer. :contentReference[oaicite:4]{index=4}

---

## ✅ Preconditions / What to check before migrating

- Your CLB must be in a VPC, and already spanning more than one Availability Zone / subnet (since ALB requires at least two subnets). :contentReference[oaicite:5]{index=5}  
- Note the listeners, health-check settings, backend instances, subnets, security groups used by the CLB — these will guide your ALB configuration.  
- Be aware: the migration “wizard” **creates a new ALB** — it does **not** convert/replace the existing CLB. You will need to manually redirect traffic to the new ALB. :contentReference[oaicite:6]{index=6}

---

## 🛠 Migration Steps (using your chosen settings)

### 1. Launch migration wizard (easy path)

1. Open AWS Console → go to **EC2 → Load Balancers**.
2. Select your existing Classic Load Balancer.  
3. Click **“Launch migration wizard”**. 
4. Choose **“Migrate to Application Load Balancer”**.
5. Under **“Name new load balancer”**, enter your chosen name:  application-ELB
(Name must be unique in your account / region; can't use the same name as existing CLB.) :contentReference[oaicite:11]{index=11}  
6. Under **“Name new target group”**, enter:  application-elb-tg
The wizard will create the target group and pre-populate it with your existing healthy EC2 instances. :contentReference[oaicite:12]{index=12}  
7. (Optional) Review the list of targets — any non-running instances won’t be registered. :contentReference[oaicite:13]{index=13}  
8. (Optional) Review tags — note that tags with keys prefixed `aws:` are **not** migrated automatically. :contentReference[oaicite:14]{index=14}  
9. In the “Summary for Application Load Balancer”, verify the settings:  
- Scheme: **Internet-facing**  
- IP address type: IPv4  
- VPC: same as your existing CLB’s VPC  
- Subnets / AZs: include at least two (e.g. subnets in different AZs) — since ALB needs ≥ 2 subnets. :contentReference[oaicite:15]{index=15}  
- Security group(s): same or equivalent to those on the CLB — ensure inbound/outbound rules are appropriate for ALB.  
- Listener: HTTP on port 80, forwarding to your new target group `application-elb-tg`  
- Health check: replicate your previous CLB health-check (protocol, port, path) if possible — ALB’s target-group health check should match. :contentReference[oaicite:16]{index=16}  
10. Click **“Create Application Load Balancer”** — this creates the ALB + target group with your configuration. :contentReference[oaicite:17]{index=17}  

### 2. Manual Migration (if you prefer to avoid the wizard / want full control)  
If you don’t want to use the wizard (or you are using IaC / CLI / Terraform), follow these steps:

- Manually create a new ALB with: the same scheme (internet-facing), subnets (≥ 2), VPC, security groups. :contentReference[oaicite:18]{index=18}  
- Create a new target group: use protocol = HTTP, port = 80 (as per your setup), and set health-check path (e.g. `/index.html`) matching your previous CLB health-check.  
- Register your running EC2 instances (the ones behind CLB) with this target group. If using Auto Scaling, attach the target group to your ASG so instances register automatically. :contentReference[oaicite:19]{index=19}  
- Create listener(s) on ALB: for example, HTTP listener on port 80, forwarding to the target group. If previously you had HTTPS on CLB, recreate HTTPS listener on ALB and reuse the same certificate (via ACM or IAM) with a default or compatible TLS policy. :contentReference[oaicite:20]{index=20}  
- (Optional) Apply relevant tags (copy over from CLB, except keys that start with `aws:` — those won’t migrate automatically). :contentReference[oaicite:21]{index=21}  

---

## 🔁 After ALB Creation — Switch Traffic

- Grab the ALB’s DNS name from AWS Console → test it in a browser or via curl to ensure your web application responds as expected.  
- Once confident, update your DNS (or routing / domain records) to point to the new ALB instead of the old CLB. If using a DNS provider that supports weighted routing (e.g. Amazon Route 53), you can gradually shift traffic (e.g. 10% → 90%) for safer transition. :contentReference[oaicite:23]{index=23}  
- Monitor responses and health-check status to ensure the new ALB + target group is working correctly.

---

## 🧹 Cleanup & Post-Migration Tasks

- Once traffic is fully routed to ALB and everything works correctly, **delete the old Classic Load Balancer** to avoid unnecessary charges.  
- Update any automation / infrastructure-as-code / scripts / CloudFormation / CI/CD tools: replace references to Classic ELB (API `elb`) with the new ALB (`elbv2`) APIs or resource types. 
- Update monitoring/metrics dashboards if you referenced CLB metrics (`AWS/ELB` namespace) — for ALB use the new namespace (`AWS/ApplicationELB`).

---

## 📋 Summary of the Migration Settings

| Setting | Value / Configuration |
|--------|------------------------|
| New Load Balancer Name | `application-ELB` |
| Scheme | Internet-facing |
| IP Address Type | IPv4 |
| Target Group Name | `application-elb-tg` |
| Listener | HTTP : 80 → target group on port 80 |
| Health-Check Path / Settings | HTTP, port 80, match existing CLB health-check (e.g. `/index.html`) |
| VPC / Subnets / AZs | Same VPC as CLB; at least two subnets across different AZs |
| Security Group(s) | Same as existing CLB (or equivalent) with appropriate inbound/outbound rules |
| Targets (EC2 Instances) | Existing running EC2 instances previously behind CLB (registered to the new target group) |

---

## ℹ️ Notes & Caveats

- The migration wizard creates a **new** ALB — it does *not* convert the CLB in place. The old CLB will continue to exist until you manually delete it.  <img width="1592" height="261" alt="image" src="https://github.com/user-attachments/assets/5efa9cd9-a9d2-4959-90ca-daaa09849537" />
- If your CLB used only one subnet, you must specify a second subnet when migrating — ALB requires at least two subnets.
- Some CLB settings may not map exactly to ALB (e.g. TCP/SSL health-checks under CLB may get translated to HTTP health-checks during migration).
- Tags whose keys start with `aws:` are not migrated automatically — you may need to re-apply them manually if needed. 

---

*End of migration plan.*  

