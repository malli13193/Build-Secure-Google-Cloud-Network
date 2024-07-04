# Building a Secure Google Cloud Network:
I'm excited to share my recent experience in building a secure Virtual Private Cloud (VPC) network on Google Cloud Platform! This hands-on lab provided invaluable insights into creating and managing isolated environments within Google Cloud, ensuring robust security for any deployed applications.

Security Configuration for Jeff's Site
Objective: Implement a secure configuration to protect Jeff's website, ensuring SSH access is limited to a bastion host via Google Cloud's Identity-Aware Proxy (IAP), and safeguard the admin console against unauthorized access.

1. Firewall Rules and VM Tags
## Firewall Rule for SSH Access:

Rule Name: iap-allow-ssh
Network: default
Priority: 1000
Direction: Ingress
Action: Allow
Targets: bastion-vm
Source IP Ranges: 35.235.240.0/20 (Google IAP IP range)
Protocols and Ports: tcp:22
This rule ensures that SSH access to virtual machines is only permitted through the IAP, effectively limiting direct SSH access to the bastion host.
# gcloud Command:
``` gcloud compute firewall-rules create iap-allow-ssh \
    --network default \
    --priority 1000 \
    --direction INGRESS \
    --action ALLOW \
    --rules tcp:22 \
    --source-ranges 35.235.240.0/20 \
    --target-tags bastion-vm
```

# startup script:
```#!/bin/bash

## SCRIPT START
apt update
curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
bash install-logging-agent.sh
cat > /tmp/file <<EOF
<source>
  @type tail

  # Parse the timestamp, but still collect the entire line as 'message'
  format /^(?<message>(?<time>[^ ]*\s*[^ ]* [^ ]*) .*)$/

  path /var/log/auth.log
  pos_file /var/lib/google-fluentd/pos/auth.log.pos
  read_from_head true
  tag auth
</source>
EOF
bash -c "cat /tmp/file >> /etc/google-fluentd/config.d/syslog.conf"
service google-fluentd restart
curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -
apt install nodejs nginx -y
cat > /etc/nginx/sites-enabled/default <<EOF
server {
  listen 80;
  location / {
    proxy_pass         http://127.0.0.1:3000;
  }
}
EOF
systemctl restart nginx
cd /tmp
wget https://github.com/bkimminich/juice-shop/releases/download/v9.3.1/juice-shop-9.3.1_node12_linux_x64.tgz
tar -xzf juice-shop-9.3.1_node12_linux_x64.tgz
cd juice-shop_9.3.1
npm start
## SCRIPT END ```
# Additional Security Measures
Enable SSL/TLS: Ensure the website uses SSL/TLS to encrypt data transmitted between the users and the web server. This protects against eavesdropping and man-in-the-middle attacks.

Use Strong Authentication: Implement multi-factor authentication (MFA) for accessing the admin console. This adds an extra layer of security by requiring more than just a password to gain access.

Regular Security Audits: Conduct regular security audits and vulnerability assessments to identify and address potential security weaknesses. This proactive approach helps to mitigate risks before they can be exploited.

Application Security: Implement web application firewalls (WAF) and intrusion detection/prevention systems (IDS/IPS) to protect against common web threats such as SQL injection, cross-site scripting (XSS), and distributed denial-of-service (DDoS) attacks.

# Summary
By implementing the above firewall rules and security measures, Jeff's site will have robust protection against unauthorized access. SSH access will be limited to the bastion host via IAP, ensuring that only authorized personnel can connect. Additionally, restricting access to the admin console to trusted IP ranges and implementing SSL/TLS and strong authentication will further enhance the site's security, reducing the risk of hacking attempts.

Building a secure VPC network on Google Cloud Platform has been an enlightening experience, demonstrating the importance of a well-configured security posture in cloud environments. As cyber threats continue to evolve, staying vigilant and proactive in our security practices is paramount. I look forward to applying these best practices in future projects and sharing more insights with the community.
