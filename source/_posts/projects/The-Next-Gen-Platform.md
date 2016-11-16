---
title: The Next Generation Platform
date: 2016-10-22 14:45:56
banner: /images/iac-dcos.jpg
categories: Projects
Projects: Iac-platform
git: https://github.com/microservices-today/IaC-platform
---
### Overview
The Next Gen Platform(NGP) enables Infrastructure as Code (IaC) to provision and manage a complete infrastructure that set up a DC/OS cluster and applications on top of it. The complete infrastructure is setup on AWS Cloud. 
The Infrastructure as code is written using [Terraform][terraform]. 

The different modules that are installed and managed using this IaC are:
 a. OpenVPN Server
 b. DC/OS Cluster
 c. Docker Private Registry
 d. API Gateway
 e. ELK
 f. EC2 Container Registry
 g. Marathon Snapshot

### NGP Infrastructure Design

![NGP Infrastructure Design](/images/NGP_Architecture.png)

### Implementing Next Gen Platform

#### Pre-requisites
- AWS IAM account with administrator privileges (Save your AWS IAM access key ID and secret access key)
- Terraform (latest version)
- AWS CLI
- For the ease of Installation, we suggest using IaC-manager, to create a manager node. This IaC will create the following for you:
  - VPC
  - Management Subnet
  - Internet Gateway
  - IAM Role with required policies attached
  - Manager Node (CentOS)
  If not using IaC-manager, Please create the above resources (except Manager Node).
- Accept Software Terms of AWS Marketplace for CentOS/CoreOs/OpenVPN Access Server. 
- Setup Tyk Hybrid account from https://cloud.tyk.io/signup for running Tyk API gateway 
- A hosted zone in AWS Route53 for your domain name. This is required to create a record for creating a friendly dns name for the load balancers.

#### Steps
- Export AWS credentials as bash variables
```
export AWS_ACCESS_KEY_ID="anaccesskey"
export AWS_SECRET_ACCESS_KEY="asecretkey"
export AWS_DEFAULT_REGION="ap-northeast-1"  //(e.g. ap-northeast-1 for Tokyo and ap-southeast-1 for Singapore region)

```
- Clone the repo [IaC-platform][iac-platform]
- Change directory to IaC-platform
- Run ./configure.sh to decide which modules to run (The different modules are explained later in this blog)
- Copy terraform.dummy to terraform.tfvars
- If you are using [IaC-Manager][iac-manager], run 
```
cat ~/terraform.out >> ~/IaC-dcos/terraform.tfvars

```

- Modify params in `terraform.tfvars` for required modules
- (Optional) Modify params in `variable.tf` to change **default values** including subnet or add AMI accordingly to your aws region
- Run `terraform plan` to see the plan to execute.
- Run `terraform apply` to run the scripts.
- You may have `prod/dev/stage` configurations in `terraform.tfvars.{prod/dev/stage}` files (already ignored by `.gitignore`).

#### Steps to Import a New Module

- Create a folder with module-name in modules directory and add the following files within that folder:
  a. module-name.tf : File to create and manage module.
  b. module-name.dummy : Dummy values for the module.
  c. module-name-variables.tf : Variables required for the module.
  d. module-name-output.tf : Outputs to be displayed after execution.
- Add module-name to module array in configure.sh.

#### Monitoring DC/OS Cluster

- [Sysdig][sysdig] containers will be running in all DC/OS nodes for proper monitoring of instances through sysdig cloud.
- [Filebeat-Docker][filebeat] containers will be running in all dcos nodes for dcos and marathon log capturing. By installing Iac-Elk, you will be able to monitor the logs through AWS elasticsearch service.

#### Outputs

The following outputs will be generated after running the IaC:

- OpenVPN URL and credentials
- DC/OS URL

#### Notes
- Provide the latest stable version while running the “configure.sh” bash script.
- Always provide a uniquely identifiable pre-tag and post-tag for your deployments.
- Adhere to AWS naming conventions when providing names for AWS resources.

### OpenVPN Server

OpenVPN is an open-source software application that implements virtual private network (VPN) for creating secure connections to remote     access facilities. This platform will install an OpenVPN server using AWS OpenVPN Access Server AMI. Steps to setup OpenVPN server using terraform can be found at IaC-openvpn. 
Running the Next Generation Platform will give you the OpenVPN connect URL and Admin URL along with the credentials.

#### Connecting to VPN

