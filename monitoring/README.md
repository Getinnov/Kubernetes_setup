```
curl -SL https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-386.tar.gz > node_exporter.tar.gz && sudo tar -xvf node_exporter.tar.gz -C /usr/local/bin/ --strip-components=1

cat > /etc/systemd/system/nodeexporter.service <<EOF
[Unit]
Description=NodeExporter

[Service]
TimeoutStartSec=0
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

rm node_exporter.tar.gz
sudo systemctl daemon-reload && sudo systemctl enable nodeexporter && sudo systemctl start nodeexporter
```

Note: node_exporter save on this git
