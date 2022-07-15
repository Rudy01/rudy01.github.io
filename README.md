Rutvik's Portfolio website: rudy01.github.io

# C1_SAIC_InternProg
This is the project space for SAIC Interns

**This document will detail the implementation and usage for the intern dashboard project. This project uses Terraform to automatically provision CloudWatch alarms and dashboards for services in two different ways: either at the same time as AWS resources are being provisioned, or after the service’s resources have already been provisioned.**

# Provisioning Templates
**In order to support dashboards for both new and existing infrastructure, we have 2 options for provisioning the dashboard:**

dash_and_infra : Provisions infrastructure and a dashboard simultaneously. Useful for standing up services and continuous monitoring (managing infrastructure and dashboards with the same state).
- Inputs
   - `service_name`: The name of the service
   - `ec2_name_list`:  A list of the EC2 instances associated with the service
   - `server_name_list`:  A list of the web servers associated with the service
   - `server_ami`: The AMI of the server (Describes what OS the server will operate on)
   - `filename_list`: A list of files names (one per lambda function).
     - Add these as .zip files to the project directory
   - `availability_zone`: The availability zone of the service
   - `list_of_emails`: A list of emails that should be notified when an alarm triggers in the service
   - `num_widgets`: The maximum number of widgets that should appear on each row
   - `width`: The width of the widgets
   - `height`: The height of the widgets
   - `aws_region`: The AWS region to provision to
- Outputs
  - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
  - `mode`: Validates that the same template is always run (`dash_and_infra` or `dash_only`)

`dash_only`: Provisions an alarm dashboard by searching for existing infrastructure with a given Service tag. Useful for adding continuous monitoring to existing resources/services (created outside of terraform or managed by a different terraform state).
- Inputs
  - `service_name`: The name of the service
  - `list_of_emails`: A list of emails that should be notified when an alarm triggers in the service
  - `num_widgets`: The maximum number of widgets that should appear on each row
  - `width`: The width of the widgets
  - `height`: The height of the widgets
  - `aws_region`: The AWS region to provision to
- Outputs
  - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
  - `mode`: Validates that the same template is always run (`dash_and_infra` or `dash_only`)
  
  **NOTE: DO NOT SWITCH WHICH TEMPLATE YOU USE FOR A SERVICE ONCE YOU HAVE PROVISIONED ITS DASHBOARD.**

**Terraform relies on a state file to keep track of the resources it manages, so if you switch from** `dash_and_infra` **to** `dash_only` **without removing your infrastructure from the state file, terraform _will_ destroy your infrastructure.**

**Switching from** `dash_only` **to** `dash_and_infra` **is also not advised, since it can result in resource duplication unless you add all of your infrastructure to a .tfvars file and the state first (assuming that infrastructure is supported by our infra modules).**

**We have error handling in place to prevent swaps between templates for active services. If you would like to switch templates, please de-provision all managed resources before trying to run the service with a different template.**

# Usage
**Let’s walk through how to make some example services:**

Service 1 (`dash_and_infra` template, 2 EC2 instances, 1 web server instance, 1 lambda function)
1. Let’s start by making a folder inside of the `examples` directory for our service and switch to it
    ```
    $ mkdir examples/service_1 && cd $_
    --------------------
    c1_saic_internprog/
    ├── examples
    │   ├── service_1
    ```
2. Next, make the .tfvars file for this service
    ```
    $ echo > service_1.tfvars
    --------------------
    c1_saic_internprog/
    ├── examples
    │   ├── service_1
    │   │   ├── hello_python.zip
    │   │   ├── service_1.tfvars
    ```
    :information_source: **Note: If you are creating lambda functions, you also need to add a .zip file of the code**
