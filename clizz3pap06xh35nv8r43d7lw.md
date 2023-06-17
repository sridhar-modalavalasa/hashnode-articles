---
title: "Build a two-tier architecture for AWS using Terraform Modules."
datePublished: Sat Jun 17 2023 12:26:42 GMT+0000 (Coordinated Universal Time)
cuid: clizz3pap06xh35nv8r43d7lw
slug: build-a-two-tier-architecture-for-aws-using-terraform-modules
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686983030837/15bd25bf-e885-4d39-a016-83c059d4539e.webp
tags: two-tier-architecture-using-terraform

---

* Custom VPC with CIDR 10.0.0.0/16.
    
* Two Public Subnets with CIDR 10.0.1.0/24 and 10.0.2.0/24 in different Availability Zones for high availability.
    
* Two Private Subnets with CIDR 10.0.3.0/24 and 10.0.4.0/24 in different Availability Zones.
    
* RDS MySQL instance (micro) in One of the Two Private Subnets.
    
* One Application Load Balancer (External) — Internet-facing, which will direct the traffic to the Public Subnets.
    
* Two EC2 t2.micro instances in each Public Subnet
    
* Leveraging modules to create reusable and shareable
    
* Using variables and data sources to make your code flexible and maintainable.
    
* State files should be stored remotely.
    

### **Terraform:**

**What is Terraform?**

Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It is used to define and provision a complete infrastructure using a declarative language. IaC helps businesses automate their infrastructures by programmatically managing an entire technology stack through code.

**Terraform core concepts:**

* **Variables**: Also used as input variables, it is key-value pair used by Terraform modules to allow customization.
    
* **Provider**: It is a plugin to interact with APIs of service and access its related resources. (We will be using AWS for this project)
    
* **Module**: It is a folder with Terraform templates where all the configurations are defined.
    
* **Resources**: refers to a block of one or more infrastructure objects (compute instances, virtual networks, etc.), which are used in configuring and managing the infrastructure.
    
* **Output Values**: These are return values of a terraform module that can be used by other configurations.
    
* **Plan**: It is one stage where it determines what needs to be created, updated, or destroyed.
    
* **Apply**: It is the last stage where it applies the changes of the infrastructure move to the desired state.
    

Read more about terraform in the below link

