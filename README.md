# C1_SAIC_InternProg
This is the project space for SAIC Interns

**This document will detail the implementation and usage for the intern dashboard project. This project uses Terraform to automatically provision CloudWatch alarms and dashboards for services in two different ways: either at the same time as AWS resources are being provisioned, or after the service’s resources have already been provisioned.**

# Provisioning Templates
`dash_and_infra`: Provisions infrastructure and a dashboard simultaneously. Useful for standing up services **and** continuous monitoring (managing infrastructure and dashboards with the **same** state).
- Inputs
  - `service_name`: The name of the common service you're provisioning for
  - `aws_region`: The AWS region to provision to
  - `beanstalk`: A list of Elastic Beanstalk definitions
    - `bean_app_name`: The name of the Beanstalk application
    - `bean_env_names`: The list of the names of the Beanstalk environments
    - `solution_stack_name`: The name of the Beanstalk solution stack
    - `subnets`: The list of the subnets and their attributes
      - `name`: The name of the subnet
      - `id`: The ID of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this `name` and `id` should be `null`
  - `ec2`: List of EC2 instance attributes
    - `name`: The name of the EC2 instance
    - `subnet`:  An object that defines a subnet
      - `name`: The name of the subnet
      - `id`: The id of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this `name` and `id` should be `null`
  - `ecs_cluster_names`: A list of cluster definitions for this service
  - `ecs_task_definitions`: A list of task definitions for ECS-EC2
    - `container_definitions`: An object containing definitions for each of the containers involved in a task
      - `name`: The name of a container
      - `image`: The image used to start a container
      - `cpu`: The amount of vCPU to reserve
      - `memory`: The amount of memory to reserve
      - `essential`: Option to decide if the task stops upon container failure
      - `port_mappings`: Allows containers to access ports on the host container instance to send or receive traffic
        - `containerPort`: The port number on the container that's bound to the user-specified or automatically assigned host port
        - `hostPort`: The port number on the container instance to reserve for your container
    - `task_name`: The name of the ECS task being generated
    - `availability_zone`: The availability zone of the service
    - `services`: The name of the ECS services to provision for a given task definition
      - `name`: The name of ECS service
      - `desired_count`: The number services you need for ECS
    - `cluster_name`: The name of the cluster to which this ECS task description is being assigned
  - `eks_clusters`: A list of the EKS cluster definitions for this service
    - `name`: The name of the EKS cluster
    - `subnets`: A list of subnets and their attributes
      - `name`: The name of the subnet
      - `id`: The ID of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
    - `user_names`: The list of the user names
  - `fargate_profiles`: A list of EKS Fargate profile definitions
    - `name`: The name of the Fargate profile
    - `subnets`: A list of subnets and their attributes
      - `name`: The name of the subnet
      - `id`: The ID of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
    - `cluster_name`: The name of the cluster to which this EKS task description is being assigned
  - `fargate_task_definitions`: A list of task definitions for ECS-Fargate
    - `containter_definitions`: An object containing definitions for each of the containers involved in a task
      - `name`: The name of a container
      - `image`: The image used to start a container
      - `cpu`: The amount of vCPU to reserve
      - `memory`: The amount of memory to reserve
      - `essential`: Option to decide if the task stops upon container failure
      - `port_mappings`: Allows containers to access ports on the host container instance to send or receive traffic
        - `containerPort`: The port number on the container that's bound to the user-specified or automatically assigned host port
        - `hostPort`: The port number on the container instance to reserve for your container
    - `task_name`: The name of the Fargate task being generated
    - `services`: The name of the Fargate services to provision for a given task definition
      - `name`: The name of Fargate service
      - `desired_count`: The number services you need for Fargate
      - `subnets`: A list of subnets and their attributes
        - `name`: The name of the subnet
        - `id`: The ID of the subnet
        - `type`: Indicates who created the subnet
          - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
          - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
          - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
    - `cluster_name`: The name of the cluster to which this Fargate task description is being assigned
  - `height`: The height of each widgets
  - `lambdas`: List of all lambda functions to use/create. 
    - `file`: The file name of the Lambda function (add .zip file to the service directory)
    - `security_group_indices`: Indices of the security_groups variable corresponding to the security groups associated with the lambda function.
    - `subnets`: The subnets this lambda function is in
      - `name`: The name of the subnet
      - `id`: The id of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
  - `list_of_emails`: A list of emails that you would like to receive updates for the dashboard alarms
    - **Must include only SAIC emails**
  - `nat_gatways`: A list of all NAT gateways and their attributes
    - `name`: The name of the NAT gateway
    - `connectivity_type`: Whether the NAT gateway is public (connects to Internet) or private (connects to other VPCs)
    - `subnets`: The subnets this route table is in
      - `name`: The name of the subnet
      - `id`: The id of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
    - `private_ip`: The private IP address for the NAT gateway
  - `network_acls`: A list of all network ACLs and their attributes
    - `network_acl_name`: The name of the network ACL
    - `rules`: Describes the rules for the NACL
      - `from_port`: The source port of traffic
      - `to_port`: The destination port of traffic
      - `protocol`: The protocol (TCP/UDP) used
      - `rule_action`: Whether to allow or deny traffic
      - `rule_number`: The numerical order of the rule. Rules are executed in numerical order
      - `egress`: Does this rule applies to outgoing traffic too, or just incoming traffic (true/false)?
    - `subnets`: The subnets this route table is in
      - `name`: The name of the subnet
      - `id`: The id of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
    - `cider_block`: The CIDR block associated with the Network ACL
  - `node_groups`: A list of EKS node group definitions
    - `name`: The name of the node group
    - `cluster_name`: The name of the cluster
    - `subnets`: A list of subnets and their attributes
      - `name`: The name of the subnet
      - `id`: The ID of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the name should be null and you should provide the subnet id
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
    - `instance_types`: A list of the EC2 instance types associated with the node group
    - `desired_size`: Desired number of worker nodes
    - `max_size`: Maximum number of worker nodes
    - `min_size`: Minimum number of worker nodes
    - `max_unavailable`: Desired max number of unavailable worker nodes during a node group update
  - `num_widgets`: The number of widgets in a dashboard row
  - `route_tables`: A list of route tables and their routes
    - `name`: The name of the route table being provisioned
    - `routes`: Provision routes for a route table within a VPC
      - `destination_cidr_block`: The IPv4 CIDR range specified in a route in the table
      - `gateway_name`: The name of a gateway
      - `gateway_type`: The type of gateway
    - `subnets`: The subnets this route table is in
      - `name`: The name of the subnet
      - `id`: The id of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this both `name` and `id` should be `null`
  - `security_groups`: A list of all security groups and their attributes
    - `security_group_name`: The name of the security group
    - `description`: A description of what the security group does
    - `rules`: Describes the rules for the security group
      - `from_port`: The source port of traffic
      - `to_port`: The destination port of traffic
      - `protocol`: The protocol (TCP/UDP) used
      - `type`: Whether traffic is coming in (`ingress`) or going out (`egress`)
    - `vpc`: The VPC for the subnet
      - `name`: The name of the VPC
      - `id`: The id of the VPC
      - `type`: Indicates who created the subnet
        - `user`: A user created the VPC using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the VPC using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for VPC that AWS created. If you want this both `name` and `id` should be `null`
    - `cidr_blocks`: The CIDR blocks associated with the security group
  - `servers`: List of EC2 server instance attributes
    - `name`: The name of the server instance
    - `security_group_index`: The index of security groups that will apply to this server. Taken from the indices of the `security_groups` object defined below
    - `subnet`: An object that defines a subnet
      - `name`: The name of the subnet
      - `id`: The id of the subnet
      - `type`: Indicates who created the subnet
        - `user`: A user created the subnet using ClickOps. If you want this the `id` should be `null` and **you should provide the subnet name**
        - `created`: We make the subnet using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default for subnet that AWS created. If you want this `name` and `id` should be `null`
    - `private_ips`: The IP address (or addresses) associated with the server.
    - `server_ami`: The AMI of the server (Describes what OS the server will operate on)
    - `container_instance`: If `true` it allows you to create an ECS container instance for the subnet
    - `cluster_name`: The name of the cluster to which this ECS instance is being assigned
  - `subnets`: An object containing the subnet names and route tables associated with those subnets
    - `cidr_block`: The CIDR block associated with the subnet
    - `tags`: A map of tags for the subnet
    - `availability_zone`: The availability zone where the subnet is located
    - `subnet_type`: The type of the subnet (user/created/default)
    - `vpc`: The VPC for the subnet
      - `name`: The name of the VPC
      - `id`: The id of the VPC
      - `type`: Indicates who created the VPC
        - `user`: A user created the VPC using ClickOps. If you want this the `id` should be null and **you should provide the subnet name**
        - `created`: We make the VPC using Terraform. If you want this the `name` should be `null` and **you should provide the subnet id**
        - `default`: The default VPC that AWS created. If you want this both `name` and `id` should be `null`
  - `user_eks_clusters`: A list of user-created EKS clusters that you would like to update permissions for and add monitoring to
    - cluster name: The name of the cluster
    - user_names: The list of the user names
  - `vpc`: A list of custom VPCs and their CIDR blocks
    - `name`: The name of the custom VPC
    - `cidr_block`: The CIDR block for the custom VPC
  - `width`: The width of each widgets
