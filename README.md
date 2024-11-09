# Nlyzer
## Network Analyser ##
**What is OpenSearch?**
OpenSearch is the flexible, scalable, open-source way to build solutions for data-intensive applications. Explore, enrich, and visualize your data with built-in performance, developer-friendly tools, and powerful integrations for machine learning, data processing, and more.

The OpenSearch project, created by Amazon, is a forked search project based on old versions of Elasticsearch and Kibana. These projects were created primarily to support Amazon OpenSearch Service (formerly Amazon Elasticsearch Service). Amazon OpenSearch Service will not deliver current or future releases of Elasticsearch and Kibana.

## 1. **Setting up OpenSearch**
The machine I will be using here is a Ubuntu Server VM. Feel free to use your prefered machine, make sure you make modifications as per your machine.

Update the machine,
```
sudo apt update
```
Install JRE and JDK,
```
apt install default-jre default-jdk
```
Export the java path to the variable JAVA_HOME,
```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
echo $JAVA_HOME
```
Before installing and setting up OpenSearch make sure that vm.max_map_count is set to at least 262144.
```
cat /proc/sys/vm/max_map_count
```

Add vm.max_map_count=262144 in /etc/sysctl.conf file

Reload to update the changes using,
```
sudo sysctl -p
```
Download OpenSearch from official website or else use package managers for apt to install as mentioned in documentation for other package manager refer to the documentation. Copy the link of the download button and use wget to install.

Unzip the downloaded file,
```
sudo dpkg -i opensearch-2.11.1-linux-x64.deb
```
Reload and enable opensearch,
```
sudo systemctl daemon-reload
sudo systemctl enable opensearch
```
Start and check the status of OpenSearch
```
sudo systemctl start opensearch
sudo systemctl status opensearch
```
Curl the opensearch url running on port 9200, with username and password as admin.
```
curl -X GET https://localhost:9200 -u 'admin:admin' --insecure
```
In opensearch.yml file, located in /etc/opensearch make the following modifications,
```
network.host: 0.0.0.0
discovery.type: single-node
plugins.security.disabled: false
In jvm.options, located at /etc/opensearch modify the heap size as per the requirement.
```
To run the OpenSearch using SSL, remove all the pem files from /etc/opensearch and generate self signed certificates.

Update the generated certificates in opensearch.yml file using the below script and make sure you comment out the duplicates.

Change the directory to, /usr/share/opensearch/plugins/opensearch-security/tools and run hash.sh to generate hash for a new password, note the hash for the password generated.
```
cd /usr/share/opensearch/plugins/opensearch-security/tools
./hash.sh
```
Now change the directory to /etc/opensearch/opensearch-security, modify internal_users.yml. Comment out all the users except admin and change the hash in the admin to the hash previously generated.

Now restart and check the status of OpenSearch
```
sudo systemctl restart opensearch
sudo systemctl status opensearch
```
Change the directory to /usr/share/opensearch/plugins/opensearch-security/tools and run securityadmin.sh to update the certificates as given below,
```
cd /usr/share/opensearch/plugins/opensearch-security/tools
OPENSEARCH_JAVA_HOME=/usr/share/opensearch/jdk ./securityadmin.sh -cd /etc/opensearch/opensearch-security/ -cacert /etc/opensearch/root-ca.pem -cert /etc/opensearch/admin.pem -key /etc/opensearch/admin-key.pem -icl -nhnv
```
Now curl the opensearch running on port 9200,
```
curl https://<IP>:9200 -u admin:<new password> -k
```
Once you get an output similar to that of the one above, you have successfully installed and configured opensearch.

## 2. **Setting up OpenSearch Dashboards**

