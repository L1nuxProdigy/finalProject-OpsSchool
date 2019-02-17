#!/bin/bash

# Downloading and installing Prerequisites
sudo apt-get update -y
sudo apt install unzip -y
sudo wget https://releases.hashicorp.com/consul/1.4.0/consul_1.4.0_linux_amd64.zip

# Unzipping and moving the binary to path
sudo unzip consul_1.4.0_linux_amd64.zip
sudo mv ./consul /usr/local/bin/consul
sudo chown ubuntu /usr/local/bin/consul


## consul client command
sudo mkdir /var/lib/consul /etc/consul.d
LOCAL_IPV4=$(curl "http://169.254.169.254/latest/meta-data/local-ipv4")

cat << EOCCF >/etc/consul.d/agent.hcl
node_name = "consul-kubernetes-node"
client_addr =  "0.0.0.0"
recursors =  ["127.0.0.1"]
bootstrap =  false
datacenter = "dc1"
data_dir = "/var/lib/consul"
enable_syslog = true
log_level = "DEBUG"
retry_join = ["provider=aws tag_key=Name tag_value=Ansible_Server-TerraBuild"]
advertise_addr = "${LOCAL_IPV4}"
EOCCF

cat << EOCSU >/etc/systemd/system/consul.service
[Unit]
Description=consul agent
Requires=network-online.target
After=network-online.target
[Service]
LimitNOFILE=65536
Restart=on-failure
ExecStart=/usr/local/bin/consul agent -config-dir /etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
Type=notify
[Install]
WantedBy=multi-user.target
EOCSU

systemctl daemon-reload
systemctl start consul &


#sudo echo '{"service": {"name": "dummy_exporter", "tags": ["dummy"], "port": 65433}}' > /etc/consul.d/dummy_exporter.json