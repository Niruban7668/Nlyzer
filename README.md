# Nlyzer
## Network Analyser ##
**What is OpenSearch?**
OpenSearch is the flexible, scalable, open-source way to build solutions for data-intensive applications. Explore, enrich, and visualize your data with built-in performance, developer-friendly tools, and powerful integrations for machine learning, data processing, and more.

The OpenSearch project, created by Amazon, is a forked search project based on old versions of Elasticsearch and Kibana. These projects were created primarily to support Amazon OpenSearch Service (formerly Amazon Elasticsearch Service). Amazon OpenSearch Service will not deliver current or future releases of Elasticsearch and Kibana.

**Setting up OpenSearch**
The machine I will be using here is a Ubuntu Server VM. Feel free to use your prefered machine, make sure you make modifications as per your machine.

Update the machine,

sudo apt update

Install JRE and JDK,

apt install default-jre default-jdk

Export the java path to the variable JAVA_HOME,

export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
echo $JAVA_HOME

Before installing and setting up OpenSearch make sure that vm.max_map_count is set to at least 262144.

cat /proc/sys/vm/max_map_count


Add vm.max_map_count=262144 in /etc/sysctl.conf file

Reload to update the changes using,

sudo sysctl -p

Download OpenSearch from official website or else use package managers for apt to install as mentioned in documentation for other package manager refer to the documentation. Copy the link of the download button and use wget to install.

Unzip the downloaded file,

sudo dpkg -i opensearch-2.11.1-linux-x64.deb

Reload and enable opensearch,

sudo systemctl daemon-reload
sudo systemctl enable opensearch

Start and check the status of OpenSearch

sudo systemctl start opensearch
sudo systemctl status opensearch

Curl the opensearch url running on port 9200, with username and password as admin.

curl -X GET https://localhost:9200 -u 'admin:admin' --insecure

In opensearch.yml file, located in /etc/opensearch make the following modifications,

network.host: 0.0.0.0
discovery.type: single-node
plugins.security.disabled: false
In jvm.options, located at /etc/opensearch modify the heap size as per the requirement.

To run the OpenSearch using SSL, remove all the pem files from /etc/opensearch and generate self signed certificates.

Update the generated certificates in opensearch.yml file using the below script and make sure you comment out the duplicates.

Change the directory to, /usr/share/opensearch/plugins/opensearch-security/tools and run hash.sh to generate hash for a new password, note the hash for the password generated.

cd /usr/share/opensearch/plugins/opensearch-security/tools
./hash.sh

Now change the directory to /etc/opensearch/opensearch-security, modify internal_users.yml. Comment out all the users except admin and change the hash in the admin to the hash previously generated.

Now restart and check the status of OpenSearch

sudo systemctl restart opensearch
sudo systemctl status opensearch

Change the directory to /usr/share/opensearch/plugins/opensearch-security/tools and run securityadmin.sh to update the certificates as given below,

cd /usr/share/opensearch/plugins/opensearch-security/tools
OPENSEARCH_JAVA_HOME=/usr/share/opensearch/jdk ./securityadmin.sh -cd /etc/opensearch/opensearch-security/ -cacert /etc/opensearch/root-ca.pem -cert /etc/opensearch/admin.pem -key /etc/opensearch/admin-key.pem -icl -nhnv

Now curl the opensearch running on port 9200,

curl https://<IP>:9200 -u admin:<new password> -k

Once you get an output similar to that of the one above, you have successfully installed and configured opensearch.

