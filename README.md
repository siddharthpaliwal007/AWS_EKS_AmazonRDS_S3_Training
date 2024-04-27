# AWS_EKS_AmazonRDS_S3_Training

This project demonstrates:

- Setting up an application running using Amazon Elastic Kubernetes Service(EKS), Amazon Relational Database Service(RDS) and Amazon S3 Services. 
- How to deploy infrastructure as a code using Terraform.
- Leverage Kubernetes and Docker to host our application in a containerized fashion
- How to migrate on-premises application files and database to an AWS environment.

## Solution Architecture

![SA](https://github.com/siddharthpaliwal007/AWS_EKS_AmazonRDS_S3_Training/assets/25987777/fae905e2-d92a-4215-bcca-8739903fdb73)

## Requisites

AWS account, AVOID using root user account.

## Steps

Download project files ⬇️

Set DB password variable
```
export TF_VAR_db_password="welcome123456"
```
​
Provision the resources with Terraform
```
terraform init
terraform plan
terraform apply
```

Connect to the database
```
mysql --host=<public_ip_address> --port=3306 -u app -p
```
​
Prepare database

Create user:
```
CREATE USER app@'%' IDENTIFIED BY 'welcome123456';
GRANT ALL PRIVILEGES ON dbcovidtesting.* TO app@'%';
FLUSH PRIVILEGES;
```
​
Create table:
```
CREATE TABLE `records` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `guest_name` varchar(200) NOT NULL,
  `home_country` varchar(200) DEFAULT NULL,
  `testing_type` varchar(200) DEFAULT NULL,
  `testing_result` varchar(200) DEFAULT NULL,
  `pdf` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1;
```

Prepare Docker image
```
docker login
docker build -t luxxy-covid-testing-system-app-en:latest .
docker tag luxxy-covid-testing-system-app-en:latest thecloudbootcamp/luxxy-covid-testing-system-app-en:latest
docker push thecloudbootcamp/luxxy-covid-testing-system-app-en:latest
```
​Reason: We push image to our DockerHub account. So that on Amazon EKS , we could directly download image thorough their. 

Prepare K8S deployment file
- Change image name to DockerHub link (Line 33)
- Change S3 bucket name (Line 42)

Connect to AWS EKS K8S
```
rm ~/.kube/config
aws eks update-kubeconfig --name <CLUSTER_NAME> --region <REGION>
kubectl cluster-info
kubectl get nodes
```
​
Deploy app to K8s
```
cd ../luxxy-kubernetes
kubectl apply -f luxxy-covid-testing-system.yaml
```

Note: While provisioning there occurs an issue, because its puts an extra tag while one tag is already there. More information about issue and its solution is listed below under "Known Issue" topic. 

Test it
```
kubectl get svc
```
​
Destroy
```
kubectl delete svc luxxy-covid-testing-system
kubectl delete deployment luxxy-covid-testing-system
terraform destroy
```

## Known Issue
Issue:
External IP is not populated - https://github.com/kubernetes/kubernetes/issues/73906

Solution:
Remove "[kubernetes.io/cluster/](http://kubernetes.io/cluster/)<CLUSTER_NAME>" of security group "EKS node shared security group"


## Acknowledgements

Special Thanks to @Jean Rodrigues and the Cloud Bootcamp Team for conducting this special training session. 