- Install OpenVPN client on local machine.
- Download the client config file.
  a. Browse to https://OpenVPN_URL/ or https://OpenVPN_Public_IP
  b. Give the openvpn admin credentials
  c. Download the client.ovpn file by clicking Yourself(user-locked profile)
     ![OpenVPN Connect](/images/openVPN.png)
  d. Run the OpenVPN client with the downloaded client config file 
     ```
     openvpn --config client.ovpn
     
     ```
   
  e. For MaC: Follow the link [OpenVPN][openvpn]
  
  Once connected to the VPN, the users can access DC/OS web interface and Jenkins Web Interface. 
  Accessing Tyk dashboard does not require a VPN connection.
  
### DC/OS Cluster

DC/OS is a distributed operating system based on Apache Mesos. It enables the management of multiple machines and simplifies the installation and management of distributed services. 
The documentation for DC/OS can be found at DC/OS. Steps to set up a DC/OS cluster in AWS Cloud using terraform can be found at IaC-DC/OS.
After running the IaC for Next Generation Platform, you can launch the DC/OS web interface by entering the DC/OS Url (Given as output for IaC).

![DC/OS Web Interface](/images/DCOS.png)

### Tyk API Gateway

Tyk is an Open Source API Management Platform and Gateway. Our full Tyk stack consists of multiple components working together, the three most important ones are:

   - Tyk Gateway: This does all the heavy lifting, and is the actual proxy doing all the work
   - Tyk Dashboard: This is the GUI to control your gateways and view analytics, as well as an extended Dashboard REST API that enables granular integration
   - Tyk Pump: A data processor that moves analytics data from your gateways (redis) into other data sinks, most importantly MongoDB for the dashboard to process.

The git repo for setting Tyk on DC/OS using terraform is available at [IaC-api-gateway][iac-api-gateway]. The pre-requisite would be to setup Tyk Hybrid account from https://cloud.tyk.io/signup.

- Tyk dashboard is accessible at http://admin.cloud.tyk.io/ 
- Provide the credentials that were configured while running NGP
![Tyk API Gateway Dashboard](/images/Tyk-API-Gateway.png)
![Tyk Registered API](/images/Tyk-API.png)

### Docker Private Registry

The registry is a stateless, server side application that stores and distributes Docker images. NGP uses docker private registry to own the image distribution pipeline. It integrates image storage and distribution tightly into in-house deployment workflow. 
The git repo for running docker private registry on DC/OS is available at [IaC-dcos-docker-registry][iac-dcos-docker-registry].

Running NGP will create Docker Private Registry container with virtual IP: 192.168.0.1. The registry storage uses S3 bucket. 
The S3 bucket name and region are specified while running the IaC. The storage root directory is set to “/docker-registry”.
To test the app, follow the below steps:
a. Pull any public docker image
   `docker pull nginx`
b. Tag the image with the IP of docker private registry app
   `docker tag nginx 192.168.0.1/nginx`
c. Push the image to docker private registry
   `docker push 192.168.0.1/nginx`
   The above steps will create a folder named docker-registry inside your specified S3 bucket. The docker image should be available in that folder.
   
### ELK 

Elasticsearch, along with Logstash and Kibana, provides a powerful platform for indexing, searching and analyzing log data.
##### Filebeat
Filebeat is a lightweight, open source shipper for log file data. Filebeat tails logs and quickly sends this information to Logstash for further parsing and enrichment.
##### Filebeat-Docker
[Filebeat-Docker][filebeat-docker] IaC creates a filebeat docker image. This docker image is configured to monitor `mesos` and `marathon app` logs. It allows to pass the logstash uri as an environment variable to the docker.
AWS provides a kibana interface for elasticsearch module. For accessing the elasticsearch or kibana though browser you have to update the your system IP address to the elasticsearch access policy as shown below:

 ![Modify Access Policy](/images/ELK-Access-Policy.png)
 
 Kibana dashboard URL is available within AWS Elasticsearch Service as shown below:
 ![Kibana URL](/images/ELK-Kibana-URL.png)
 
 Accessing the Kibana URL will give you a dashboard as shown below:
 ![Kibana Dashboard](/images/Kibana-dashboard.png)
 
### Marathon Snapshot

[Marathon snapshot][iac-marathon-snapshot] IaC is used to take snapshots of marathon applications. These snapshots are stored in a S3 bucket (Bucket name is specified while running NGP). 
This is performed by running a Lambda function which is periodically triggered by a CloudWatch metric.
This app takes snapshots of marathon applications every hour.
The IaC, also creates an AWS API gateway to trigger the Lambda function in order to restore the marathon snapshot.

