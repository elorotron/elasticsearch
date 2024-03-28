# Elastic search compose file, and migration indecies from elasticsearc to opensearch/elasticsearch on any version
# 1. Install requirements
### npm
### elasticdump
### kubectl (optional if your elasticsearch located in k8s without public access)
### aws cli (optional if your elasticsearch located in k8s without public access)
## 1.1 NPM
### sudo apt update
### sudo apt install -y npm
## 1.2 Elasticdump (https://github.com/elasticsearch-dump/elasticsearch-dump)
### sudo npm install elasticdump -g
## 1.3. Kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) (Optional if you already have kubectl with access to EKS cluster)
### sudo apt-get update
### sudo apt-get install -y apt-transport-https ca-certificates curl
### curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
### echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
### sudo apt-get update
### sudo apt-get install -y kubectl
### kubectl version --client
## 1.4. AWS CLI (optional if you already have aws cli with access to EKS cluster)
### curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
### sudo apt install zip
### unzip awscliv2.zip
### sudo ./aws/install
### aws --version
### aws configure (and paste your credentials with access to EKS, output is optional)
# 2. Backup required indices
## 2.1. Make directory where you wan’t save indices, for example:
### mkdir /home/ubuntu/backup
## 2.2. Take indices
### multielasticdump --direction=dump --match='<NAME_FOR_INDEX>$' --input=http://<elasticsearch-endpoint> --output=/home/ubuntu/backup
#### –direction=dump - get indices into .json file
#### –output - where to save dumped indices
#### –match - regex expression for indices which you won't backup
#### '^.*$'  - mean all indices are dumped
#### ‘example$’ - mean only indices with name “example” are dumped
#### –input - link to your endpoint (if your elastic search open localhost:9200, or https://<elasticsearch-endpoint>)
#### important: if your elasticsearch/opensearch required some auth, you need paste link like this: https://<login>:’<Paswword>’@<elasticsearch-endpoint>
## 2.3. Verify that indices are dumped in your folder
### ls -la /home/ubuntu/backup
# 3. Restore dumped indices on opensearch
## 3.1. Restore indices
### multielasticdump --direction=load --input=/home/ubuntu/backup --output=https://<login>:'<password>'@<elasticsearch-endpoint> --match='^.*$'
#### –direction=load - means that your upload your dumped indices into opensearch
#### – match means that all indices located in folder “backup” will be pushed to the –input (opensearch)
# 4. Verify that all indices are uploaded
## 4.1 GET query
### curl -XGET -u <login>:'<password>' <domain_endpoint>/_cat/indices