[Terraform deep dive](https://developer.hashicorp.com/terraform/docs)

**What are terraform modules?**

A Terraform module allows you to create logical abstraction on the top of some resource set. In other words, a module will enable you to group resources and reuse this group later, possibly many times. Terraform module allows us to use the concept of DRY (Don’t Repete Yourself). With the use of terraform modules, you can write the code for various resources once and reuse them in a different environment as per your need!

you can read more details about terraform modules here below link:

[Modules concept deep dive](https://itoutposts.com/blog/terraform-modules-overview/)

**What are root & child Modules?**

The root module calls the child module and includes the child module’s resources. You can call a child module multiple times within the same configuration, and multiple root configurations can use the same child module.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686984100176/a82a5a03-dae3-4c5e-b7dc-36e946cce33e.webp align="center")

### **Prerequisites:**

* Visual studio code.
    
* Install Terraform CLI in server.
    
* Install AWS CLI and configure the AWS credentials in Server.
    
* GitHub account.
    
* AWS account access with access key and secret key.
    

### **Architecture:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686984998695/ab3ce470-3418-4900-86ac-3a834c5a73b6.png align="center")

[Click here for the GitHub to complete the project](https://github.com/sridhar-modalavalasa/Terraform-2-Tier)

### **Build Five Modules:**

We will create five child modules\*\***<mark>: VPC, EC2 instance, RDS MySQL instance, Application-Load-Balancer, and Security Group.</mark>**\*\*

This was the structure and modules I followed to create this two-tier architecture as shown below. For more details check in the GitHub repository.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686986432467/f4237634-b75a-462a-875f-9ec1d9f8ae88.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686985464675/371b9b12-2d8b-4400-95f3-66b527bfb27a.png align="center")

Open your source code editor, create, and name your new folder.

```plaintext
mkdir Terraform-2-Tier
cd Terraform-2-Tier
touch main.tf outputs.tf variables.tf
```

Create four files named main.tf, variables.tf, providers.tf and .gitignore

Open **.gitignore** and copy and paste the following

```plaintext
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*
*.tfplan

# Crash log files
crash.log

# Exclude all .tfvars files, which are likely to contain sentitive data, such as
# password, private keys, and other secrets.
*.tfvars
```

When it comes to version control. It is good practice not to include your .tfstate or .tfvars because they contain sensitive data. Instead, it’s best to upload them into an encrypted bucket or Terraform Cloud.

**<mark>The remaining files will be used for our root module, but we will configure that later on.</mark>**

### **1.VPC Module:**

* Create a new folder named or directory **Modules**
    
* Enter the folder Modules and create a new folder named **vpc.**
    
* Create three files named **main.tf, outputs.tf, and variables.tf** inside the **vpc** folder.
    

```plaintext
#---/Modules/vpc---

# create vpc
resource "aws_vpc" "vpc" {
  cidr_block           = var.vpc_cidr
  instance_tenancy     = "default"
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# create internet gateway and attach it to vpc
resource "aws_internet_gateway" "internet_gateway" {
  vpc_id = aws_vpc.vpc.id

  tags = {
    Name = "${var.project_name}-igw"
  }
}

# use data source to get all avalablility zones in region
data "aws_availability_zones" "available_zones" {}

# create public subnet az1
resource "aws_subnet" "public_subnet_az1" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.public_subnet1_az1_cidr
  availability_zone       = data.aws_availability_zones.available_zones.names[0]
  map_public_ip_on_launch = true

  tags = {
    Name = "public subnet1"
  }
}

# create public subnet az2
resource "aws_subnet" "public_subnet_az2" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.public_subnet2_az2_cidr
  availability_zone       = data.aws_availability_zones.available_zones.names[1]
  map_public_ip_on_launch = true

  tags = {
    Name = "public subnet2"
  }
}

# create route table and add public route
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.internet_gateway.id
  }

  tags = {
    Name = "public route table"
  }
}

# associate public subnet az1 to "public route table"
resource "aws_route_table_association" "public_subnet_az1_route_table_association" {
  subnet_id      = aws_subnet.public_subnet_az1.id
  route_table_id = aws_route_table.public_route_table.id
}

# associate public subnet az2 to "public route table"
resource "aws_route_table_association" "public_subnet_az2_route_table_association" {
  subnet_id      = aws_subnet.public_subnet_az2.id
  route_table_id = aws_route_table.public_route_table.id
}

# create private data subnet az1
resource "aws_subnet" "private_data_subnet_az1" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.private_subnet1_az1_cidr
  availability_zone       = data.aws_availability_zones.available_zones.names[2]
  map_public_ip_on_launch = false

  tags = {
    Name = "private subnet1"
  }
}

# create private data subnet az2
resource "aws_subnet" "private_data_subnet_az2" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.private_subnet2_az2_cidr
  availability_zone       = data.aws_availability_zones.available_zones.names[3]
  map_public_ip_on_launch = false

  tags = {
    Name = "private subnet2"
  }
}

####Create NAT gateway 

resource "aws_nat_gateway" "nat_gateway" {
  connectivity_type = "public"
  subnet_id         = aws_subnet.public_subnet_az1.id
  allocation_id     = aws_eip.eip_nat_gateway.id
  tags = {
    Name = "NAT_GW"
  }
}

resource "aws_eip" "eip_nat_gateway" {
  depends_on = [aws_internet_gateway.internet_gateway]
  vpc        = true

  tags = {
    Name = "EIP"
  }
}

####Craete private route table#####
resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.nat_gateway.id
  }

  tags = {
    Name = "private_route_table"
  }
}

# associate private subnet az1 to "private route table"
resource "aws_route_table_association" "private_subnet_az1_route_table_association" {
  subnet_id      = aws_subnet.private_data_subnet_az1.id
  route_table_id = aws_route_table.private_route_table.id
}

# associate private subnet az2 to "private route table"
resource "aws_route_table_association" "private_subnet_az2_route_table_association" {
  subnet_id      = aws_subnet.private_data_subnet_az2.id
  route_table_id = aws_route_table.private_route_table.id
}
```

Open the **main.tf** file inside the vpc folder and copy and paste the following code into your **main.tf** file.

I have included the following terraform resources in this module: **<mark>VPC, internet gateway, route table, route table association, four availability zones, two public subnets, two private subnets, Elastic IP, NAT gateway and data source to retrieve AZs.</mark>**

Also, you will notice that most values are stored in variables; this is a better practice than hard coding the values.

Open the **variables.tf** file inside the **vpc** folder and copy and paste the following code into your **variables.tf** file.

```plaintext
#---/Modules/vpc---
variable "project_name" {
  type        = string
  description = "This configures the vpc name"
}
variable "vpc_cidr" {
  type        = string
  description = "This configures the vpc cidr"
}
variable "public_subnet1_az1_cidr" {
  type        = string
  description = "This configures the public subnet1 cidr"
}
variable "public_subnet2_az2_cidr" {
  type        = string
  description = "This configures the public subnet2 cidr"
}
variable "private_subnet1_az1_cidr" {
  type        = string
  description = "This configures the private subnet1 cidr"
}
variable "private_subnet2_az2_cidr" {
  type        = string
  description = "This configures the private subnet2 cidr"
}
```

**<mark>We will not define all the child modules in the variables files. We will define them using the terraform.tfvars file in the root module later.</mark>**

Open the **outputs.tf** file inside the vpc folder and copy and paste the following code into your **outputs.ts** file.

```plaintext
#--/Modules/vpc---
output "project_name" {
  value = var.project_name
}

output "vpc_id" {
  value = aws_vpc.vpc.id
}

output "public_subnet1_az1" {
  value = aws_subnet.public_subnet_az1.id
}

output "public_subnet1_az2" {
  value = aws_subnet.public_subnet_az2.id
}

output "private_subnet1_az1" {
  value = aws_subnet.private_data_subnet_az1.id
}

output "private_subnet1_az2" {
  value = aws_subnet.private_data_subnet_az2.id
}

output "internet_gateway" {
  value = aws_internet_gateway.internet_gateway
}

output "subnet_ids" {
  value = [aws_subnet.public_subnet_az1.id, aws_subnet.public_subnet_az2.id]
}
```

**<mark>Outputs export values from an existing module to be used by other modules, including the root module.</mark>**

**<mark>For example, if I need to reference my vpc id as an attribute for my security group. I will export my vpc id in outputs, then enter the output name as a variable in the security group module as var.vpc_id, then add vpc_id to my security group variable list, then define it in the root module as “module.vpc.vpc_id”.</mark>**

**This concludes the VPC Module.**

### 2.Security-Group Module

* Enter the folder Modules and create a new folder named security-group
    
* Create three files named **main.tf**, **outputs.tf,** and **variables.tf** inside the **security-group** folder.
    

```plaintext
mkdir security-group
cd security-group
touch main.tf outputs.tf variables.tf
```

Open the **main.tf** file inside the security-group folder and copy and paste the following code.

```plaintext
#create security group for the application load balancer
resource "aws_security_group" "alb_security_group" {
  name        = "alb security group"
  description = "enable http access on port 80"
  vpc_id      = var.vpc_id


  ingress {
    description = "http access"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "alb_sg"
  }
}

# create security group for SSH 
resource "aws_security_group" "ssh-security-group" {
  name        = "SSH Access"
  description = "enable ssh access on port 22"
  vpc_id      = var.vpc_id


  ingress {
    description = "ssh access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "SSH Security Group"
  }
}

# create security group for the webserver via alb

resource "aws_security_group" "ec2_security_group" {
  name        = "ec2 security group"
  description = "enable http access on port 80 via alb sg & ssh port 22 via internet"
  vpc_id      = var.vpc_id


  ingress {
    description     = "http access"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = ["${aws_security_group.alb_security_group.id}"]
  }

  ingress {
    description     = "ssh access"
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = ["${aws_security_group.ssh-security-group.id}"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Webserver Security Group"
  }
}

# create security group for the Database

resource "aws_security_group" "database-security-group" {
  name        = "Database Security Group"
  description = "Enable MYSQL access on port 3306 "
  vpc_id      = var.vpc_id


  ingress {
    description     = "MYSQL access"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = ["${aws_security_group.ec2_security_group.id}"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Database Security Group"
  }
}
```

For the security group module, I created four resources which are security groups.

* **<mark>The first security group is for the application load balancer port 80 facing the internet.</mark>**
    
* **<mark>The second security group is for ssh port 22 access to the Ec2.</mark>**
    
* **<mark>The third security group is for our ec2 instances that will only receive traffic via the application load balancer’s security group.</mark>**
    
* **<mark>The fourth security group is for the database instance, which will receive traffic via the ec2 instance security group</mark>**
    

Open the **variables.tf** file inside the security-group folder and copy and paste the following code.

```plaintext
#---/Modules/security-group---

variable "vpc_id" {}
```

**<mark>Because I used a variable that was exported from our vpc module, I had to add it to our variables file. This variable will be defined in the root module.</mark>**

Open the **outputs.tf** file inside the security-group folder and copy and paste the following code.

```plaintext
#---/Modules/security-group---

output "alb_security_group" {
  value = aws_security_group.alb_security_group.id
}

output "ssh_security_group" {
  value = aws_security_group.ssh-security-group.id
}

output "ec2_security_group" {
  value = aws_security_group.ec2_security_group.id
}

output "database-security-group" {
  value = aws_security_group.database-security-group.id
}
```

**This concludes the security-group module.**

### 3.EC2 Module

* Enter the folder Modules and create a new folder named ec2
    
* Create three files named **main.tf, outputs.tf, and variables.tf** inside the ec2 folder.
    

```plaintext
mkdir ec2
cd ec2
touch main.tf outputs.tf variables.tf
```

Open the **main.tf** file inside the ec2 folder and copy and paste the following code.

```plaintext
#---/Modules/ec2---

# compute module main.tf

resource "aws_instance" "app_server" {
  count = length(var.subnet_ids)

  ami                    = var.ami
  instance_type          = var.instance_type
  key_name               = var.keyname
  subnet_id              = var.subnet_ids[count.index]
  vpc_security_group_ids = [var.ec2_security_group]
  tags = {
    Name = "Webserver ${count.index}"
  }
}
```

**<mark>For our instances, we only need one resource. Notice the number of instances will be based on the number of subnets entered in the subnet_ids variable. Also, subnet_ids is a variable exported from our vpc module in the outputs.tf.</mark>**

Open the **variables.tf** file inside the ec2 folder and copy and paste the following code.

```plaintext
variable "ami" {}
variable "instance_type" {}
variable "ec2_web_tag_name" {}
variable "keyname" {}
variable "ec2_security_group" {}
variable "subnet_ids" {
  type = list(string)
}
```

**<mark>Variable “subnet_ids,” has to be specified as a list since the variable’s value will have multiple names.</mark>**

Open the **outputs.tf** file inside the ec2 folder and copy and paste the following code.

```plaintext
#---/Modules/ec2---

output "aws_instance" {
  value = aws_instance.app_server
}
```

**This concludes the ec2 module.**

### 4.Load Balancer Module

* Enter the folder Modules and create a new folder named alb
    
* Create two files named **main.tf** and **variables.tf** inside the alb folder.
    

```plaintext
mkdir alb
cd alb
touch main.tf variables.tf
```

Open the **main.tf** file inside the alb folder and copy and paste the following code.

```plaintext
#---/Modules/alb---

resource "aws_lb_target_group" "target-group" {
  health_check {
    enabled             = true
    interval            = 300
    path                = "/"
    timeout             = 60
    matcher             = 200
    healthy_threshold   = 5
    unhealthy_threshold = 5
  }
  name        = "${var.project_name}-tg"
  port        = 80
  protocol    = "HTTP"
  target_type = "instance"
  vpc_id      = var.vpc_id
}

resource "aws_lb" "application-lb" {
  name               = "${var.project_name}-alb"
  internal           = false
  ip_address_type    = "ipv4"
  load_balancer_type = "application"
  security_groups    = [var.alb_security_group]
  subnets            = var.subnet_ids

  tags = {
    Environment = "${var.project_name}-alb"
  }
}

resource "aws_lb_listener" "alb-listener" {
  load_balancer_arn = aws_lb.application-lb.arn
  port              = "80"
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.target-group.arn
  }
}



resource "aws_lb_target_group_attachment" "ec2_attach" {
  count            = length(var.aws_instance)
  target_group_arn = aws_lb_target_group.target-group.arn
  target_id        = var.aws_instance[count.index].id

}
```

**<mark>For this module, we need four resources. Target group, load balancer, listening port, and target group attachment.</mark>**

**<mark>Notice for target group attachment; the target ec2 is based on the count.index, so based on the number of instances you have will be placed in the target group.</mark>**

Open the **variables.tf** file inside the alb folder and copy and paste the following code.

```plaintext
#---/Modules/alb---

variable "vpc_id" {}
variable "alb_security_group" {}
variable "subnet_ids" {
  type = list(string)
}
variable "aws_instance" {}
variable "project_name" {}
```

**This concludes the application-load-balancer module.**

### 5.RDS Instance Module

* Enter the folder Modules and create a new folder named database
    
* Create two files named **main.tf** and **variables.tf** inside the database folder.
    

```plaintext
mkdir database
cd database
touch main.tf variables.tf
```

Open the **main.tf** file inside the database folder and copy and paste the following code.

```plaintext
#---/Modules/database---

resource "aws_db_subnet_group" "db_subnet" {
  name       = "dbsubnet"
  subnet_ids = [var.private_subnet1_az1, var.private_subnet1_az2]

  tags = {
    Name = "dbsubnet"
  }
}

resource "aws_db_instance" "mysql" {
  identifier          = var.identifier
  engine              = var.engine
  engine_version      = var.engine_version
  instance_class      = "db.t3.micro"
  allocated_storage   = 20
  skip_final_snapshot = true

  db_name  = var.db_name
  username = var.username
  password = var.password

  vpc_security_group_ids = [var.database-security-group]

  db_subnet_group_name = aws_db_subnet_group.db_subnet.name


}
```

Open the **variables.tf** file inside the database folder and copy and paste the following code.

```plaintext
#---/Modules/database---

variable "identifier" {}
variable "engine" {}
variable "engine_version" {}
variable "db_name" {}
variable "username" {
  sensitive = true
}
variable "password" {
  sensitive = true
}
variable "database-security-group" {}

variable "private_subnet1_az1" {}
variable "private_subnet1_az2" {}
```

**<mark>Notice for variables “username” &amp; “password,” I set the sensitive value to true. This will not display the value when terraform displays the infrastructure when deploying.</mark>**

**This concludes the database module.**

**<mark>Let’s put everything together in the root module</mark>**

Navigate to the root folder which is outside of the Modules folder.

```plaintext
cd ../.. (From database module)
or
cd ..(From Module folder)
```

Open the **main.tf** file inside the Terraform-2-tier folder (this would be the root folder you named) and copy and paste the following code.

```plaintext
#---/root---

#create vpc
module "vpc" {
  source                   = "./Modules/vpc"
  project_name             = var.project_name
  vpc_cidr                 = var.vpc_cidr
  public_subnet1_az1_cidr  = var.public_subnet1_az1_cidr
  public_subnet2_az2_cidr  = var.public_subnet2_az2_cidr
  private_subnet1_az1_cidr = var.private_subnet1_az1_cidr
  private_subnet2_az2_cidr = var.private_subnet2_az2_cidr

}

module "security-group" {
  source = "./Modules/security-group"
  vpc_id = module.vpc.vpc_id
}

#create an ec2 instance
module "Ec2" {
  source             = "./Modules/ec2"
  ami                = var.ami
  instance_type      = var.instance_type
  ec2_web_tag_name   = var.ec2_web_tag_name
  keyname            = var.keyname
  ec2_security_group = module.security-group.ec2_security_group
  subnet_ids         = module.vpc.subnet_ids

}

#create a RDS database instance

module "database" {
  source                  = "./Modules/database"
  identifier              = var.identifier
  engine                  = var.engine
  engine_version          = var.engine_version
  db_name                 = var.db_name
  username                = var.username
  password                = var.password
  database-security-group = module.security-group.database-security-group
  private_subnet1_az1     = module.vpc.private_subnet1_az1
  private_subnet1_az2     = module.vpc.private_subnet1_az2

}

#create an application load balancer

module "alb" {
  source             = "./Modules/alb"
  vpc_id             = module.vpc.vpc_id
  alb_security_group = module.security-group.alb_security_group
  subnet_ids         = module.vpc.subnet_ids
  aws_instance       = module.Ec2.aws_instance
  project_name       = var.project_name

}
```

<mark>For our root </mark> [<mark>main.tf</mark>](http://main.tf) <mark>we need to input all the modules that were created. After inputting your module, you must specify the source path of your module. Then, every variable that has not been defined in that module must be defined in the root module within that same module block. If an attribute is referenced from another module, you must define the variable as “module.modulename.outputname”</mark>

<mark>As a form of good practice, I use variables in the </mark> [<mark>main.tf</mark>](http://main.tf) <mark>instead of hard coding our values.</mark>

Copy all variables from each module and paste them into our root variables.

**<mark>Do Not copy variables that were exported from another Module onto the root variables file. Also do not duplicate any of the same variables.</mark>**

Open the **variables.tf** and paste the code which is shown in below.

```plaintext
#---/root---

#vpc
variable "project_name" {
  type        = string
  description = "This configures the vpc name"
}
variable "vpc_cidr" {
  type        = string
  description = "This configures the vpc cidr"
}
variable "public_subnet1_az1_cidr" {
  type        = string
  description = "This configures the public subnet1 cidr"
}
variable "public_subnet2_az2_cidr" {
  type        = string
  description = "This configures the public subnet2 cidr"
}
variable "private_subnet1_az1_cidr" {
  type        = string
  description = "This configures the private subnet1 cidr"
}
variable "private_subnet2_az2_cidr" {
  type        = string
  description = "This configures the private subnet2 cidr"
}

#ec2
variable "ami" {}
variable "instance_type" {}
variable "ec2_web_tag_name" {}
variable "keyname" {}
variable "region" {}


#database
variable "identifier" {}
variable "engine" {}
variable "engine_version" {}
variable "db_name" {}
variable "username" {
  sensitive = true
}
variable "password" {
  sensitive = true
}
```

Open the **providers.tf** file inside the Terraform-2-tier folder (this would be the root folder you named) and copy and paste the following code.

```plaintext
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }

# backend block
  backend "s3" {
    bucket = "s3-backend-10wocops"
    key    = "Terraform_Two_Tier/terraform.tfstate"
    region = "us-east-1"
  }
}

# Configure the AWS Provider

provider "aws" {
  region = "us-east-1"
  
}
```

This confirms the platform and version to use to create our resources. You can have multiple provider blocks.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686999135632/caea44d3-539a-4a27-84c1-ed1c7436a299.png align="center")

**<mark>We are using the backend block for remotely storing the terraform.tfstate file. we have to create the bucket first and then we have to use it in the backend block in Terraform. where the key provides the path of the file inside the bucket.</mark>**

Create a file named **terraform.tfvars**

```plaintext
touch terraform.tfvars
```

**<mark>Within this file, we will input all of our variable values.</mark>**

```plaintext
region                   = "us-east-1"
project_name             = "10WOCOps-week-03"
vpc_cidr                 = "10.0.0.0/16"
public_subnet1_az1_cidr  = "10.0.1.0/24"
public_subnet2_az2_cidr  = "10.0.2.0/24"
private_subnet1_az1_cidr = "10.0.3.0/24"
private_subnet2_az2_cidr = "10.0.4.0/24"
ami                      = "ami-022e1a32d3f742bd8"
instance_type            = "t2.micro"
ec2_web_tag_name         = "week03"
keyname                  = "shh-sree"
identifier               = "db-mysql"
engine                   = "mysql"
engine_version           = "8.0.32"
db_name                  = "dbmysql"
username                 = "user"
password                 = "password"
```

**<mark>Make the necessary adjustments to your variables based on your AWS account. For example, use your existing key name for your ec2 instance.</mark>**

**<mark>Don't look for the access_key and secret_key. I have configured those values by using aws configure command through aws cli. It is not a best practice of exposing the specific user credentials in the files.</mark>**

**<mark>I have pasted link below for configuring the aws credentials:</mark>**

[aws configure credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

**This concludes the root module.**

**Let’s install our modules and providers and put everything together.**

Run the following command

```plaintext
terraform init
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686997540406/66a2f6d2-dbc3-42cc-ba3c-33c9df712612.png align="center")

Ensure that our syntax is correct with the following command

```plaintext
terraform validate
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686998392141/f2dd6a20-f5db-4067-bfab-59b0a8e41dce.png align="center")

Run the following command to display the infrastructure that will be created.

```plaintext
terraform plan
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686998550643/03e258cb-c7f7-4353-98a4-a2d9dff089f6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686998673515/f4d5d8aa-b183-441f-b814-abe4a06910b5.png align="center")

At this point, you should see the entire layout of your infrastructure.

Run the infrastructure.

```plaintext
terraform apply
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686998791535/b0fc1bd9-a7f0-4445-848c-169b7a5aba1b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686998903505/d16d67d9-c104-4b9f-bd9e-f2d30853057f.png align="center")

Log onto the console and confirm the resources have been created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686999775310/c5cb31d6-77b9-4097-9eca-18afa04f9964.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686999865565/b85b8792-afa7-49e8-867f-f6a183cf18dc.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686999930566/78c53de6-bb99-4998-8293-f86db2bce9ad.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686999996234/d837fa1a-d96b-4f94-bbe9-1b37b2f74048.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002155057/37c6ea52-896d-41e8-bef6-06605f75f8c0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002213227/d83a06dc-a474-4444-bc8b-042e9892e634.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002342349/d4c28d1c-52c5-469f-a869-1eeb9fbae367.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002462526/ff5dd4a4-ba6b-4f74-a6f9-97894ec69b2c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002546138/82d2cf34-0e7a-49e8-ada1-b5bef5bb56ad.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002612051/b21d98e9-419e-4f07-98b6-39db7e09a95a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002680982/4556daa1-a8f3-4f78-9aa5-eb52268fec8c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002789713/ee1797e7-a16c-430c-b052-fcd38fe1a1c1.png align="center")

