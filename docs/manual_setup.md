![alt text](https://ineuron.ai/images/ineuron-logo.png)

# Manual Deployment of Sensor Fault Detection Project

## AWS CLI Installation

### Install AWS CLI in Linux System

To install awscli in linux system, execute the following commands

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

```bash
sudo apt install awscli
```

### Install AWS CLI in Windows System

To install awscli in windows system, follow the process, 

Install the msi package of awscli

```bash
https://awscli.amazonaws.com/AWSCLIV2.msi
```

Once the download and double click on the installer, and click on next and agree to the terms and conditions, and click on next and click on install button. Once the installation is done, click on finish and proceed to the futher steps

### Install AWS CLI in MacOS System

To install awscli on MacOS system, execute the following commands

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

```bash
sudo installer -pkg AWSCLIV2.pkg -target /
```

### Check AWS CLI installation

To check aws cli is working fine or not, execute the following commands

```bash
aws --version
```

# Manual deployment of the application to instance

## Login to AWS Console

First step is to login to the console, you can open aws console in your browser and login with your credentials

![](images/aws_login_page.png)

## Creating IAM role for getting access to AWS resources

Once you are logged into you aws account, you need to create IAM user for getting credentials

![](images/aws_iam_page.png)

On click on IAM, and you will redirected to IAM console, from there click on add users 

![](images/aws_iam_console.png)

![](images/aws_iam_add_users.png)

You need to add user details like user name with programmatic access. Once that is done, click on next permissions

![](images/aws_iam_user_settings.png)

With that done, we need attach policies to the user to grant accesss to AWS resources. Some of the policies are S3 access, EC2 access and ECR access

![](images/aws_iam_add_ec2_and_ecr_full_access.png)

![](images/aws_iam_add_s3_full_access.png)

Once the policies are attached, review the user name and policies attached to the user

![](images/aws_iam_review_user.png)

Once the user name and policies are reviewed, click on download csv, and there are csv file which contains your AWS credentials. Kindly keep this in secret and do not share with anyone

![](images/aws_iam_user_success.png)

## Create Simple Storage Service Bucket for storing data 

Go to services and select the S3 service, next you will redirected to S3 console, and click on create bucket button

![](images/aws_s3_service.png)

![](images/aws_s3_console_page.png)

On clicking the create bucket button, we will redirected to s3 bucket settings page, we need to put the bucket name and enable some bucket settings.

Note - This process is same for creating another bucket also

Bucket Name - [unique-name]-sensor-model, [unique-name]-sensor-datasource

![](images/aws_s3_create_bucket_name.png)

Uncheck block public access

![](images/aws_s3_create_bucket_block_access.png)

Enable Bucket Versioning

![](images/aws_s3_create_bucket_versioning.png)

Once the bucket settings are implemented, we can create the bucket

![](images/aws_s3_create_bucket_button.png)

## Creating Elastic Compute Cloud instance for deployment

Go to services and select the EC2 service, and then click on the instances button and you will use a list of instances, by default no instances will be running. Click on the launch instance button.

![](images/aws_ec2_services_page.png)

![](images/aws_ec2_console.png)

![](images/aws_ec2_instances_page.png)

Once you click on the launch instance button, you will see launch instance page, where you need fill the instance details.

Tag Name - Application Server

AMI - Deep Learning AMI, 20.04

Instance Type - t2.large

Key Pair name - ineuron

Network Security Group - 22, 8080

![](images/launch_aws_instance_tag.png)

![](images/launch_aws_instance_ami.png)

![](images/launch_aws_instance_deep_learning_ami.png)

![](images/launch_aws_instance_instance_type.png)

For generating the key pair click on create a new key pair

![](images/launch_aws_instance_key_pair.png)

Coming to the network settings, click on edit and then in the inbound security groups rule click on add security group role and in port range write 8080, in the source select 0.0.0.0/0. Once that is done, click on launch instance and then after some time EC2 instance will reach running state.

![](images/launch_aws_instance_sg.png)

![](images/launch_aws_instance_8080_port_sg.png)

![](images/launch_aws_instance_sg.png)

![](images/launch_aws_instance_success.png)

After the launch is completed, we can see in the ec2 console that EC2 instance is in running state

![](images/aws_instance_running_page.png)

## GitHub Runner Setup

### Note - This steps are same for manual and automated deployments. If you are doing automated deployments, follow from these steps

Once the instance is running, click on the instance id and then click on connect and copy the ssh command. Open any SSH client like Putty, Mobaxterm or powershell terminal and change directory to the place where the pem file is present, and paste the command and then click on enter. Type yes on prompted.

![](images/aws_instance_connect.png)

![](images/aws_instance_ssh_connect.png)

We are using git bash terminal, to make an SSH connection to AWS EC2 instance.

![](images/aws_instance_connect_ssh_git_bash.png)

After that you will be successfully connected to EC2 instance via SSH.

![](images/aws_instance_connect_success.png)

Once the connection is successfully established with EC2 instance, we can setup the github runner for automated deployments via github actions. In order to setup runners, we need to execute the bash scripts present in the scripts/setup_runner.sh

First in order to run the scripts, we need to get the github token for our repo. The github token can be found in runners sections and settings page. Follow the screenshots to get the token

![](images/getting_github_token_part_1.png)

![](images/getting_github_token_part_2.png)

![](images/github_runner_page.png)

![](images/github_runner_pat.png)

Copy the github actions, highlighted the screenshot and store it somewhere safe. Once that is done, we can download the scripts to EC2 machine and execute them with the github token, we previously copied.

Navigate to the scripts folder in the repo and find the setup_runner.sh, click on raw button and copy the url, which is avaiable in the browser

![](images/setup_runner_script_github.png)

![](images/setup_runner_content.png)

Once we get the url of the scripts, go to the git bash terminal where ssh connection has been made. execute the commands. After the script is downloaded, run the downloaded scripts with github token

```bash
wget https://raw.githubusercontent.com/Machine-Learning-01/sensor-fault-detection/main/scripts/setup_runner.sh
```

Put your github token, which was previously copied from the runner page

```bash
bash setup_runner.sh <github-token>
```

![](images/get_execution_scripts_in_ssh.png)

![](images/run_setup_script_with_token.png)

After successfull execution of the scripts, we can see in github runner page, there will be runner named self-hosted with idle status that means, runner setup was successfully done. 

![](images/runner_status_check.png)

Once runner is successfully setup with github, we can use github workflows to automate CI-CD tasks. 

## Create, Build and Push container image Elastic Container Registry

We need to use Elastic Container Registry to store the container images in the cloud, where it will be secured. In order to create ECR, follow the steps mentioned in the screnshots

Navigate to the AWS console and in the search bar, type ecr and click on Elastic Container Registry button. You will be redirected to ecr console. Click on get started, next give the repo name, and click on create repository and ecr repo will be created.

![](images/aws_ecr_page.png)

![](images/aws_ecr_console.png)

![](images/aws_ecr_create_repo.png)

Once the ecr repo is created, we need to build and push the image to ECR, in order to do so, follow the screenshot attached 

![](images/aws_ecr_image_push_commands.png)

Follow the commands mentioned in your console for building and pushing the image to ECR.

![](images/aws_ecr_image_build.png)

On successfull execution of the commands, we will be able to see the image present in the ecr repo with tags

![](images/aws_ecr_image_push_success.png)

## Running the container image in EC2 instance 

On the build and push of the image is done, we need to pull the image from ECR and then run the image to check whether the container image is working fine or not. In order to do so first we need to install aws cli in EC2 machine and then configure our 
credentials

Install awscli in EC2 machine, and configure your aws credentials

```bash
sudo apt install awscli -y
```

![](images/install_awscli_in_ec2.png)

```bash
aws configure
```

![](images/aws_configure_in_ec2.png)

Put your credentials which was downloaded previously as csv file. You need give AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY and AWS_DEFAULT_REGION. For Default output format give json

![](images/post_aws_configure.png)

Once the aws credentials are configured, we can login to ecr and then pull the image from ecr and run the container image.

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your_account_id>.dkr.ecr.us-east-1.amazonaws.com
```

![](images/aws_ecr_login_for_pull_image.png)

```bash
docker push <your_account_id>.dkr.ecr.us-east-1.amazonaws.com/test:latest
```

![](images/aws_ecr_pull_image.png)

Once the image is pulled from ECR and we can run the image with env variables and check whether the application is running fine or not.

```bash
docker run -d -e AWS_ACCESS_KEY_ID="<AWS_ACCESS_KEY_ID>" -e AWS_SECRET_ACCESS_KEY="<AWS_SECRET_ACCESS_KEY>" -e AWS_DEFAULT_REGION="<AWS_DEFAULT_REGION>" -e MONGO_DB_URL="<MONGO_DB_URL>" -p 8080:8080 <your_account_id>.dkr.ecr.us-east-1.amazonaws.com/<your_repo_name>:latest
```

![](images/aws_ecr_image_run.png)

On successfull execution of the command, the container image will generated as random string. To check whether the container is running fine or not, execute the following commands

```bash
docker ps -a
```

![](images/container_status.png)

Now that container image is running successfully, we can go to the ec2 console and in the created instance, we can grab the public ip of the instance and paste that in the browser with port 8080. Eg - public-ip:8080 and press enter you should be able to see the fastapi docs page with train and predict routes.

![](images/aws_instance_running_page.png)

![](images/aws_ec2_instance_summary.png)

![](images/aws_ec2_instance_public_ip_with_port.png)

![](images/fastapi_docs_page.png)

Once you are able to able fastapi docs page, we will be using two routes, one is train and another is predict routes. You can hit those routes depending on your choice.

## Deleting the Elastic Container Registry Repository

Once the deployment and testing is done, in the cloud it is time to delete the resources which were created. To delete the ecr repo, follow the steps

![](images/aws_ecr_delete_repo.png)

On successfull execution, the ecr repo will be deleted

## Deleting the Simple Storage Service Bucket 

To delete the s3 bucket, follow the steps

Note - these steps are applicable for both the buckets

Before deleting the bucket, we need to make sure that the all the objects are deleted from the bucket. To delete the all objects in the bucket. Follow the screenshots attached

![](images/aws_s3_delete_all_versioned_objects.png)

![](images/aws_s3_delete_all_versioned_objects_final.png)

Now that all the objects are deleted we can delete the bucket. To delete the bucket, follow the screenshots attached

![](images/aws_s3_delete_bucket.png)

![](images/aws_s3_delete_bucket_final.png)

## Deleting the Elastic Compute Cloud Instance

To delete the ec2 instance, follow the steps

![](images/aws_ec2_instance_terminate.png)

On successfull execution, the ec2 instance will be terminated and after few hours be removed the console
