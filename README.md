# Simple Uptime Monitoring with Prometheus, EC2 Auto Discovery, and Discord Alerts

## 1. Prometheus Setup:
Deploy Prometheus on a centralized server:
* Create an ec2 instnace of ubuntu - "prometheus_server"
* Create with the instance security group with inbound rules of ports: 80, 22, 9090, 9100, 9093  and associate it with the prometheus_server.
* In the prometheus_server type the commands:
```
sudo apt-get update
sudo apt-get install prometheus
```
*Edit the Prometheus configuration file (/etc/prometheus/prometheus.yml) to include the EC2 Auto Discovery settings. 
Open the configuration file in a text editor:
```
sudo nano /etc/prometheus/prometheus.yml
```
*Add the following EC2 service discovery configuration to the file:
```
- job_name: 'ec2_auto_discovery'
    ec2_sd_configs:
      - region: '<your-aws-region>'
        access_key: '<your-aws-access-key>'
        secret_key: '<your-aws-secret-key>'
        port: 9100
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        target_label: instance
```

## 2. Node Exporter Deployment:
*Create ec2 instance/s and associated them with the security group of the prometheus_server.
* In the instance/s type the commands:
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.2.2.linux-amd64.tar.gz
sudo cp node_exporter-1.2.2.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.2.2.linux-amd64.tar.gz node_exporter-1.2.2.linux-amd64
nohup node_exporter &
```
* Verify Prometheus Collects Node Exporter Metrics:
Access the Prometheus web UI (http://<your-prometheus-server-ip>:9090) and use the Prometheus expression browser
to query Node Exporter metrics, for example: "up".  This should return a list of instances reporting as "up," indicating that Prometheus is successfully collecting metrics from Node Exporter on your EC2 instances.


## 3. Verify Auto Discovery:
Now, Prometheus should automatically discover and monitor any new EC2 instances based on the configured EC2 Auto Discovery. To verify this:
Launch a new EC2 instance in your AWS environment.
After a short time, check the Prometheus web UI to confirm that the new instance appears in the list of targets.
* Check Prometheus Targets:
Access the Prometheus web UI, and navigate to the "Status" -> "Targets" page. Look for the job named 'ec2_auto_discovery'. 
It should show the new EC2 instance as a target, and the status should be "UP."

## 4. Uptime Alerting Rules:
Create file /etc/prometheus/prometheus.rules.yml
Open the prometheus.rules.yml file in a text editor.
Add rules specifying conditions for uptime alerting.
If your prometheus.yml file doesn't already include a reference to the rules file, you may need to update it.
After this type this command: sudo systemctl reload prometheus
Verify Alerts in Prometheus UI:
Open Prometheus web UI (usually at http://localhost:9090) and navigate to the "Alerts" tab. You should see alerts triggered by the defined rules.

## 5. Alertmanager Configuration and discord Integration:
Download Alertmanager v0.25.0 (in a separate directory):
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
tar xvfz alertmanager-0.25.0.linux-amd64.tar.gz
cd alertmanager-0.25.0.linux-amd64
```
Log in to your discord account and create a new server.
Inside your Discord server, you can create channels for different purposes. 
One channel could be dedicated to receiving notifications from Prometheus and Alertmanager.
To allow Alertmanager to send messages to your Discord channel, you'll need to set up a webhook.
Go to the channel where you want the notifications.
Click on the gear icon next to the channel name.
Select "Integrations."
Click on "Create Webhook."
Copy the webhook URL.
In your Alertmanager configuration file (alertmanager.yml), update the Discord configuration with the webhook URL.
Start Alertmanager with the above config:
```
./alertmanager
```
## 6. Testing and Validation:
Simulate downtime scenarios to validate the alerting system.
Ensure that Discord receives timely notifications for uptime-related issues.
