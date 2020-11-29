---
layout: default
title: Splunk
---
Registration Splunk Cloud instance
==================================
[Registration](https://www.splunk.com/page/sign_up/cloud_trial?redirecturl=/getsplunk/cloud_trial)

[Go to Instance page](https://splunkcommunities.force.com/customers/apex/RMEC_InstancePage)


Instalation Splunk enterprise
=====================
```
sudo yum install wget

wget -O splunk-8.1.0.1-24fd52428b5a-linux-2.6-x86_64.rpm 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=8.1.0.1&product=splunk&filename=splunk-8.1.0.1-24fd52428b5a-linux-2.6-x86_64.rpm&wget=true'

sudo rpm -i splunk-8.1.0.1-24fd52428b5a-linux-2.6-x86_64.rpm

cd /opt/splunk/

sudo ./bin/splunk start --accept-license

sudo ./bin/splunk enable boot-start
```
Open http://your-ip:8000/

Set up receiving with Splunk Web

Use Splunk Web to set up a receiver:

1. Log into the receiver as admin or an administrative equivalent.
1. Click Settings > Forwarding and receiving.
1. At Configure receiving, click Add new.
1. Specify the TCP port you want the receiver to listen on (the listening port, also known as the receiving port). For example, if you enter "9997," the receiver listens for connections from forwarders on port 9997. You can specify any unused port. You can use a tool like netstat to determine what ports are available on your system. Make sure the port you select is not in use by splunkweb or splunkd.
1. Click Save. Splunk software starts listening for incoming data on the port you specified.

1. Settings -> Server settings -> General settings -> Pause indexing if free disk space (in MB) falls below
Change to 100MB
1. Restart Splunk

Instalation forwarder
=====================
```
sudo yum install wget

wget -O splunkforwarder-8.1.0-f57c09e87251-linux-2.6-x86_64.rpm 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=8.1.0&product=universalforwarder&filename=splunkforwarder-8.1.0-f57c09e87251-linux-2.6-x86_64.rpm&wget=true'

sudo rpm -i splunkforwarder-8.1.0-f57c09e87251-linux-2.6-x86_64.rpm

cd /opt/splunkorwarder/

sudo ./bin/splunk start --accept-license

sudo ./bin/splunk enable boot-start
```

After you complete the installation of the universal forwarder, you must configure it before it can do anything.

You can configure the forwarder from the command line or by using configuration files. If you want to configure from the command line, the forwarder must be running.

1. Start the universal forwarder, accept the license agreement, and provide credentials. 
1. Configure the universal forwarder, either from the command line or with a configuration file. output.conf
1. Restart the forwarder service to enable the configuration changes you made.

```
./splunk add forward-server <host name or ip address>:<listening port>
```

Upload the data
===============



1. Install Nginx
```
sudo yum install epel-release
sudo yum install nginx
sudo service nginx start
```
2. Add New Splunk monitors to forwarder for nginx logs

```
sudo ./bin/splunk add monitor /var/log/nginx/access.log -index nginx -sourcetype nginx:access
sudo ./bin/splunk add monitor /var/log/nginx/error.log -index nginx -sourcetype nginx:error
```
3. Install jenkins

```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins java-1.8.0-openjdk-devel
sudo systemctl daemon-reload
sudo systemctl start jenkins
```

3. Add New Splunk monitors to forwarder for jenkins logs

```
sudo ./bin/splunk add monitor /var/log/jenkins/jenkins.log -index jenkins -sourcetype jenkins
```

Source types and fields
=======================

Field extraction:
```
Jenkins:
	^[^\t\n]*\t(?P<Severity>\w+)\t(?P<component>[^:]+):\s+(?P<message>.+)

nginx:access:
    ^(?:[^"\n]*"){5}(?P<Agent>[^"]+)

nginx:error:
    ^(?:[^:\n]*:){5}\s+(?P<Client_IP>[^,]+)[^"\n]*"(?P<Method>\w+)\s+(?P<URL>/\w+)
```


SPL
===
1. Field-value pair matching
This example shows field-value pair matching for specific values of source IP (src) and destination IP (dst).

```
search src="10.9.165.*" OR dst="10.9.165.8"
```

2. Using boolean and comparison operators
This example shows field-value pair matching with boolean and comparison operators. This example searches for events with code values of either 10, 29, or 43 and any host that is not "localhost", and an xqp value that is greater than 5.

```
search (code=10 OR code=29 OR code=43) host!="localhost" xqp>5
```

An alternative is to use the IN operator, because you are specifying multiple field-value pairs on the same field. The revised search is:

```
search code IN(10, 29, 43) host!="localhost" xqp>5
```

3. Using wildcards
This example shows field-value pair matching with wildcards. This example searches for events from all of the web servers that have an HTTP client and server error status.

```
search host=webserver* (status=4* OR status=5*)
```

An alternative is to use the IN operator, because you are specifying two field-value pairs on the same field. The revised search is:

```
search host=webserver* status IN(4*, 5*)
```

4. Using the IN operator
This example shows how to use the IN operator to specify a list of field-value pair matchings. In the events from an access.log file, search the action field for the values addtocart or purchase.

```
search sourcetype=access_combined_wcookie action IN (addtocart, purchase)
```

5. Using the NOT or != comparisons
Searching with the boolean "NOT" comparison operator is not the same as using the "!=" comparison.

The following search returns everything except fieldA="value2", including all other fields.

```
search NOT fieldA="value2"
```

The following search returns events where fieldA exists and does not have the value "value2".

```
search fieldA!="value2"
```

If you use a wildcard for the value, NOT fieldA=* returns events where fieldA is null or undefined, and fieldA!=* never returns any events.


Visualisation and dashboarding
=========================================

```
index= "nginx" | timechart count
index= "nginx" | timechart count by sourcetype
index= "jenkins" | timechart count by component
index= "jenkins" | table component
index= "jenkins" | stats count by host
index= "jenkins" OR index="_internal"  | stats count by host
index= "jenkins" | dedup component | table component

```