3. Change the values of service_1 variables by opening the service_1.tfvars file
    ```
    service_name      = "service_1"
    ec2_name_list     = ["Instance_1", "Instance_2"]
    server_name_list  = ["Server_1"]
    filename_list     = ["hello_python.zip"]
    aws_region        = "us-east-1"
    availability_zone = "us-east-1a"
    server_ami        = "ami-052efd3df9dad4825"
    list_of_emails    = ["John.Doe@saic.com", "Jane.Doe@saic.com"]
    num_widgets       = 8
    width             = 4
    height            = 7
    ```
    Each of these variables controls a different aspect of our configuration. To see which variables control what aspect of the config, please view the [inputs documentation](#dash_and_infra) for this template
4. Create a new workspace for this service in the `examples/templates/dash_and_infra` folder
    ```
    terraform workspace new service_1
    ```
    For more about terraform workspaces, please see State: [Workspaces | Terraform by HashiCorp](https://www.terraform.io/language/state/workspaces)
5. Create a Taskfile for the service
    ```
    version: "3"
    
    tasks:
      backend:
        desc: deploy remote backend
        dir: aws_backend
        cmds:
          - terraform init
          - terraform apply -auto-approve
      services:
        desc: deploy all services
        cmds:
          - task backend service_1
      destroy_services:
        desc: destroy all services
          - task destroy_service_1 destroy_backend
      destroy_backend:
        desc: destroying backend
        dir: aws_backend
        cmds:
          - terraform init
          - terraform destroy -auto-approve
      service_1:
          desc: deploy service 1
          dir: examples/templates/dash_and_infra
          cmds:
        	- terraform workspace select service_1
        	- terraform apply -auto-approve -var-file=../../service_1/service_1.tfvars
      destroy_service_1:
        desc: destroying service_1
        dir: examples/templates/dash_and_infra
        cmds: 
          - terraform workspace select service_1
          - terraform destroy -auto-approve -var-file=../../service_1/service_1.tfvars
    ```
    For more info on the Taskfile, go to: [Home | Task](https://taskfile.dev/)

Service 2 (`dash_only` template, 1 EC2 instance, 1 web server instance, 1 lambda function)

1. Let’s start by making a folder inside of the `examples` directory for our service and switch to it
    ```
    $ mkdir examples/service_2 && cd $_
    --------------------
    c1_saic_internprog/
    ├── examples
    │   ├── service_2
    ```
2. Next, make the necessary terraform files for this service
    ```
    $ mkdir examples/service_2 && cd $_
    --------------------
    c1_saic_internprog/
    ├── examples
    │   ├── service_2
    ```
3. Change the values of service_2 variables by opening the service_2.tfvars file
	```
	service_name      = "service_2"
	list_of_emails    = ["John.Doe@saic.com", "Jane.Doe@saic.com"]
	num_widgets       = 8
	width             = 4
	height            = 7
	aws_region        = "us-east-1"
	```
    Each of these variables controls a different aspect of our configuration. To see which variables control what aspect of the config, please view the [inputs documentation](#dash_only) for this template
4. Create a new workspace for this service in the `examples/templates/dash_only` directory
    ```
    $ terraform workspace new service_2
    ```
5. Add the service to the Taskfile
    ```
    version: "3"

    tasks:
        backend:
            desc: deploy remote backend
            dir: aws_backend
            cmds:
                - terraform init
                - terraform apply -auto-approve
        services:
            desc: deploy all services
            cmds:
                - task backend service_1
        destroy_services:
            desc: destroy all services
                - task destroy_service_1 destroy_backend
        destroy_backend:
            desc: destroying backend
            dir: aws_backend
            cmds:
                - terraform init
                - terraform destroy -auto-approve
        service_1:
            desc: deploy service 1
            dir: examples/templates/dash_and_infra
            cmds:
                - terraform workspace select service_1
                - terraform apply -auto-approve -var-file=../../service_1/service_1.tfvars
        destroy_service_1:
            desc: destroying service_1
            dir: examples/templates/dash_and_infra
            cmds: 
                - terraform workspace select service_1
                - terraform destroy -auto-approve -var-file=../../service_1/service_1.tfvars
        service_2:
            desc: deploy service 2
            dir: examples/templates/dash_only
            cmds:
                - aws lambda list-functions > ../../../modules/lambda_data/output.json
                - terraform workspace select service_2
                - terraform apply -auto-approve -var-file=../../service_2/service_2.tfvars
        destroy_service_2:
            desc: destroying service_2
            dir: examples/templates/dash_only
            cmds:
                - terraform workspace select service_2 
                - terraform destroy -auto-approve -var-file=../../service_2/service_2.tfvars
    ```
	For dash_only services, it is a best practice to refresh the list of active lambda functions before provisioning any new dashboards, as there may have been functions added since the last refresh

	Each time you add a new service, you will need to:
	
		a. Add provisioning and deprovisioning tasks for the service (service_2 and destroy_service_2)
		b. Add those tasks to the global provisioning and deprovisioning tasks (services and destroy_services)
	
	For more info on the Taskfile, go to: [Home | Task](https://taskfile.dev/)

From the home directory (C1_SAIC_InternProg) run the Taskfile with the following commands from the command line:
1. Before provisioning any services, provision a remote backend: `$ task backend`
2. To deploy all services: `$ task services`
3. To run an individual service:

	a. Service 1: `$ task service_1`
	
	b. Service 2: `$ task service_2`
4. To destroy all services (and the backend): `$ task destroy_services`
5. To destroy an individual service:

	a. Service 1: `$ task destroy_service_1`
	
	b. Service 2: `$ task destroy_service_2`
6. Backend: `$ task destroy_backend`
7. To test the services: `$ task testing`

:information_source: When creating a test service for your code, make sure you provision a separate backend to prevent your other infrastructure from being influenced by the test

# Modules
**This is a list of modules used by the services and their inputs/outputs**
 - `composite_alarms`: Provision a composite alarm for a service
	 - Inputs
		 - `service_name`: The name of the service you’re making the alarm for
		 - `alarm_text`: A formatted string containing all of the alarms associated with a service Automatically provisioned based on the list_of_alarm_arns output in the service’s remote state file
	 - Outputs
		 - `alarm_arn`: The ARN of the generated composite alarm.
 - `dashboard`: Create a CloudWatch dashboard for a service
	 - Inputs
		 - `service_name`: The name of the service dashboard being generated
		 - `json`: The JSON-formatted template that defines what the dashboard will look like Automatically generated by the json_formatting [module](#json-formatting)
	 - Outputs
		 - None
 - `ec2`: Provision EC2 instances
	 - Inputs
		 - `availability_zone`: The AWS availability zone to provision the instance to
		 - `tags`: The Name (instance name) and Service (service name) tags associated with an instance
	 - Outputs
		 - `instance_id`: The Instance ID of the EC2 instance
		 - `instance_name`: The name of the EC2 instance
 - `ec2_alarms`: Provision alarms for EC2 instances
	 - Inputs
		 - `ec2_instance_id`: The instance ID of the target EC2 instance
		 - `ec2_instance_name`: The instance name of the target EC2 instance
		 - `sns_arn:` The ARN of the appropriate SNS topic based on service
		 - `tags`: The tags associated with the widget alarms
	 - Outputs
		 - `list_of_alarm_arns`: A list of all of the alarm ARNs for the widget
		 - `ec2_instance_name`: The instance name of the target EC2 instance
 - `ec2_data`: Gather the names and ids of the EC2 instances
	 - Inputs
		 - `service_name`: The name of the service the instances are being provisioned for
	 - Outputs
		 - `ids_list`: A list of EC2 instance IDs for that service
		 - `names_list`: A list of EC2 instance IDs for that service
 - `json_formatting`: Format the JSON file for the target dashboard
	 - Inputs
		 - `list_of_widgets`: A list of objects representing each widget that is going in the dashboard. Automatically generated using the output from the ‘*_alarms’ modules.
		 - `num_widgets`: The maximum number of widgets in a row of the dashboard
		 - `width`: The width of a widget on the dashboard
		 - `height`: The height of a widget on the dashboard
	 - Outputs
		 - `widget_json`: A formatted JSON string containing the dashboard body of the target dashboard
 - `lambda`: Provision Lambda functions
	 - Inputs
		 - `tags`: All applicable tags to the lambda instance
		 - `filename`: The path to the .zip file that contains your Lambda function code
		 - `service_name`: The service you are provisioning for
		 - `role_arn`: The role ARN for Lambda IAM
	 - Outputs
		 - `function_name`: The name of the Lambda function you just provisioned
 - `lambda_alarms`: Provision alarms for Lambda function
	 - Inputs
		 - `function_name`: Name of target Lambda function
		 - `sns_arn`: ARN of the appropriate SNS topic based on service name
		 - `tags`: Tags associated with widget alarms
	 - Outputs
		 - `list_of_alarm_arns`: A list of all of the alarm ARNs for the widget
		 - `function_name`: The function name of the target Lambda function
 - `lambda_data`: Gather a list of Lambda functions associated with a service
	 - Inputs
		 - `service_name`: The name of the service you are provisioning alarms for
	 - Outputs
		 - `function_list`: A list of Lambda functions associated with that service
 - `lambda_iam`: Provision service-specific roles for a Lambda function
	 - Inputs
		 - `service_name`: The service to provision for
	 - Outputs
		 - `role_arn`: The service to provision for
 - `networking`: Create network infrastructure for servers
	 - Inputs
		 - `interface_name`: The name of the network interface you're provisioning. Usually the same as the instance name
	 - Outputs
		 - `nic_id`: The ID of the network interface
 - `sns`: Create a service topic in AWS SNS and subscribe everyone in list_of_emails to it
	 - Inputs
		 - `list_of_emails`: A list of people who you want to receive notifications for an SNS topic
		 - `name`: The name of the SNS topic (usually service name)
	 - Outputs
		 - None
 - `server`: Provision web server EC2 instances
	 - Inputs
		 - `nic_id`: The ID of the network interface
		 - `tags`: All applicable tags for the newly provisioned web server
		 - `availability_zone`: The specific availability zone to provision to
		 - `ami`: The AMI of the servers you'd like to provision
	 - Outputs
		 - `instance_id`: The Instance ID of the web server EC2 instance
		 - `instance_name`: The name of the web server EC2 instance
