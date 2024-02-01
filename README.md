Simple Uptime Monitoring with Prometheus, EC2 Auto Discovery, and Discord Alerts
1. Prometheus Setup:
Deploy Prometheus on a centralized server:

Create an EC2 instance of Ubuntu named "prometheus_server".

Configure the instance security group with inbound rules for ports: 80, 22, 9090, 9100, 9093, and associate it with the "prometheus_server" instance.

Install Prometheus on the server:

bash
Copy code
sudo apt-get update
sudo apt-get install prometheus
Edit the Prometheus configuration file (/etc/prometheus/prometheus.yml) to include EC2 Auto Discovery settings:

bash
Copy code
sudo nano /etc/prometheus/prometheus.yml
Add the following EC2 service discovery configuration:

yaml
Copy code
- job_name: 'ec2_auto_discovery'
  ec2_sd_configs:
    - region: '<your-aws-region>'
      access_key: '<your-aws-access-key>'
      secret_key: '<your-aws-secret-key>'
      port: 9100
  relabel_configs:
    - source_labels: [__meta_ec2_tag_Name]
      target_label: instance
2. Node Exporter Deployment:
Create EC2 instance(s) and associate them with the security group of the "prometheus_server".

On the instance(s), execute:

bash
Copy code
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.2.2.linux-amd64.tar.gz
sudo cp node_exporter-1.2.2.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.2.2.linux-amd64.tar.gz node_exporter-1.2.2.linux-amd64
nohup node_exporter &
Verify Prometheus collects Node Exporter metrics by accessing the Prometheus web UI (http://<your-prometheus-server-ip>:9090).

3. Verify Auto Discovery:
Prometheus should automatically discover and monitor new EC2 instances. To verify:

Launch a new EC2 instance in your AWS environment.
After a short time, check the Prometheus web UI to confirm the new instance appears in the list of targets.
Check Prometheus Targets: Access the Prometheus web UI, navigate to "Status" -> "Targets." Look for the job named 'ec2_auto_discovery,' showing the new EC2 instance as a target with status "UP."
4. Uptime Alerting Rules:
Create the file /etc/prometheus/prometheus.rules.yml and add rules specifying conditions for uptime alerting.

Update your prometheus.yml file to include a reference to the rules file.

Reload Prometheus:

bash
Copy code
sudo systemctl reload prometheus
Verify alerts in the Prometheus UI (http://localhost:9090). You should see alerts triggered by the defined rules.

5. Alertmanager Configuration and Discord Integration:
Download Alertmanager v0.25.0:

bash
Copy code
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
tar xvfz alertmanager-0.25.0.linux-amd64.tar.gz
cd alertmanager-0.25.0.linux-amd64
In your Discord server:

Create a channel for Prometheus and Alertmanager notifications.
Set up a webhook in the channel and copy the URL.
Update the Alertmanager configuration file (alertmanager.yml) with the webhook URL.

Start Alertmanager:

bash
Copy code
./alertmanager
7. Testing and Validation:
Simulate downtime scenarios to validate the alerting system.

Ensure Discord receives timely notifications for uptime-related issues.


