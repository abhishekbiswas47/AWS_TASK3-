![alt text](https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcSqbDdtw15lXcffklE79HPByzmbarOFIUeoQQ&usqp=CAU)
# AWS_TASK3-

## Task Description-

We have to create a web portal for our company with all the security as much as possible.
So, we use Wordpress software with dedicated database server.
Database should not be accessible from the outside world for security purposes.
We only need to public the WordPress to clients.
So here are the steps for proper understanding!

Steps:
1) Write a Infrastructure as code using terraform, which automatically create a VPC.

2) In that VPC we have to create 2 subnets:
    a)  public  subnet [ Accessible for Public World! ] 
    b)  private subnet [ Restricted for Public World! ]

3) Create a public facing internet gateway for connect our VPC/Network to the internet world and attach this gateway to our VPC.

4) Create  a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.

5) Launch an ec2 instance which has Wordpress setup already having the security group allowing  port 80 so that our client can connect to our wordpress site.
Also attach the key to instance for further login into it.

6) Launch an ec2 instance which has MYSQL setup already with security group allowing  port 3306 in private subnet so that our wordpress vm can connect with the same.
Also attach the key with the same.

Note: Wordpress instance has to be part of public subnet so that our client can connect our site. 
mysql instance has to be part of private  subnet so that outside world can't connect to it.
Don't forgot to add auto ip assign and auto dns name assignment option to be enabled.

### Prerequisite-
1. AWS Console Account
2. AWS CLI
3. Terraform Installed
4. Putty or Mobaxterm

## Lets Begin!!!
#### Step 1:
##### Defining the provider on which we have to create infrastructure.
    provider "aws" {
    region = “ap-south-1”
    profile = “default”
    }

#### Step 2:
##### Creating a VPC with CIDR Block 192.168.0.0/16 and enable DNS Hostnames to assign dns names to instances.
    resource “aws_vpc” “myvpc”{
     cidr_block = “192.168.0.0/16”
     instance_tenancy = “default”
     enable_dns_hostnames = true
    tags = {
     Name = “abhishekvpc”
     }
    }

#### Step 3:
##### Creating public and private subnet.
##### Public subnet-
    resource "aws_subnet" "public" {
     vpc_id     = aws_vpc.vpc.id
     cidr_block = "192.168.10.0/24"
     availability_zone = "ap-south-1b"
     map_public_ip_on_launch = "true"
    tags = {
      Name = "public-subnet"
     }
    }
##### Private subnet-
    resource "aws_subnet" "private" {
     vpc_id     = aws_vpc.vpc.id
     cidr_block = "192.168.20.0/24"
     availability_zone = "ap-south-1a"
    tags = {
      Name = "private-subnet"
     }
    }

#### Step 4:
#####  Creating internet gateway to connect our VPC to internet.
    resource "aws_internet_gateway" "gateway" {
     vpc_id = aws_vpc.myvpc.id
    tags = {
      Name = "abhi_gateway"
     }
    }

#### Step 5:
##### Write a Terraform code to Create a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.
    resource "aws_route_table" "route" {
      vpc_id = aws_vpc.vpc.id
           route {
              cidr_block = "0.0.0.0/0"
              gateway_id = aws_internet_gateway.gateway.id
          }
    tags = {
                 Name = "gatewayroute"
          }
       }
    resource "aws_route_table_association" "public"
     {
      subnet_id   = aws_subnet.public.id
      route_table_id = aws_route_table.route.id
     }

#### Step 6:
##### Write a terraform code to configure security groups allowing SSH, HTTP and TCP.
    resource "aws_security_group" "task3-sg" {
      name        = "task3-sg"
      description = "Allow traffic SSH,TCP, HTTP"
      vpc_id      = aws_vpc.vpc.id
    ingress {
        description = "HTTP"
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }
  
    ingress {
        description = "SSH"
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }
    ingress {
        description = "TCP"
        from_port   = 3306
        to_port     = 3306
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
        Name = "task3-sg"
      }
    }

#### Step 7:
##### Write a terraform code to Launch an ec2 instance which has Wordpress setup already having the security group allowing port 80 so that our client can connect to our wordpress site.
    resource "aws_instance" "wordpress" {
       ami = "ami-004a955bfb611bf13"
       instance_type = "t2.micro"
       associate_public_ip_address = true
       subnet_id = aws_subnet.public.id
       vpc_security_group_ids = [ aws_security_group.task3.id]
       key_name = "aniket1234"
    tags = { 
             Name = "abhios"
         }
     }
To use our terraform code first we have to initialize it by using this command-:

    terraform init

After that we have to run this command and the terraform will perform the task.

    terraform apply --auto-approve

###Few screenshots of the task-
