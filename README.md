# Define provider and AWS region
provider "aws" {
  region = "us-west-2"  # Change this to your desired AWS region
}

# Define variables
variable "cluster_name" {
  default = "my-k8s-cluster"
}

variable "instance_type" {
  default = "t2.micro"  # Change this to the instance type you prefer
}

variable "node_count" {
  default = 3  # Number of nodes in the Kubernetes cluster
}

# Create an AWS VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}

# Create an AWS Internet Gateway for the VPC
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id
}

# Create a public subnet within the VPC
resource "aws_subnet" "public_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-west-2a"  # Change this to your desired availability zone
  map_public_ip_on_launch = true
}

# Create an AWS security group for the Kubernetes nodes
resource "aws_security_group" "k8s_nodes_sg" {
  vpc_id = aws_vpc.my_vpc.id

  # Allow inbound traffic from anywhere to port 22 (SSH)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow inbound traffic from the VPC to port 6443 (Kubernetes API server)
  ingress {
    from_port   = 6443
    to_port     = 6443
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.my_vpc.cidr_block]
  }

  # Allow outbound traffic to anywhere
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create AWS IAM role for the Kubernetes nodes