**<mark>Finally, we have implemented an Two tier architecture in AWS Cloud using Terraform modules concept.</mark>**

Now we have to check in the S3 Storage bucket whether the terraform state file has been stored or not in the given bucket.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687002989071/bd686bcb-12ee-4fa0-ac03-08e9916f5ff5.png align="center")

The folder have been created now we will enter in to the bucket to see the terraform state file has been created or not.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687003082712/aec85b35-f908-4888-9327-e24f5ec9e071.png align="center")

The state file has been stored in the AWS S3 storage by terraform backend module we have used in the HCL.

```plaintext
terraform destroy
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687003439394/1f104fe1-017c-4e15-b0dd-7618c87ac307.png align="center")

### References:

[https://www.youtube.com/watch?v=LiyFnMOhhtE](https://www.youtube.com/watch?v=LiyFnMOhhtE)

[https://registry.terraform.io/browse/modules](https://registry.terraform.io/browse/modules)

[https://registry.terraform.io/providers/hashicorp/aws/latest](https://registry.terraform.io/providers/hashicorp/aws/latest)

[https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

**GitHub Repository:**

[MY GITHUB REPOSITORY](https://github.com/sridhar-modalavalasa)

**<mark>search for the above project repo Terraform-2-tier</mark>**

### Success!

**If everything was done correctly, you’ve just successfully implemented a two-tier architecture in AWS using Terraform and storing the state file remotely in Amazon S3!**

If you’ve got this far, **thanks for reading!** I hope it was worthwhile for you.

Happy Learning.

# **#10WeeksOfCloudOps**