# End-to-End Cloud Automation with Terraform, Ansible, LB, and GitHub
Cloud Infrastructure Automation with Terraform and Ansible
![End-to-End Cloud Automation with Terraform, Ansible, LB, and GitHub](https://github.com/user-attachments/assets/d4bd950a-2afc-462b-8d32-011d4cc2d768)

## OverView-Setup: 
In this project I use Terraform as Infrastucture as code tool, By using Terraform i create Entire Infrastucrture on AWS Cloud 

## Tools/Technology use:
  This Entire project is fully automatic only set RoundRobin backend IPs is maunal(90% Automation & 10% Manual)
1. Terraform (Create Ansible Master-Slave Architecture for LB)
2. AWS Cloud(EC2, VPC, Subnet, Internate_gateway, Route_table, Security_Group)
3. Shell Scripting (Download Ansible, Clone my LB GitHub repo)
4. GitHub(Kept Ansible-playbook for LB & BackEnd)
5. Ansible(using Terraform dynamically create Inventory, set Ansible config file)
6. Load Balancer(haproxy LB is used as FrontEnd)
7. Apache webserver(Httpd webserver used as BackEnd for LB)


##  1.Data Source: aws_ami
**most_recent = true** -> This retrive latest ami in AWS.

**owners = ["amazon"]** -> AMIs owned by Amazon.

    data "aws_ami" "PS-ami-block" {
      most_recent = true
      owners = ["amazon"]
      filter {
        name = "name"
        values = ["amzn2-ami-kernel-5.10-hvm-*-x86_64-gp2"]
      }
      filter {
        name = "root-device-type"
        values = ["ebs"]
      }
      filter {
        name = "virtualization-type"
        values = ["hvm"]
     }
    }

 ## 2.  Resource: aws_vpc
 **cidr_block:** Defines the IP range for the VPC.

    resource "aws_vpc" "PS-vpc-block" {
      cidr_block = "10.0.0.0/16"
      tags = {
        Name = "TF-Pratik-vpc"
      }
    }

 ## 3. Resource: aws_subnet
**count:** --> The count meta-argument allows the creation of multiple resources based on a list or number. Here, it creates a subnet for each
CIDR range in var.SubnetRange.

**vpc_id:** Links the subnet to the previously created VPC.

**cidr_block:** The CIDR block for the subnet, dynamically assigned from the SubnetRange variable.The IP address range for each subnet
is dynamically set by the element function

**availability_zone:** Defines the availability zone for the subnet, chosen from the AZRange variable. This assigns the subnet to a specific 
Availability Zone (AZ) using the element function again.

**map_public_ip_on_launch:** Ensures that instances launched in this subnet will automatically receive a public IP.

    resource "aws_subnet" "PS-Subnet-block" {
      count = length(var.SubnetRange)
      vpc_id = aws_vpc.PS-vpc-block.id
      cidr_block = element(var.SubnetRange, count.index)
      availability_zone = element(var.AZRange, count.index)
      map_public_ip_on_launch = true
      tags = {
        Name = "TF-Pratik-Subnet"
      }
      depends_on = [
        aws_vpc.PS-vpc-block
      ]
    }
 
