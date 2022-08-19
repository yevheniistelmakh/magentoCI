# __How to install Magento on AWS__ 


![Magento + AWS](https://magecomp.com/blog/wp-content/uploads/2020/04/How-to-Install-Magento-on-Amazon-Web-Services-950x500.png)

# Description:<br/>
### This tutorial is designed to help you understand how to install Magenta on ABC. By following this guide, you will easily install and configure the Magenta application step by step.<br/>
# Requirements:<br/>
* Terraform version 1.2.2 (How to install terraform you could look [here](https://www.terraform.io/cli/install/apt))
* Terragrunt version 0.37.1 (How to install terraform you could look [here](https://terragrunt.gruntwork.io/docs/getting-started/install/))
* AWS CLI - installed and configured
* Folder with Magenta installed. How to do it you can see [here](https://docs.google.com/document/d/1h4_H9FUSgaWqv4UpOcUU1aZcsrPPr6sZ_oSS_hQkp50/edit?usp=sharing)
* Dump the database that Magenta creates during installation. How to do this you can see [here](https://docs.google.com/document/d/1h4_H9FUSgaWqv4UpOcUU1aZcsrPPr6sZ_oSS_hQkp50/edit?usp=sharing).
* Account GitHub
* Prepare the Dockerfile and dependencies in advance to create the Nginx image (see examples [here](https://github.com/yevheniistelmakh/magentoCI/tree/master/nginx)).
* Prepare the Dockerfile and dependencies in advance to create the Application image (see examples [here](https://github.com/yevheniistelmakh/magentoCI/tree/master/build_magento_image)).



## Introduction:
1. ## Preparing and configuring the working environment<br/>
   1.1 install terraform and terragrunt 

   1.2 create a working folder and load the terragrunt and terraform code into it 

   1.3 edit common.json and region.json files with appropriate settings/values

   1.4 create infrastructure using terragrunt
2. ## Prepare and create pipelines, create resources (pipelines in Github Action, create images and upload them to a private repository in AWS)<br/> 
   2.1 Preparing files for building the nginx image
  
    
   2.2 Preparing files for building the application image<br/>
  

   2.3 Creating a pipeline for build and uploading images to the AWS ECR<br/>


3. ## Create a pipeline to create a task-definition (use the [task-definition.json](https://github.com/yevheniistelmakh/magentoCI/blob/master/task-definition.json) template)
   3.1 edit task-definition.json file with appropriate settings/values
   
   3.2 create a task definition using pipeline and template task-definition.json 

   3.3 create ECS-service

   3.4 enter the container with the application and configure the database
4. ### Infrastructure cost 
5. ### Literature and resources used
====================================================================

|

|

====================================================================


1) ## Preparing and configuring the working environment
   1.1 install [terraform](https://www.terraform.io/cli/install/apt) and [terragrunt](https://terragrunt.gruntwork.io/docs/getting-started/install/) (Requirement:
Terraform >=1.2.2,
Terragrunt >=0.37.1)

   1.2 create a working folder and load the terragrunt and terraform code into it
   ```
   # mkdir work_folder
   # cd work_folder
   # git clone https://github.com/your_work_profile/terraform-terragrunt-code.git
   ```

   1.3 edit common.json and region.json files with appropriate settings/values<br/>
   
   ```
   {
     "region": "your region",
     "vpc-name-cird-block": "1.1.1.1 //cird-block "
   
   ```
   ```
   {
    "namespace": "starter", // the name for using as a part of name of AWS resources
    "profile": "%aws_profile%", // AWS profile name if a few exists
    "credentials_file": "~/.aws/credentials", // file with AWS credentials
    "iam_role": "%IAM_role%", // if no credentials are provided
    "stage": "sb-0001", // project or environment or account name, should be the same with folder name inside
    "aws_account_id": "%AWS_ACCOUNT_ID%",
    "zone-name": "my-domain.com" // domain name which will be used
    "emails_list": [      // the  list of emails for Amazon SNS
        "email.number1@mail.com",
        "email.number2@mail.com"
   ]
   }
   ```
   1.4 create infrastructure using terragrunt
      1)  Create state for Terraform
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/S3/tf-state
      # terragrunt apply

      ```
      2) Create VPC and related for environment 
      
      ```
      ----- Create VPC -----

      # cd infrastructure/terraform/sb-0001/eu-central-1/VPC/vpc-001
      # terragrunt apply
      ```
      ```
      ----- Create NAT Gateway -----

      # cd infrastructure/terraform/sb-0001/eu-central-1/VPC/nat-0001
      # terragrunt apply
      ```
      ```
      ----- Create VPC -----

      # cd infrastructure/terraform/sb-0001/eu-central-1/VPC/alb-0001
      # terragrunt apply
      ```
      3) Create S3 bucket for application
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/S3/app_bucket
      # terragrunt apply
      ```
      4) Create ECR
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/ECR/app-0001
      # terragrunt apply
      ```
      5) Create Route53
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/route53/private_env 
      # terragrunt apply
      ```
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/route53/public_zone-0002
      # terragrunt apply
      ```
      6) Create SES
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/SES
      # terragrunt apply
      ```
      7) Create ACM
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/ACM/acm-0001
      # terragrunt apply
      ```
      8) Create SSM
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/SSM/param_store/mysql-0001
      # terragrunt apply
      ```
      9)  Create Bastion
       > [!CAUTION]
       > Edit terragrunt.hcl file, update __ec2-public-key-data__  parameter with valid SSH public key data. 
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/EC2/bastion-0001
      # terragrunt apply
      ```
      10)  Create ALB
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/ALB/alb-0001
      # terragrunt apply
      ```
      11)  Create ECS
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/ECS/ecs-0001
      # terragrunt apply
      ```
      12) Create EC 
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/EC/ec-0001
      # terragrunt apply
      ``` 
      13)  Create ES
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/ES/es-0001
      # terragrunt apply
      ```  
      14)   Create RDS
      ```
      # cd infrastructure/terraform/sb-0001/eu-central-1/RDS/mysql-0001
      # terragrunt apply
      ```   
2) ## Prepare and create pipelines, create resources (pipelines in github action, create images and upload them to a private repository in AWS)<br/>
   2.1) Preparing files for building the nginx image<br/>
    Prepare and upload the Dockerfile and all necessary files for nginx to GitHub repository.
   [Example](https://github.com/yevheniistelmakh/magentoCI/tree/master/nginx)
   
   2.2) Preparing files for building the application image<br/>
    ```
   -------- Dockerfile for Magento 2.4.4 with dependencies ---------

   FROM php:8.1-fpm-alpine3.14
   RUN deluser www-data && adduser -u 1005 --disabled-password www-data 
   RUN curl -sSLf \
        -o /usr/local/bin/install-php-extensions \
        https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions && \
   chmod +x /usr/local/bin/install-php-extensions && \
   install-php-extensions gd 
   RUN install-php-extensions bcmath intl pdo_mysql soap sockets  xsl zip
   RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer 
   COPY magento3 /var/www/html
   COPY php-fpm/files/php.ini "$PHP_INI_DIR/php.ini"
   RUN composer install
   RUN chown -R www-data:www-data /var/www/html
   ```
   Prepare and upload the Dockerfile and necessary files to (as well as the installed application itself). Used Magento version 2.4.4<br/>
   [Example files for application image](https://github.com/yevheniistelmakh/magentoCI/tree/master/build_magento_image)
   
   
   [How to install magento 2.4.4 locally](https://docs.google.com/document/d/1h4_H9FUSgaWqv4UpOcUU1aZcsrPPr6sZ_oSS_hQkp50/edit?usp=sharing)


   2.3) Creating a pipeline for uploading images to the AWS ECR <br/>
   Create a pipeline for Nginx. Сreate exactly the same for pushing the image of the application.
   ```
   name: Create and push nginx image to ECR

   on:
     workflow_dispatch:
       inputs:
         AWS_ACCOUNT:
           description: 'AWS ACCOUNT'
           required: true
           type: text
           default: 123412341234
         AWS_REGION:
           description: 'AWS REGION'
           required: true
           type: text
           default: eu-central-1
         ECR_REPO:
           description: 'ECR REPOSITORY'
           required: true
           type: text
           default: nginx
         IMAGE_TAG:
           description: 'IMAGE TAG'
           required: true
           type: text
           default: latest
         fail_scan:
           description: 'Fail build on scan findings?'
           required: true
           type: boolean
           default: true

   env:
     DOCKER_IMAGE: ${{ github.event.inputs.AWS_ACCOUNT }}.dkr.ecr.${{ github.event.inputs.AWS_REGION }}.amazonaws.com/${{ github.event.inputs.ECR_REPO }}:${{ github.event.inputs.IMAGE_TAG }}
   
   jobs:
     deploy:
       name: Build and Deploy
       runs-on: ubuntu-latest
       permissions:
         # required for all workflows
         security-events: write
         # only required for workflows in private repositories
         actions: read
         contents: read

       steps:
       - name: Checkout
         uses: actions/checkout@v2

       - name: Configure AWS credentials
         uses: aws-actions/configure-aws-credentials@v1
         with:
           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
           aws-region: ${{ github.event.inputs.AWS_REGION }}

       - name: Login to Amazon ECR
         uses: aws-actions/amazon-ecr-login@v1
          
       - name: 'Build ${{ github.event.inputs.ECR_REPOSITORY }} image'
         run: 
           docker build -f nginx/Dockerfile -t ${{env.DOCKER_IMAGE}} . 
        
       - name: 'Scan ${{ github.event.inputs.ECR_REPOSITORY }} image'
         id: scan
         uses: anchore/scan-action@v3.2.5
         with:
           image:  '${{ env.DOCKER_IMAGE }}'
           fail-build: '${{ github.event.inputs.fail_scan }}'
           severity-cutoff:  High
           acs-report-enable: true
      
       - name: upload Anchore scan SARIF report
         uses: github/codeql-action/upload-sarif@v2
         with:
           sarif_file: ${{ steps.scan.outputs.sarif }}
        
       - name: 'push ${{ github.event.inputs.ECR_REPOSITORY }} image'
         run: 
           docker push ${{ env.DOCKER_IMAGE }}
     
   ```
   Upload images to ECR. Let's move on to the next step.



3. ## Create a pipeline to create a task-definition (use the [task-definition.json](https://github.com/yevheniistelmakh/magentoCI/blob/master/task-definition.json) template)
   3.1 edit task-definition.json file with appropriate settings/values.<br/>
   Replace and substitute your values in the task-definition.json file. Change container names, region, task name, etc.
   
   3.2 create a task definition using pipeline and template task-definition.json in AWS ECS. <br/>
   ```
   
   name: Create Task Definition
   
   on:
    workflow_dispatch:
      inputs:
        AWS_ACCOUNT:
          description: 'AWS account'
          required: true
          type: text
          default: 123412341234
        AWS_REGION:
          description: 'AWS region'
          required: true
          type: text
          default: eu-west-2
        ECR_REPO:
          description: 'ECR repository'
          required: true
          type: choice
          options:
             - common
             - aplication
        CONTAINER_NAME_NGINX:
           description: 'Container name first images'
           required: true      
           type: choice
           options:
              - sigma_magento-nginx
        IMAGE_TAG_NGINX:
         description: 'Image tag first images'
         required: true
         type: text
         default: nginx-1.0
        CONTAINER_NAME_PHP:
          description: 'Container name second images'
          required: true
          type: choice
          options:
             - sigma_magento-php_fpm
        IMAGE_TAG_PHP:
         description: 'Image tag second images'
         required: true
         type: text
         default:  magento     
        ECS_CLUSTER_NAME:
          description: 'ECS cluster'
          required: true
          type: text
          default:  ecs-prod-0001
        
   env:
     ECS-TASK_DEFENITION-JSON_FILE:      task-definition.json 
      
   permissions:
     contents: read
   
   jobs:
     deploy:
       name: Deploy ubuntu
       runs-on: ubuntu-latest
       environment: production
   
       steps:
       - name: Git clone repo
         uses: actions/checkout@v3
   
       - name: Configure AWS credentials
         uses: aws-actions/configure-aws-credentials@v1
         with:
           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
           aws-region: ${{ github.event.inputs.AWS_REGION }}
           
       - name: Login to Amazon ECR
         uses: aws-actions/amazon-ecr-login@v1
          
       - name: Render Amazon ECS task definition for first container nginx
         uses: aws-actions/amazon-ecs-render-task-definition@v1
         with:
           task-definition: ${{ env.ECS-TASK_DEFENITION-JSON_FILE }}
           container-name: ${{ github.event.inputs.CONTAINER_NAME_NGINX }}
           image: ${{ github.event.inputs.AWS_ACCOUNT }}.dkr.ecr.${{ github.   event.inputs.AWS_REGION }}.amazonaws.com/${{ github.event.inputs.   ECR_REPO }}:${{ github.event.inputs.IMAGE_TAG_NGINX }}
       
       - name: Render Amazon ECS task definition for second container php
         uses: aws-actions/amazon-ecs-render-task-definition@v1
         id: task-def
         with:
           task-definition: ${{ env.ECS-TASK_DEFENITION-JSON_FILE }}
           container-name: ${{ github.event.inputs.CONTAINER_NAME_PHP }}
           image: ${{ github.event.inputs.AWS_ACCOUNT }}.dkr.ecr.${{ github.   event.inputs.AWS_REGION }}.amazonaws.com/${{ github.event.inputs.   ECR_REPO }}:${{ github.event.inputs.IMAGE_TAG_PHP }}
         
       - name: Deploy Amazon ECS task definition
         uses: aws-actions/amazon-ecs-deploy-task-definition@v1
         with:
           task-definition: ${{ steps.task-def.outputs.task-definition }}
           wait-for-service-stability: true
   ```
   [Example pipeline for create task definition](https://github.com/yevheniistelmakh/magentoCI/blob/master/.github/workflows/Create-Task-Definitions-ECS.yml) <br/>
   Run and create a task definition.

   3.3 create ECS-service<br/>
   ```
   # cd infrastructure/terraform/sb-0001/eu-central-1/ECS/ecs-0001
   # terragrunt apply
   ```

   3.4 enter the container with the application and configure the database<br/>
   Connect to the ECS instance. Go in to container with the application. Open the env.php file and change the database parameters. You can also change the name of the admin panel in this file.
   ```
   # docker exec -it application_container sh
   # cd /var/www/html/app/etc
   # apk add nano 
   # nano env.php
   ``` 
   After that, the application should start.


4. ## Infrastructure cost<br/>
[Estimate summary](https://calculator.aws/#/estimate?id=37d4e302edd010985844ee67794231046c498062)

================================================================<br/>
![AWS Cost](https://scontent.fhrk1-1.fna.fbcdn.net/v/t39.30808-6/300233403_5388726444528059_3913294422592281872_n.jpg?_nc_cat=109&ccb=1-7&_nc_sid=0debeb&_nc_ohc=8fafkJ9cm0EAX9DRzGm&_nc_ht=scontent.fhrk1-1.fna&oh=00_AT_bM_cOGfbcZhvEMkRp2uTMY-uCe2Py29qV7LwX7irfAg&oe=6302F833)<br/>

5. ## Literature and resources used<br/>
   * [Source code Magento on GitHub](https://github.com/magento/magento2/tree/2.4-develop)
   * [System requirements for Magento](https://devdocs.magento.com/guides/v2.4/install-gde/system-requirements.html)
   * [Required PHP settings](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/php-settings.html)
   * [](https://docs.google.com/document/d/1h4_H9FUSgaWqv4UpOcUU1aZcsrPPr6sZ_oSS_hQkp50/edit)

   <!-- TOC -->autoauto- [__How to install Magento on AWS__](#__how-to-install-magento-on-aws__)auto- [Description:<br/>](#descriptionbr)auto        - [This tutorial is designed to help you understand how to install Magenta on ABC. By following this guide, you will easily install and configure the Magenta application step by step.<br/>](#this-tutorial-is-designed-to-help-you-understand-how-to-install-magenta-on-abc-by-following-this-guide-you-will-easily-install-and-configure-the-magenta-application-step-by-stepbr)auto- [Requirements:<br/>](#requirementsbr)auto    - [Introduction:](#introduction)autoauto<!-- /TOC -->