### EC2 Container Registry

EC2 Container Registry (ECR) is a fully-managed Docker container registry by Amazon. NGP will deploy a marathon container that performs the ECR-Login in regular intervals. It then places the compressed the docker config in a location (shared location across all the agents) where the marathon can access. 
It can be used as an alternative to the “Docker Private Registry” app discussed earlier. 
To test the app, follow the below steps:
a. Pull any public docker image
b. Tag the image with the ID of EC2 container registry
c. Push the image to EC2 container registry

### The Twelve Factors of Next Gen Platform

- Codebase

The code repository for “The Next Generation Platform” is always tracked in Git version control system. It is a distributed system with multiple codebases. 
Each component/module in NGP is an app and each individually comply with twelve-factor.

- Dependencies

NGP relies on explicit dependencies which are mentioned as pre-requisites.
It does not rely on implicit existence of system-wide packages.

- Config

The app specific configurations (such as credentials to external services, resource handles to caches) are not stored as constants in the code.
They are strictly separated from the code and they vary across environments(staging, production, development).

- Backing Services

A backing service is any service the app consumes over the network as part of its normal operation.
NGP uses the caching system Redis for Tyk API Gateway and binary asset service, Amazon S3.

- Code Release

The release processes for NGP are managed automatically using Jenkins. 

- Processes

The ‘Next Generation Platform’ runs applications as stateless docker containers.

- Port binding

The NGP executes the applications inside docker containers. These containers exports services by binding to a port and listening to requests coming on that port. 
The routing layer handles routing requests from a public-facing hostname to the port-bound processes.

- Concurrency 
 
 All the running apps in NGP can be scaled using the auto-scaling feature.

- Disposability

The apps run using NGP are disposable,meaning they can be started or stopped at a moment’s notice. 
This facilitates rapid deployment of code or config changes.

- Dev/prod parity

The NGP keeps its development, staging and production environments as similar as possible. 
The NGP is designed for continuous deployment by keeping the gap between production and development small.

- Logs

The NGP does not attempt to write or manage log files. Each running process writes its event stream to ‘stdout’. This platform has its own mechanism to rotate the logs generated by the apps. 
It uses Filebeat to collect the logs and is processed using ELK.

- Admin Processes

The admin/management processes are run in an identical environment as the NGP. They run against a release, using the same codebase and config. 
The developers use ssh or other remote command execution mechanism to run administration tasks.

### DevOps Best Practices

- Active participation : developers, operations staff, and support people must work closely together on a regular basis.
- Integrated change management : change management is the act of ensuring successful and meaningful evolution of the IT infrastructure to better support the overall organization. The NGP architecture is continuously evolved to ensure high availability.
- Continuous Integration : Continuous integration (CI) is the discipline of building and validating a project, through automated regression testing and sometimes code analysis whenever updated code is checked into the version control system. NGP uses Jenkins to ensure this.
- Integrated Deployment Planning : Experienced development teams will do deployment planning continuously throughout construction with active stakeholder participation from development, operations, and support groups.
- Continuous Deployment : With continuous deployment, when your integration is successful in one sandbox, your changes are automatically promoted to the next sandbox, and integration is automatically started there. NGP uses Jenkins for enabling Continuous deployment of applications.
- Application Monitoring : This is the operational practice of monitoring running solutions and applications once they are in production. NGP uses ELK along with file-beats for log monitoring. This also implements Sysdig for monitoring using Sysdig cloud.


[terraform]: <https://www.terraform.io/>
[iac-platform]: <https://github.com/microservices-today/IaC-platform>
[sysdig]: <http://www.sysdig.org/> 
[filebeat]: <https://github.com/microservices-today/filebeat-docker>
[openvpn]: <https://openvpn.net/index.php/access-server/docs/quick-start-guide/495-connecting-to-openvpn-access-server-using-the-connect-client-on-mac.html>
[iac-api-gateway]: <https://github.com/microservices-today/IaC-api-gateway>
[iac-dcos-docker-registry]: <https://github.com/microservices-today/IaC-dcos-docker-registry>
[filebeat-docker]: <https://github.com/microservices-today/filebeat-docker>
[iac-marathon-snapshot]: <https://github.com/microservices-today/IaC-marathon-snapshots> 