- Outputs
  - `aws_region`: The AWS region to provision to
  - `eks_clusters`: Outputs a map of EKS clusters and the users to be given admin access to them
  - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
  - `mode`: Validates that the same template is always run (dash_and_infra or dash_only)
  - `public_ip`: Outputs the public IP address of the web server so it can be http-checked in our tests

`dash_only`: Provisions an alarm dashboard by searching for existing infrastructure with a given Service tag. Useful for adding continuous monitoring to existing resources/services (created **outside** of terraform or managed by a **different** terraform state).
- Inputs
  - `service_name`: The name of the common service you're provisioning for
  - `aws_region`: The AWS region to provision to
  - `height`: The height of each widgets
  - `list_of_emails`: A list of emails that you would like to receive updates for the dashboard alarms
    - **Must include only SAIC emails**
  - `num_widgets`: The number of widgets in a dashboard row
  - `width`: The width of each widgets
- Outputs
  - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
  - `mode`: Validates that the same template is always run (`dash_and_infra` or `dash_only`)

:warning: **NOTE: DO NOT SWITCH WHICH TEMPLATE YOU USE FOR A SERVICE ONCE YOU HAVE PROVISIONED ITS DASHBOARD.**

**Terraform relies on a state file to keep track of the resources it manages, so if you switch from `dash_and_infra` to `dash_only` without removing your infrastructure from the state file, terraform will destroy your infrastructure.**

**Switching from `dash_only` to `dash_and_infra` is also not advised, since it can result in resource duplication unless you add all of your infrastructure to a .tfvars file and the state first (assuming that infrastructure is supported by our infra modules).**

**We have error handling in place to prevent swaps between templates for active services. If you would like to switch templates, please de-provision all managed resources before trying to run the service with a different template.**

# Usage
**Let’s walk through how to make an example service that uses each of our modules with a `dash_and_infra` implementation or a `dash_only` (and `infra_only` for testing purposes) implementation. Both of these examples can be found in the `examples` directory of the repo:**

