# Build-Secure-Google-Cloud-Network
#!/bin/bash

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
## SCRIPT END 
