CICD deployment pipeline using Jenkins

Types of Deployment

1. Local Deployment
2. Development Deployment
3. Production Deployment

deployments types image

1. Local Deployment

Steps

1.1 Create a AWS EC2 instance
1.2 SSH into instance and run the commands - sudo apt update - git clone repo_url - Install node - Install dependencies - run commands
1.3 Add environment variables
1.4 Copy the IP address + port number application is running
1.5 Application is live

2. Development Deployment

Steps

2.1 Create 2 AWS EC2 Instance named Jenkins and SonarQube
2.2 SSH into Jenkins Instance and run the commands - sudo apt update - Install Java - Install Jenkins - Install Docker
2.3 Get password for Jenkins - cat var/lib/jenkins/secrets/initialAdminPassword
2.4 Give permission to run the docker commands - sudo chmod 666 /var/run/docker.sock
2.5 Open Jenkins (IP address : 8080) and paste the password
2.6 Install plugins
2.7 SSH into SonarQube Instance and run the commands - sudo apt update - Install Docker
2.8 Give permission to run the docker commands - sudo chmod 666 /var/run/docker.sock
2.9 Run sonarqube container with port mapping and in dedatched mode
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
2.11 To ckeck (IP address : 9000) username = admin and password = admin for first time login
2.12 Inside Jenkins install plugins - Node - Docker - Kubernetes - Docker pipeline - Kubernetes CLI
2.13 Trivy plugin is not avaliable inside jenkins then install it on jenkins instance which will be avaliable to use inside jenkins
2.14 Configure Plugin
[Jenkins --> Tools --> SonarQube and Node and Docker --> Apply]
2.14 Create a administator token from sonarqube which will be used by jenkins sonarqube plugin
2.15 Save the token in the credentials --> global cerdentials --> add credentials --> Select Secret text --> Secret and ID -->Create
2.15 If GIT repo is private --> Create a token from github --> Developer setting --> personal access token --> Generate token --> select scope
2.16 Create a pipeline in jenkins
2.16 Pipeline scripts --> Define stages --> Add tools
2.17 From pipeline syntax add credentials for git
[Snippet Generator --> git --> repo URL --> Branch --> Credentails --> github_username --> password = token_generated --> Add --> Generate pipline snippit --> copy and paste in pipline scipt]
2.18 Configure sonarqube server inside jeniks [system --> sonarqube installation --> server ULR (IP address of the SonarQube instance) --> add token --> Add]
2.17 From pipeline syntax add credentials for docker
[Snippet Generator --> docker --> docker registry URL --> Add credentials --> dockerhub_username and docker_hub_password --> Add --> Generate pipline snippit --> copy and paste in pipline scipt]
2.20 Build Pipeline

3. Production Deployment

3.1 Inside Jenkins Instance
3.2 SSH into instance and Install CLIs - AWS CLI - KUBECTL - EKSCTL
3.3 Create a IAM user and polices - EC2fullaccess - EKS_CNI_Policy - EKSClusterPolicy EKSWorkerNodePolicy FormationFullAccess IAMFullAccess and inline policy
3.4 create a Access key --> CLI --> create access key --> downlaod key
3.5 SSH Instance and run the commands -- aws configure --> access_id -- sccess key -- region
3.6 Create EKS cluster using run commands -- name + region + zones + without-nodes-group
3.7 eksctl utils associate-iam-oidc-provider \
 -- region region_name
-- cluster cluster_name
-- approve
3.8 Creatte node group
eksctl create nodegroup -- cluster=cluster_name
-- region = region
-- name = node2
-- nodes = 3
-- nodes-min= 2
-- nodes-max = 4
--node-volume-sixe =20
--ssh-access
--ssh-public-key = key
-- manged
-- external-dns-access
-- full-ecr--access
-- asg-access
3.9 To check nodes are running --> kubectl get nodes
3.10 Inside AWS EKS Cluster --> Network --> control plane and worker node groups --> est all traffic allow
3.11 Create RBAC
-- Cretate Service account
-- create role
-- bind role to service account
-- Generate secret using servie account
-- Store env variables inside vault.
3.12 Add stage in the pipeline script
[Snippet Generator --> withKubeCrediential --> kind --> secret text --> add seceret created in service account --> kubernetes api endpoint from AWS --> cluster name --> namespace --> Add --> Generate pipline snippit --> copy and paste in pipline scipt]
3.13 Cretare another stage too verify the deployment
3.14 Built Pipeline
3.15 Check for logs for pods
