Start working environment
========================== 

Go to Monitoring and Logging workshop folder: $HOME/monitoring-logging-workshop​ 

Start provisioned virtual machines: vagrant up --no-provision​ 

Check virtual machines status: vagrant status​ 

 

Install New Relic Infra Agent 
==============================
```
echo "license_key: YOUR_LICENSE_KEY" | sudo tee -a /etc/newrelic-infra.yml 

cat /etc/os-release 

sudo curl -o /etc/yum.repos.d/newrelic-infra.repo https://download.newrelic.com/infrastructure_agent/linux/yum/el/7/x86_64/newrelic-infra.repo

sudo yum -q makecache -y --disablerepo='*' --enablerepo='newrelic-infra'

sudo yum install newrelic-infra -y


sudo systemctl start newrelic-infra.service​

sudo systemctl status newrelic-infra.service​
```
 

Agent log forwarding
==================== 
`
In your system go to newrelic-infra logging folder: ```cd /etc/newrelic-infra/logging.d​```

Create a copy of file.yml.example: ```cp file.yml.example file.yml​ ```

Edit the copied .yml file to have the following configurations: ​ 
```
vim file.yml​ 
```
 
```
logs:​ 

    # Fetch syslog​ 

  - name: syslog​ 

    file: /var/log/syslog​ 
```
 

 

Get the agent logs into New Relic 
==================================
Enable logging by editing /etc/newrelic-infra.yml​ 
```
license_key: <license key>​ 

log_file: /var/log/newrelic-logs.log​ 
```
Restart the agent using systemctl​ 
```
sudo systemctl restart newrelic-infra​.service
```
 

Create alert for DiskFreePercent
================================ 

```
NRQL> SELECT max(`host.disk.usedPercent`) FROM Metric 
```
​ 

 

NRQL Examples
============= 

View Infrastructure events in a table view since 11AM till now​​ 
```
NRQL> SELECT * FROM InfrastructureEvent SINCE '11:00' UNTIL now 
```
Get unique count of user sessions for the entire week​​ 
```
NRQL> SELECT uniqueCount(session) FROM PageView SINCE 1 week ago 
```
How many of our infrastructure instances run Linux as main OS? 
```
NRQL> SELECT count(`host.operatingSystem`) FROM Metric WHERE host.operatingSystem ='linux' SINCE 1 day ago
```
NRQL Practice Answers 
=====================

What is the unique number of processes (`process.name`) per a user (`process.userName`)? Exclude the root user​ 
```
NRQL>  SELECT uniqueCount(`process.name`) FROM Metric WHERE `process.userName` != 'root' FACET `process.userName` SINCE 8 hours ago​ 
```
Get the 50th, 75th, 99th percentiles of the metric `host.net.transmitPacketsPerSecond` and plot it in a timeseries line chart​ 
```
NRQL> SELECT percentile(`host.net.transmitPacketsPerSecond`, 50, 75, 99) FROM Metric SINCE 8 hours ago TIMESERIES
```
What is the percentages of nodes used grouped by `host.hostname` and plot it in a timeseries line chart​ 
```
NRQL> SELECT (max(`host.disk.inodesUsed`)/max(`host.disk.inodesTotal`)) * 100 FROM Metric FACET `host.hostname` SINCE 8 hours ago TIMESERIES
```

AWS Integration 
===============

Permission Statement for AWS Policy 
```
{​ 

  "Version": "2012-10-17",​ 

  "Statement": [​ 

    {​ 

      "Effect": "Allow",​ 

      "Action": [​ 

        "budgets:ViewBudget"​ 

      ],​ 

      "Resource": "*"​ 

    }​ 

  ]​ 

}​ 
```