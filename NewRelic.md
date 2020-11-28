---
layout: default
title: NewRelic
---
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


AWS Integration 
===============

Link your AWS Account to New Relic
1. Go to one.newrelic.com  > Infrastructure > AWS​

2. From the IAM console, click Create role, then click Another AWS account:​

	a) For Account ID, use ```754728514883```.​

	b) Check the Require external ID box. ​ 
	For External ID, enter your New Relic account ID ​(Account Settings > API Keys). ​

	c) Do not enable the setting to Require MFA (multi-factor authentication).​

3. Attach the Policy ```ReadOnlyAccess​```

4. Name the role name as ```NewRelicInfrastructure-Integrations​```

5. Add Inline Policy with the following permission statement: 
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