Download OpenSearch from official website or else use package managers for apt to install as mentioned in documentation for other package manager refer to the documentation. Copy the link of the download button and use wget to install.
```
wget https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/2.11.1/opensearch-dashboards-2.11.1-linux-x64.deb
```
Unzip the downloaded file,
```
sudo dpkg -i opensearch-dashboards-2.11.1-linux-x64.deb
```
Reload and enable, opensearch. Start and check the status of Opensearch
```
sudo systemctl daemon-reload
sudo systemctl enable opensearch-dashboards
sudo systemctl start opensearch-dashboards
sudo systemctl status opensearch-dashboards
```
Edit opensearch-dashboards.yml file in /etc/opensearch-dashboards and add the following code.
```
server.port: 5601
server.host: "0.0.0.0"
opensearch.hosts: ["https://0.0.0.0:9200"]
opensearch.ssl.verificationMode: none
opensearch.username: "admin"
opensearch.password: "<Pass>"
```
Save the file and exit, now restart opensearch-dashboards.
```
sudo systemctl restart opensearch-dashboards
```
Now try to access the opensearch-dashboard, running on port 5601,

Login using the username and password supplied in yml configuration file.

## 3. **Zeek Installation**

Update and upgrade the ubuntu using apt.
```
sudo apt-get update
sudo apt-get upgrade
```
Install dependencies using the below command.
```
apt-get install -y --no-install-recommends g++ cmake make libpcap-dev
```
Now for Ubuntu 22.04 based machines, use the following commands to add zeek repository into binary packages.
```
echo 'deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/ /' | sudo tee /etc/apt/sources.list.d/security:zeek.list
curl -fsSL https://download.opensuse.org/repositories/security:zeek/xUbuntu_22.04/Release.key | gpg  --dearmor | sudo tee /etc/apt/trusted.gpg.d/security_zeek.gpg > /dev/null
```
Now run the update command and install command to install zeek
```
sudo apt update
sudo apt install zeek
```
Once zeek is successfully installed, change the directory to /opt/zeek, export the zeek home directory path into .bashrc file.
```
nano ~/.bashrc
export PATH=/opt/zeek/bin:$PATH

source ~/.bashrc
which zeek
zeek --version
```
Now change directory to /opt/zeek/etc and edit node.cfg file, replace interface with system network interface, you can find your system network interface using the below command
```
ifconfig
```

Now change the directory to /opt/zeek/bin, and run the below command to check the zeekctl script,
```
./zeekctl check
```
Run the following command to deploy and check status of zeekctl,
```
./zeekctl deploy
./zeekctl status
```
You can check the log file from the directory, /opt/zeek/logs/current for the present logs.

## 4. **Sending Zeek Logs to OpenSearch using Logstash**

first let us install logstash in the machine with zeek.
```
wget https://artifacts.opensearch.org/logstash/logstash-oss-with-opensearch-output-plugin-8.9.0-linux-x64.tar.gz
```
Unzip the file, using the below command.
```
tar -zxvf logstash-oss-with-opensearch-output-plugin-8.9.0-linux-x64.tar.gz
```
Now before we run logstash let’s first install OpenSearch-Dashboard out plugin for logstash, which we will use in the pipeline configuration,
```
cd logstash-8.9.0
bin/logstash-plugin install logstash-output-opensearch
```
Once the plugin is installed, test the logstash using the below command and enter “Hello World” in the same terminal once command is executing, as shown below you must get an output.
```
bin/logstash -e "input { stdin { } } output { stdout { } }"
```
Since the logstash is working as expected, let’s go ahead and write the pipeline file for the zeek, save the file in config directory.
```
input {
  stdin {
    codec => json
  }
  file {
    path => "/opt/zeek/logs/current/*.log"
    # Change the path to where the zeek logs are stored
  }
}

output {
  opensearch {
    hosts => "https://<address>:9200"
    # Change the address to your OpenSearch address
    auth_type => {
     type => 'basic'    
     user => 'admin'
     password => ‘<password>'
    }
    index => "zeek-%{+YYYY.MM.dd}"
    ssl_certificate_verification => false
  }
}
```
Save and exit, run logstash with below command. Replace all the relevant details, modify the code as per the needs.
```
bin/logstash -f config/pipeline.conf --config.reload.automatic
```
You will get a output similar to that of shown below and create a index in OpenSearch Dashboard and the logs will be shown in dashboard as well.


Now access OpenSearch-Dashboards page to create index for zeek logs,


Now you can view the logs in the discover section once the index is setup.