**Note on shell scripts (`kube_config.sh` and `aws.sh`):**
Unfortunately, Terraform does not have support for all AWS API calls. If/when you encounter services (like EKS (kube_config.sh) or listing all available for certain services (aws.sh), there is usually an AWS CLI or kubectl command that will perform the same functionality. **Try to do as much as possible through terraform**, but know that you can do things through shell scripts if you need to.

`aws.sh` should be run **before** provisioning dash_only services. kube_config.sh should be run after provisioning dash_and_infra or infra_only services that require Kubernetes permissions/monitoring configuration.

:information_source: kube_config.sh does install the [CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-EKS.html) on your Kubernetes cluster

**Simultaneous (`dash_and_infra` template, 2 EC2 instances, 3 server EC2 instances (2 used in containers), 2 lambda functions, 1 VPC, 4 subnets, 4 route tables, 2 NAT gateways, 1 security group, 1 ECS-EC2 task, 1 ECS-Fargate task, 1 ECS cluster, 1 EKS cluster, 1 EKS-Fargate profile, 1 EKS-EC2 node group, and 1 Elastic Beanstalk app (with 1 environment))**
1. Let’s start by making a folder inside of the examples directory for our service and switch to it
   ```
   $ mkdir examples/simultaneous && cd $_
   --------------------
   c1_saic_internprog/
   ├── examples
   │   ├── simultaneous
   ```
2. Next, make the .tfvars file for this example
   ```
   $ echo > dash.tfvars
   --------------------
   c1_saic_internprog/
   ├── examples
   │   ├── simultaneous
   │   │   ├── hello_python.zip
   │   │   ├── hello_python_2.zip
   │   │   ├── simultaneous.tfvars
   ```
   :information_source: **Note: If you are creating lambda functions, you also need to add a .zip file of the code**
3. Change the values of simultaneous variables by opening the simultaneous.tfvars file
   ```
   service_name = "simultaneous"
   aws_region   = "us-east-1"
   ec2 = [{
     name = "Jeffrey"
     subnet = {
       name = "subnet_1"
       id   = null
       type = "created"
     }
     },
     {
       name = "Greg"
       subnet = {
         name = "subnet_2"
         id   = null
         type = "created"
       }
   }]

   servers = [{
     name                 = "Steve"
     security_group_index = 0
     subnet = {
       name = "subnet_1"
       id   = null
       type = "created"
     }
     private_ips        = ["10.0.1.60"]
     server_ami         = "ami-052efd3df9dad4825"
     container_instance = false
     cluster_name       = null
     },
     {
       name                 = "ECS_instance_1"
       security_group_index = 0
       subnet = {
         name = "subnet_1"
         id   = null
         type = "created"
       }
       private_ips        = ["10.0.1.70"]
       server_ami         = "ami-061c737b1691cb15f"
       container_instance = true
       cluster_name       = "ecs_cluster_1"
     },
     {
       name                 = "ECS_instance_2"
       security_group_index = 0
       subnet = {
         name = "subnet_2"
         id   = null
         type = "created"
       }
       private_ips        = ["10.0.10.60"]
       server_ami         = "ami-061c737b1691cb15f"
       container_instance = true
       cluster_name       = "ecs_cluster_1"
   }]

   lambdas = [{
     file                   = "hello_python.zip"
     security_group_indices = [0]
     subnets = [{
       name = "subnet_1"
       id   = null
       type = "created"
     }]
     },
     {
       file                   = "hello_python_2.zip"
       security_group_indices = [0]
       subnets = [{
         name = "subnet_2"
         id   = null
         type = "created"
       }]
   }]

   list_of_emails = ["John.Doe@saic.com", "Jane.Doe@saic.com"]
   num_widgets    = 6
   width          = 4
   height         = 7

   vpc = [{
     name       = "simultaneous"
     cidr_block = "10.0.0.0/16"
   }]

   route_tables = [{
     name = "public_internet_us_east_1a"
     routes = [{
       destination_cidr_block = "0.0.0.0/0"
       gateway_name           = "simultaneous"
       gateway_type           = "internet"
     }]
     subnets = [{
       name = "subnet_1"
       id   = null
       type = "created"
     }]
     },
     {
       name = "public_internet_us_east_1b"
       routes = [{
         destination_cidr_block = "0.0.0.0/0"
         gateway_name           = "simultaneous"
         gateway_type           = "internet"
       }]
       subnets = [{
         name = "subnet_2"
         id   = null
         type = "created"
       }]
     },
     { name = "private_public_us_east_1a"
       routes = [{
         destination_cidr_block = "0.0.0.0/0"
         gateway_name           = "ngw_us_east_1a"
         gateway_type           = "nat"
       }]
       subnets = [{
         name = "subnet_3"
         id   = null
         type = "created"
       }]
     },
     {
       name = "private_public_us_east_1b"
       routes = [{
         destination_cidr_block = "0.0.0.0/0"
         gateway_name           = "ngw_us_east_1b"
         gateway_type           = "nat"
       }]
       subnets = [{
         name = "subnet_4"
         id   = null
         type = "created"
       }]
   }]

   nat_gateways = [{
     name              = "ngw_us_east_1a"
     connectivity_type = "public"
     subnet = {
       name = "subnet_1"
       id   = null
       type = "created"
     }
     private_ip = "10.0.1.1"
     },
     {
       name              = "ngw_us_east_1b"
       connectivity_type = "public"
       subnet = {
         name = "subnet_2"
         id   = null
         type = "created"
       }
       private_ip = "10.0.2.1"
   }]

   subnets = [{
     cidr_block        = "10.0.1.0/24"
     route_table_index = 0
     tags = {
       Name                                  = "subnet_1"
       "kubernetes.io/cluster/eks_cluster_1" = "shared"
     }
     subnet_type       = "public"
     availability_zone = "us-east-1a"
     vpc = {
       name = "simultaneous"
       id   = null
       type = "created"
     }
     },
     {
       cidr_block        = "10.0.10.0/24"
       route_table_index = 1
       tags = {
         Name                                  = "subnet_2"
         "kubernetes.io/cluster/eks_cluster_1" = "shared"
       }
       availability_zone = "us-east-1b"
       subnet_type       = "public"
       vpc = {
         name = "simultaneous"
         id   = null
         type = "created"
       }
     },
     {
       cidr_block        = "10.0.3.0/24"
       route_table_index = 2
       tags = {
         Name                                  = "subnet_3"
         "kubernetes.io/cluster/eks_cluster_1" = "shared"
       }
       availability_zone = "us-east-1a"
       subnet_type       = "private"
       vpc = {
         name = "simultaneous"
         id   = null
         type = "created"
       }
     },
     {
       cidr_block        = "10.0.5.0/24"
       route_table_index = 3
       tags = {
         Name                                  = "subnet_4"
         "kubernetes.io/cluster/eks_cluster_1" = "shared"
       }
       availability_zone = "us-east-1b"
       subnet_type       = "private"
       vpc = {
         name = "simultaneous"
         id   = null
         type = "created"
       }
     }
   ]

   security_groups = [{
     security_group_name = "internet"
     description         = "Allows HTTP and HTTPS traffic"
     rules = [{
       from_port = 443
       to_port   = 443
       protocol  = "tcp"
       type      = "ingress"
       },
       {
         from_port = 80
         to_port   = 80
         protocol  = "tcp"
         type      = "ingress"
       },
       {
         from_port = 0
         to_port   = 0
         protocol  = "-1"
         type      = "egress"
     }]
     vpc = {
       name = "simultaneous"
       id   = null
       type = "created"
     }

     cidr_blocks = ["0.0.0.0/0"]
     }
   ]

   ecs_task_definitions = [{
     container_definitions = [{
       name      = "ecs_container_1"
       image     = "saicoss/anvil:latest"
       cpu       = 10
       memory    = 512
       essential = true
       port_mappings = [{
         containerPort = 80
         hostPort      = 80
       }]
     }]

     task_name         = "ecs_task_1"
     availability_zone = "us-east-1a"
     services = [{
       name          = "do_this_thing_ecs"
       desired_count = 2
     }]
     cluster_name = "ecs_cluster_1"
   }]

   fargate_task_definitions = [{
     container_definitions = [{
       name      = "fargate_container_1"
       image     = "saicoss/anvil:latest"
       cpu       = 10
       memory    = 512
       essential = true
       port_mappings = [{
         containerPort = 80
         hostPort      = 80
       }]
     }]

     task_name = "fargate_task_1"
     services = [{
       name          = "do_this_thing_fargate"
       desired_count = 2
       subnet = {
         name = "subnet_1"
         id   = null
         type = "created"
       }
     }]
     cluster_name = "ecs_cluster_1"
   }]

   ecs_cluster_names = ["ecs_cluster_1"]

   eks_clusters = [{
     name = "eks_cluster_1"
     subnets = [{
       name = "subnet_1"
       id   = null
       type = "created"
       },
       {
         name = "subnet_2"
         id   = null
         type = "created"
     }]

     user_names = ["John.Doe@saic.com", "Jane.Doe@saic.com"]
   }]

   fargate_profiles = [{
     name = "fargate_profile_1",
     subnets = [{
       name = "subnet_3"
       id   = null
       type = "created"
       },
       {
         name = "subnet_4"
         id   = null
         type = "created"
     }]
     cluster_name = "eks_cluster_1"
   }]

   node_groups = [{
     name         = "node_group_1"
     cluster_name = "eks_cluster_1"
     subnets = [{
       name = "subnet_1"
       id   = null
       type = "created"
       },
       {
         name = "subnet_2"
         id   = null
         type = "created"
     }]
     instance_types  = ["t3.medium"]
     desired_size    = 1
     max_size        = 1
     min_size        = 1
     max_unavailable = 1
   }]

   beanstalk = [{
     bean_app_name       = "Sparky"
     bean_env_names      = ["Dog-House"]
     solution_stack_name = "64bit Amazon Linux 2 v3.5.3 running Go 1" // "Solution Stack"
     subnets = [{
       name = "subnet_1"
       id   = null
       type = "created"
       },
       {
         name = "subnet_2"
         id   = null
         type = "created"
     }]
   }]
   ```
   Each of these variables controls a different aspect of our configuration. To see which variables control what aspect of the config, please view the [inputs documentation](#provisioning-templates) for this template
4. Create a new workspace for this service in the `examples/templates/dash_and_infra folder`
   ```
   $ terraform workspace new simultaneous
   ```
   For more about terraform workspaces, please see [Workspaces | Terraform by HashiCorp](https://www.terraform.io/language/state/workspaces)
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
          - task backend simultaneous
      destroy_services:
        desc: destroy all services
          - task destroy_simultaneous destroy_backend
      destroy_backend:
        desc: destroying backend
        dir: aws_backend
        cmds:
          - terraform init
          - terraform destroy -auto-approve
      simultaneous:
        desc: provision the simultaneous test example
        dir: examples/templates/dash_and_infra
        cmds:
          - terraform init -reconfigure
          - if (terraform workspace list | grep 'simultaneous'); then terraform workspace select simultaneous; else terraform workspace new simultaneous; fi
          - terraform apply -auto-approve -var-file-../../simultaneous/simultaneous.tfvars
          - sh kube_config.sh
      destroy_simultaneous:
        desc: de-provision the simultaneous test example
        dir: examples/templates/dash_and_infra
        cmds:
          - terraform init -reconfigure
          - if (terraform workspace list | grep 'simultaneous'); then terraform workspace select simultaneous; else terraform workspace new simultaneous; fi
          - terraform destroy -auto-approve -var-file-../../simultaneous/simultaneous.tfvars   
    ```
   For more info on the Taskfile, go to: [Home | Task](https://taskfile.dev/)

**Separate (`dash_only` template, 2 EC2 instances, 3 server EC2 instances (2 used in containers), 2 lambda functions, 1 VPC, 4 subnets, 4 route tables, 2 NAT gateways, 1 security group, 1 ECS-EC2 task, 1 ECS-Fargate task, 1 ECS cluster, 1 EKS cluster, 1 EKS-Fargate profile, 1 EKS-EC2 node group, and 1 Elastic Beanstalk app (with 1 environment). (`infra_only` files not shown, please go to our repo to see the .tfvars and Task commands associated with the underlying infrastructure)**
1. Let’s start by making a folder inside of the `examples` directory for our service and switch to it
   ```
   $ mkdir examples/separate && cd $_
   --------------------
   c1_saic_internprog/
   ├── examples
   │   ├── separate
   ```
2. Next, make the necessary terraform files for this service
   ```
   $ echo > dash.tfvars
   --------------------
   c1_saic_internprog/
   ├── examples
   │   ├── separate
   │   │   ├── dash.tfvars
   ```
3. Change the values of separate variables by opening the separate.tfvars file
   ```
   service_name      = "separate"
   list_of_emails    = ["John.Doe@saic.com", "Jane.Doe@saic.com"]
   num_widgets       = 6
   width             = 4
   height            = 7
   aws_region        = "us-east-1"
   ```
   Each of these variables controls a different aspect of our configuration. To see which variables control what aspect of the config, please view the [inputs documentation](#provisioning-templates) for this template
4. Create a new workspace for this service in the `examples/templates/dash_only` directory
   ```
   $ terraform workspace new separate
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
          - task backend simultaneous separate
      destroy_services:
        desc: destroy all services
          - task destroy_simultaneous destroy_backend destroy_separate
      destroy_backend:
        desc: destroying backend
        dir: aws_backend
        cmds:
          - terraform init
          - terraform destroy -auto-approve
      simultaneous:
        desc: provision the simultaneous test example
        dir: examples/templates/dash_and_infra
        cmds:
          - terraform init -reconfigure
          - if (terraform workspace list | grep 'simultaneous'); then terraform workspace select simultaneous; else terraform workspace new simultaneous; fi
          - terraform apply -auto-approve -var-file-../../simultaneous/simultaneous.tfvars
          - sh kube_config.sh
      destroy_simultaneous:
        desc: de-provision the simultaneous test example
        dir: examples/templates/dash_and_infra
        cmds:
          - terraform init -reconfigure
          - if (terraform workspace list | grep 'simultaneous'); then terraform workspace select simultaneous; else terraform workspace new simultaneous; fi
          - terraform destroy -auto-approve -var-file-../../simultaneous/simultaneous.tfvars
      separate:
        desc: deploy separate
        dir: examples/templates/dash_only
        cmds:
          - sh aws.sh
          - terraform init -reconfigure
          - if (terraform workspace list | grep 'separate'); then terraform workspace select separate; else terraform workspace new separate; fi
          - terraform apply -auto-approve -var-file-../../separate/dash.tfvars
      destroy_separate:
        desc: destroying separate
        dir: examples/templates/dash_only
        cmds:
          - terraform init -reconfigure
          - if (terraform workspace list | grep 'separate'); then terraform workspace select separate; else terraform workspace new separate; fi
          - terraform destroy -auto-approve -var-file=../../separate/dash.tfvars  
    ```
  a. Each time you add a new service, you will need to:
    
     i. Add provisioning and deprovisioning tasks for the service (separate and destroy_separate)
    ii. Add those tasks to the global provisioning and deprovisioning tasks (services and destroy_services)
    
  b. For more info on the Taskfile, go to: [Home | Task](https://taskfile.dev/)
  
**From the home directory (C1_SAIC_InternProg) run the Taskfile with the following commands from the command line:**
1. Before provisioning any services, provision a remote backend: `$ task backend`
2. To deploy all services: `$ task services`
3. To run an individual service:

    a. **Simultaneous**: `$ task simultaneous`
  
    b. **Separate**: `$ task separate`
  
4. To destroy all services (and the backend): `$ task destroy_services`
5. To destroy an individual service:

    a. **Simultaneous: $ task destroy_simultaneous**
  
    b. **Separate: $ task destroy_separate**
  
    c. **Backend: $ task destroy_backend**
  
6. To test the services: `$ task test`

:information_source: When creating a test service for your code, make sure you provision a separate backend to prevent your other infrastructure from being influenced by the test. The terra_test.go file controls the testing example. Learn more about the Terratest package [here](https://terratest.gruntwork.io/) and [here](https://www.youtube.com/watch?v=xhHOW0EF5u8)

**We have provided testing examples that reflect our current implementations of each deployment. As services are added and changed, the .tfvars files for each of these implementations will need to be updated accordingly.**

**Testing**

All of our tests are written in Go using the [Terratest](https://terratest.gruntwork.io/) package. Far more extensive documentation on everything that Terratest can do is available [here](https://terratest.gruntwork.io/docs/), but here’s what you need to know to get started:
```
import (
	"fmt"
	"testing"
	"time"

	http_helper "github.com/gruntwork-io/terratest/modules/http-helper"

	"github.com/gruntwork-io/terratest/modules/logger"

	"github.com/gruntwork-io/terratest/modules/terraform"

	"github.com/gruntwork-io/terratest/modules/shell"
)

func TestDashAndInfra(t *testing.T) {
	// retryable errors in terraform testing.
	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../examples/templates/dash_and_infra", VarFiles: []string{"../../simultaneous/simultaneous.tfvars"},
	})

	defer terraform.Destroy(t, terraformOptions)

	terraform.Init(t, terraformOptions)
	terraform.WorkspaceSelectOrNew(t, terraformOptions, "simultaneous")
	terraform.Apply(t, terraformOptions)

	shell.RunCommandAndGetOutput(t, shell.Command{Command: "sh", Args: []string{"kube_config.sh"}, WorkingDir: "../examples/templates/dash_and_infra", Env: map[string]string{"": ""}, Logger: logger.New(nil)})

}
```
`terraform.WithDefaultRetryableErrors()` allows you to specify the `TerraformDir` you’d like to operate out of and any VarFiles (.tfvars) that you’d like to use

`defer terraform.Destroy()` makes sure that provisioned infrastructure is destroyed once the test ends

`terraform.Init()`, `terraform.WorkspaceSelectOrNew()`, and `terraform.Apply()` work the same way that the corresponding terraform CLI command does

`shell.RunCommandAndGetOutput()` runs a bash shell command and gets the output as a string

We recommend following [test-driven development](https://www.datree.io/resources/automated-testing-tools-for-infrastructure-as-code) best-practices and writing your Terratest tests first, then writing the module code that will succeed when run against those test examples.

# Modules
**This is a list of modules used by the services and their inputs/outputs**

Template for below documentation:
- `Name`: Description
  - Inputs
    - `input name`: Input description
  - Outputs
    - output name: Output description

#
- `beanstalk`: Provision an elastic beanstalk app
  - Inputs
    - `bean_app_name`: The name of the Elastic Beanstalk Application
    - `bean_env_name`: The list of the Elastic Beanstalk Environment names
    - `instance_profile_name`: Instance profile name for Elastic Beanstalk
    - `solution_stack_name`: The name of the solution stack being used
    - `subnet_ids`: The list of IDs for the subnets
  - Outputs
    - None
- `beanstalk_alarms`: Provision alarms for Elastic Beanstalk
  - Input
    - `bean_app_name`: Name of the Elastic Beanstalk Application
    - `bean_env_names`: List of the Elastic Beanstalk Environment names
    - `sns_arn:` ARN of the appropriate SNS topic based on service
  - Output
    - `bean_app_name`: The name of the Elastic Beanstalk Application
    - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
- `beanstalk_data`: Gather a map of all Elastic Beanstalk applications
  - Input
    - `service_name`: The name of the common service you're provisioning for
  - Output
    - `bean_app_map`: The local map of the Elastic Beanstalk Applications
- `beanstalk_iam`: IAM role for Elastic Beanstalk
  - Input
    - `service_name`: The name of the common service you're provisioning for
  - Output
    - `instance_profile_name`: The name of the instance profile for Beanstalk EC2 instances
    - `role_arn`: The ARN of the role associated with the Beanstalk service
- `composite_alarms`: Provision a composite alarm for a service
  - Inputs
    - `alarm_text`: A formatted list containing all of the alarm names associated with a service. Automatically provisioned based on the list_of_alarm_arns output in the service’s remote state
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `alarm_arn`: The ARN of the generated composite alarm.
- `dashboard`: Create a CloudWatch dashboard for a service
  - Inputs
    - `json`: JSON formatting template file that defines what the dashboard will look like. Automatically generated by the json_formatting module
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - None
- `ec2`: Provision EC2 instances
  - Inputs
    - `service_name`: The workspace you should be provisioning into (same as service_name)
    - `subnet_id`: ID of the VPC subnet to provision in
    - `tags`: All applicable tags for an EC2 instance
  - Outputs
    - `instance_id`: The instance ID of the EC2 instance
    - `instance_name`: The name of the EC2 instance
- `ec2_alarms`: Provision alarms for EC2 instances
  - Inputs
    - `ec2_instance_id`: The instance ID of the target EC2 instance
    - `ec2_instance_name`: The instance name of the target EC2 instance
    - sns_arn: ARN of the appropriate SNS topic based on service
  - Outputs
    - `ec2_instance_name`: The instance name of the target EC2 instance
    - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
- `ec2_data`: Gather the names and ids of the EC2 instances
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `ids_list`: A list of EC2 instance IDs for that service
    - `names_list`: A list of EC2 instance names for that service
- `ecs`: Provisions an ECS task with the EC2 launch type
  - Inputs
    - `availability_zone`: The specific AWS availability zone to provision the ECS Task
    - `cluster_name`: The name of the cluster to which this ECS task description is being assigned
    - `container_definitions`: An object containing definitions for each of the containers involved in a task
      - `name`: The name of a container
      - `image`: The image used to start a container
      - `essential`: Option to decide if the task stops upon container failure
      - `port_mappings`: Allows containers to access ports on the host container instance to send or receive traffic
        - `containerPort`: The port number on the container that's bound to the user-specified or automatically assigned host port
        - `hostPort`: The port number on the container instance to reserve for your container
    - `services`: The name of the ECS services to provision for a given task definition
      - `name`: The name of ECS service
      - `desired_count`: The number services you need for ECS
    - `task_name`: The name of the ECS task being generated
  - Outputs
    - None
- `ecs_alarms`: Provision alarms for ECS instances
  - Inputs
    - `cluster_name`: Cluster name of target ECS Cluster
    - `sns_arn`: ARN of the appropriate SNS topic based on service
  - Outputs
    - `cluster_name`: The name of the ECS cluster
    - `list_of_alarm_arns`: A list of all alarms provisioned for the service. Used to create an overview dashboard
- `ecs_cluster`: Provisions an ECS cluster for a given service
    - Inputs
      - `cluster_name`: The name of the cluster to which this ECS task description is being assigned
    - Outputs
      - `cluster_arn`: The ARN of the ECS cluster
- `ecs_data`: Read data about all active ECS clusters with a given service pet name
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `ecs_names_list`: A list of the names of all active ECS clusters with a given service pet name
- `ecs_iam`: IAM for ECS clusters
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `instance_profile_name`: The name of the instance profile for ECS-EC2 instances
    - `role_arn`: The ARN of the role associated with ECS
- `eks_alarms`: Provision alarms for EKS instances
  - Input
    - `cluster_name`: The name of the EKS cluster
    - `sns_arn`: ARN of the appropriate SNS topic based on service
  - Output
    - `cluster_name`: The name of the EKS cluster
    - `list_of_alarm_arns`: A list of all alarms provisioned for this module. Used to create a service overview dashboard
- `eks_cluster`: Provisions an EKS cluster for a given service
  - Input:
    - `cluster_name`: The name of the EKS cluster
    - `role_arn`: The ARN of the role provisioned for EKS clusters
    - `subnet_ids`: A list of subnet IDs to provision to
    - `user_names`: The name of the common service you're provisioning for
  - Output:
    - `cluster_name`: The cluster name of the target EKS name
    - `endpoint`: The endpoint for the EKS cluster
    - `kubeconfig-certificate-authority-data`: The Kubernetes certificate of authority for data
    - `user_names`: The list of user ARNs that need access to the EKS cluster
- `eks_cluster_iam`: Creates an EKS cluster IAM role for a given service
  - Input:
    - `service_name`: The name of the common service you're provisioning for
  - Output
    - `role_arn`: The ARN of the role associated with EKS Cluster IAM
- `eks_data`: Gets a list of EKS clusters for a given service
  - Input:
    - `service_name`: The name of the common service you're provisioning for
  - Output
    - `names_list`: A list of EKS Clusters for that service
- `eks_fargate_profile`: Creates and assigns a Fargate profile for a given cluster
  - Input
    - `cluster_name`: The name of the EKS cluster
    - `fargate_profile_name`: The name of the Fargate profile
    - `namespace`: The namespace of the Fargate profile
    - `role_arn`: The ARN of the pod execution
    - `subnet_ids`: The list of IDs for the subnets
  - Output
    - None
- `eks_fargate_profile_iam`: Creates a Fargate profile IAM role for a given service
  - Input
    - `service_name`: The name of the common service you're provisioning for
  - Output
    - `role_arn`: The ARN of the role associated with EKS Fargate Profile IAM
- `eks_node_group`: Provisions a new EKS node group
  - Input
    - `cluster_name`: The name of the cluster to which this EKS Node Group task description is being assigned
    - `desired_size`: Desired number of worker nodes
    - `instance_types`: A list of the EC2 instance types associated with the node group
    - `max_size`: Maximum number of worker nodes
    - `max_unavailable`: Desired max number of unavailable worker nodes during a node group update
    - `min_size`: Minimum number of worker nodes
    - `node_group_name`: The name of the node group
    - `node_role_arn`: The ARN of the IAM role for the node group
    - `subnet_ids`: The list of IDs for the subnets
  - Output:
    - None
- `eks_node_group_iam`: Creates an EKS node group IAM role for a given service
  - Input
    - `service_name`: The name of the common service you're provisioning for
  - Output
    - `role_arn`: The ARN of the role associated with EKS Node Group IAM
- `eks_user_data`: Gets a list of IAM users from a list of given usernames
  - Input
    - `cluster_name`: The name of the cluster to which this EKS User Data task description is being assigned
    - `user_names`: The list of usernames of group members
  - Output
    - `cluster_name`: The name of the target EKS cluster
    - `user_names`: The list of user ARNs that need access to the EKS cluster
- `fargate`: Provisions an ECS task with the Fargate launch type
  - Input
    - `cluster_arn`: The ARN of the cluster to which this ECS task description is being assigned
    - `container_definitions`: An object containing definitions for each of the containers involved in a task
      - `name`: The name of a container
      - `image`: The image used to start a container
      - `essential`: Option to decide if the task stops upon container failure
      - `port_mappings`: Allows containers to access ports on the host container instance to send or receive traffic
        - `containerPort`: The port number on the container that's bound to the user-specified or automatically assigned host port
        - `hostPort`: The port number on the container instance to reserve for your container
    - `cpu`: The amount of vCPU to reserve
    - `memory`: The amount of memory to reserve
    - `services`: The name of the Fargate services to provision for a given task definition
      - `name`: The name of Fargate service
      - `desired_count`: The number services you need for Fargate
    - `subnets`: A list of subnet IDs to provision to
    - `task_name`:  The name of the Fargate task being generated
  - Output
    - None
- `internet_gateway`: Provision an internet gateway for a VPC
  - Inputs
    - `internet_gateway_name`: The name of the internet gateway
    - `vpc_id`: The VPC ID for the Internet Gateway
  - Outputs
    - `internet_gateway_id`: The ID of the internet gateway
    - `internet_gateway_name`: The name of the internet gateway
- `json_formatting`: Format the JSON file for the target dashboard
  - Inputs
    - `height`: The height of a widget on the dashboard
    - `list_of_widgets`: A list of objects representing each widget that is going in the dashboard. Automatically generated using the output from the *_alarms modules
      - `name`: The name of widget
      - `alarms`: The list of alarms for the widget
    - `num_widgets`: The maximum number of widgets in a row of the dashboard
    - `width`: The width of a widget on the dashboard
  - Outputs
    - `widget_json`: A formatted JSON string containing the dashboard body of the target dashboard
- `lambda`: Provision Lambda functions
  - Inputs
    - `filename`: The path to the .zip file that contains your Lambda function code
    - `role_arn`: The role ARN for Lambda IAM
    - `security_group_ids`: List of security group IDs associated with the Lambda function
    - `service_name`: The name of the common service you're provisioning for
    - `subnet_ids`: The list of IDs for the subnets
    - `tags`: All applicable tags for the Lambda function
  - Outputs
    - `function_name`: The name of the Lambda function
- `lambda_alarms`: Provision alarms for Lambda function
  - Inputs
    - `function_name`: Name of target Lambda function
    - `sns_arn`: ARN of the appropriate SNS topic based on service
  - Outputs
    - `function_name`: The function name of the target Lambda function
    - `list_of_alarm_arns`: A list of all alarms provisioned for this module. Used to create a service overview dashboard
- `lambda_data`: Gather a list of Lambda functions associated with a service
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `function_list` : A list of Lambda functions associated with that service
- `lambda_iam`: Provision service-specific roles for a Lambda function
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `role_arn`: The ARN of the role being provisioned
- `nacl`: Provision Network ACL for a VPC
  - Inputs
    - `cidr_block`: CIDR block associated with the VPC
    - `network_acl_name`: The name of the Network ACL being provisioned
    - `rules`: An object containing a list of rules associated with the Network ACL
      - `from_port`: The port that data is coming from
      - `to_port`: The port that the data going to
      - `protocol`: The type of connection protocol (TCP/UDP/-1)
      - `rule_action`: Whether to “allow" or “deny" the traffic
      - `rule_number`: Determines the order in which rules are executed
      - `egress`: Whether or not the rule applies to outgoing traffic
    - `subnet_ids`: The list of IDs for the subnets
  - Outputs
    - `network_acl_id`: The ID of the Network ACL being provisioned
- `nat_gateway`: Provision a NAT gateway to connect private subnets to the Internet or privately to other VPCs
  - Inputs
    - `connectivity_type`: Whether the NAT gateway is public (connects to Internet) or private (connects to other VPCs)
    - `gateway_name`: The name of the NAT gateway
    - `subnet_id`: ID of the VPC subnet to provision in
  - Outputs
    - `nat_gateway_id`: The ID of the NAT gateway
    - `nat_gateway_name`: The name of the NAT gateway
- `route_table`: Provision route tables within a VPC
  - Inputs
    - `gateway_map`: A map of the names and IDs of all of the gateways associated with a service
    - `route_table_name`: The name of the route table being provisioned
    - `routes`: Provision routes for a route table within a VPC
      - `destination_cidr_block`: The IPv4 CIDR range specified in a route in the table
      - `gateway_name`: The name of a gateway
      - `gateway_type`: The type of gateway
    - `subnet_ids`: The IDs of the subnet to associate the route table with
  - Outputs
    - `route_table_id`: The ID of the route table being provisioned
- `security_groups`: Provision security groups for a VPC
  - Inputs
    - `cidr_blocks`: CIDR blocks associated with the VPC
    - `description`: A description of the security group being provisioned
    - `rules`: A map of the rules list to be associated with the security group
      - `from_port`: The port that data is coming from
      - `to_port`: The port that the data going to
      - `protocol`: The type of connection protocol (TCP/UDP/-1)
      - `type`: Set a way to send data through a specific port (ingress/egress)
    - `security_group_name`: The name of the security group being provisioned
    - `vpc_id`: The VPC ID for the security group
  - Outputs
    - `security_group_id`: The ID of the security group being provisioned
- `server`: Provision web server EC2 instances
  - Inputs
    - `ami`: The AMI of the servers you'd like to provision
    - `cluster_name`: The name of the cluster to which this server task description is being assigned
    - `instance_profile`: The instance profile to associate with the server (needed for container instances)
    - `private_ips`: The private IP addresses of the server's private network interface
    - `security_group`: ID of the security group that should apply to the server's network interface
    - `subnet_id`: ID of the VPC subnet to provision in
    - `tags`: All applicable tags for a web server
  - Outputs
    - `instance_id`: The instance ID of the server instance
    - `instance_name`: The name of the server instance
    - `instance_public_ip`: The public IP of the instance
- `sns`: Create a service topic in AWS SNS and subscribe everyone in list_of_emails to it
  - Inputs
    - `list_of_emails`: A list of emails that you would like to receive updates for the dashboard alarms
    - `name`: The name of the SNS topic (usually service name)
  - Outputs
    - `topic_arn`: The ARN for the SNS topic
- `subnet`: Provision subnets for a VPC
  - Inputs
    - `availability_zone`: The availability zone of the subnet
    - `cidr_block`: The CIDR Block associated with the VPC
    - `subnet_type`: The subnet type (public/private)
    - `tags`: All applicable tags for a subnet
    - `vpc_id`: The VPC ID for the subnet
  - Outputs
    - `cidr_block`: The CIDR block occupied of the subnet
    - `subnet_id`: The ID of the subnet
    - `subnet_name`: The name of the subnet
- `vpc`: Provision a VPC
  - Inputs
    - `cidr_block`: The CIDR Block associated with the VPC
    - `vpc_name`: The name of the VPC
  - Outputs
    - `cidr_block`: The CIDR Block of the VPC
    - `vpc_id`: The ID of the VPC
    - `vpc_name`: The name of the VPC
- `vpc_alarms`: Provision alarms for a VPC
  - Inputs
    - `sns_arn`: ARN of the appropriate SNS topic based on service
    - `vpc_name`: The VPC ID for the VPC alarm
  - Outputs
    - `list_of_alarm_arns`: A list of all alarms provisioned for this module. Used to create a service overview dashboard
    - `vpc_id`: The ID of the VPC
- `vpc_data`: Reads the list of active VPCs for alarm provisioning (used in dash_only)
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `list_of_vpcs`: A list of IDs corresponding to all active VPCs
- `vpc_log_group`: Get the packets, bytes, vpc-id, and flow-direction from the CloudWatch Log Groups
  - Inputs
    - `log_group_name`: The name of the CloudWatch log group associated with the flow log
    - `role_arn`: The ARN of the IAM role
    - `traffic_type`: The type of traffic capture to the log. (ACCEPT/ALL/REJECT)
    - `vpc_ids`: The VPC IDs for the flow logs
  - Output
    - `log_group_name`: The name of the CloudWatch log group associated with the flow log
- `vpc_log_group_iam`: IAM Role for VPC Log Groups
  - Inputs
    - `service_name`: The name of the common service you're provisioning for
  - Outputs
    - `role_arn`: The ARN of the role being provisioned
- `vpc_metric_filter`: Creates metric filters to turn log data into VPC metrics (for use in alarms)
  - Inputs
    - `log_group_name`: The name of the CloudWatch log group associated with the metric filter
  - Outputs
    - None

# Known Issues
A list of known issues, errors, or improvements that need to be addressed in the future.
1. Trying to use multiple default subnets within a default VPC can cause errors or undesirable behavior. With the current implementation, you can only use multiple default subnets iteratively (`a, b, c` is OK, `a, c` is not). If you include the default subnet 3 times within a single `subnets[]` list of objects, you would get 3 different default subnets for services that require multi-subnet support (like `eks_cluster`).
2. It’s also possible to include an availability zone where there isn’t a default subnet (`us-east-1b` in `us-east-1`, for example). Until module `precondition {}` support is added in Terraform, you will have to do some error catching on the subnets input using a `can(contains())` (or similar function) within each module
3. We specify a lot of `object()` or `list(object())` variables in Terraform. As of the [1.3.0 prerelease](https://github.com/hashicorp/terraform/releases/tag/v1.3.0-alpha20220803), there is planned support for `optional()` attributes within objects. **We would highly recommend implementing this feature as soon as it is released to the public, so that the user does not have to specify every single field within an object for each object.**
