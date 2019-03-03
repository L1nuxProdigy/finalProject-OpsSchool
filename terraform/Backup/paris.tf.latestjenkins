##################################################################################
# VARIABLES
##################################################################################

### Connection Vars ###
variable "aws_access_key" {}
variable "aws_secret_key" {}
variable "private_key_path" {}
variable "aws_key_name" {
  default = "paris"
}

### AMI Vars ###
variable "ubuntu_image_18-04" {default = "ami-08182c55a1c188dee"}
variable "ubuntu_image_16-04" {default = "ami-075b44448d2276521"}
variable "RHEL_image" {default = "ami-5026902d"}

### Machines Configurations Scripts ###

# Ansible
variable "rhel_ansible_server_user_data_script" {}
variable "ubuntu_ansible_server_user_data_script" {}

# Jenkins
variable "jenkins_server_user_data_script" {}
variable "jenkins_another_user_data_script" {}
variable "jenkins_node_user_data_script" {}

# Kubernetes
variable "kubernetes_server_user_data_script" {}
variable "kubernetes_node_user_data_script" {}



##################################################################################
# PROVIDERS
##################################################################################

provider "aws" {
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "eu-west-3"
}

##################################################################################
# IAM Resources
##################################################################################

#
## Consul IAM Resources ##
#
resource "aws_iam_instance_profile" "Consul_IAM_Profile" {
  name  = "Consul_Profile"
  role = "${aws_iam_role.Consul_IAM_Role.name}"
}

resource "aws_iam_role_policy" "Consul_IAM_Policy" {
  name = "Consul-Describe-Policy"
  role = "${aws_iam_role.Consul_IAM_Role.id}"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:DescribeInstances"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
EOF
}

resource "aws_iam_role" "Consul_IAM_Role" {
  name = "Consul-Role"
  description = "Created By Terraform"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

#
## Jenkins IAM Resources ##
#
resource "aws_iam_instance_profile" "Jenkins_IAM_Profile" {
  name  = "Jenkins_Profile"
  role = "${aws_iam_role.Jenkins_IAM_Role.name}"
}

resource "aws_iam_role_policy" "Jenkins_IAM_Policy" {
  name = "Jenkins-Describe-Policy"
  role = "${aws_iam_role.Jenkins_IAM_Role.id}"

  policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "ec2:*",
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "elasticloadbalancing:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "autoscaling:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:AWSServiceName": [
                        "autoscaling.amazonaws.com",
                        "ec2scheduled.amazonaws.com",
                        "elasticloadbalancing.amazonaws.com",
                        "spot.amazonaws.com",
                        "spotfleet.amazonaws.com",
                        "transitgateway.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
EOF
}

resource "aws_iam_role" "Jenkins_IAM_Role" {
  name = "Jenkins-Role"
  description = "Created By Terraform"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

##################################################################################
# VPC Resources
##################################################################################

resource "aws_vpc" "VPC_main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = "TRUE"
  tags = {
	Name = "Terraform_VPC"
  }
}

resource "aws_subnet" "Subnet_main" {
  vpc_id     = "${aws_vpc.VPC_main.id}"
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = "TRUE"

  tags = {
	Name = "Terraform_Subnet"
  }
}

resource "aws_internet_gateway" "IG_main" {
  vpc_id = "${aws_vpc.VPC_main.id}"

  tags = {
    Name = "Terraform_IG"
  }
}

resource "aws_route_table" "Route_Table_main" {
  vpc_id = "${aws_vpc.VPC_main.id}"

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.IG_main.id}"
  }

  tags = {
    Name = "Terraform_Route"
  }
}

resource "aws_main_route_table_association" "VPC_Route_Association" {
  vpc_id         = "${aws_vpc.VPC_main.id}"
  route_table_id = "${aws_route_table.Route_Table_main.id}"
}

resource "aws_route_table_association" "VPC_Subnet_Association" {
  subnet_id      = "${aws_subnet.Subnet_main.id}"
  route_table_id = "${aws_route_table.Route_Table_main.id}"
}

	
resource "aws_security_group" "SecurityGroup_main" {
	name        = "Terraform_SG"
	description = "ssh for now"
	vpc_id      = "${aws_vpc.VPC_main.id}"

	ingress {
		from_port   = 22
		to_port     = 22
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "SSH"
      }
	ingress {
		from_port   = 65433
		to_port     = 65433
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Dummy Exporter APP"
      }
	ingress {
		from_port   = 5601
		to_port     = 5601
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Kibana"
      }
	ingress {
		from_port   = 3000
		to_port     = 3000
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Grafana"
      }
	ingress {
		from_port   = 9200
		to_port     = 9200
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "ElasticSearch"
      }
	ingress {
		from_port   = 9090
		to_port     = 9090
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Prometheus"
      }
	ingress {
		from_port   = 8300
		to_port     = 8300
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Consul"
	}
	ingress {
		from_port   = 8301
		to_port     = 8301
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Consul"
	}
	ingress {
		from_port   = 8500
		to_port     = 8500
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Consul Web"
	}
	ingress {
		from_port   = 8080
		to_port     = 8080
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "Jenkins"
	}
	ingress {
		from_port   = 30001
		to_port     = 30001
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "K8s Expose Service"
      }
	ingress {
		from_port   = 6443
		to_port     = 6443
		protocol    = "TCP"
		cidr_blocks = ["0.0.0.0/0"]
		description = "K8s cluster join"
	}
	egress {
		from_port       = 0
		to_port         = 0
		protocol        = "-1"
		cidr_blocks     = ["0.0.0.0/0"]
        
	}
	
	tags = {
    Name = "Terraform_SG"
  }
}

##################################################################################
# EC2 Resources
##################################################################################

resource "aws_instance" "Ansible_and_Consul_Server" {
	ami           = "${var.ubuntu_image_18-04}"
	instance_type = "t2.micro"
	key_name        = "${var.aws_key_name}"
	subnet_id = "${aws_subnet.Subnet_main.id}"
	vpc_security_group_ids = ["${aws_security_group.SecurityGroup_main.id}"]
	iam_instance_profile = "${aws_iam_instance_profile.Consul_IAM_Profile.name}"
	
	tags = {
	Name = "Ansible_Server-TerraBuild"
	}
	
	user_data = "${file(var.ubuntu_ansible_server_user_data_script)}"
}

resource "aws_instance" "Jenkins_Server" {
	ami           = "${var.ubuntu_image_18-04}"
	instance_type = "t2.micro"
	key_name        = "${var.aws_key_name}"
	subnet_id = "${aws_subnet.Subnet_main.id}"
	vpc_security_group_ids = ["${aws_security_group.SecurityGroup_main.id}"]
	iam_instance_profile = "${aws_iam_instance_profile.Jenkins_IAM_Profile.name}"
	
	tags = {
	Name = "Jenkins_Server-TerraBuild"
	}
	
	user_data = "${file(var.jenkins_server_user_data_script)}"
}

resource "aws_instance" "Jenkins_Slave" {
	ami           = "${var.ubuntu_image_18-04}"
	instance_type = "t2.micro"
	key_name        = "${var.aws_key_name}"
	subnet_id = "${aws_subnet.Subnet_main.id}"
	vpc_security_group_ids = ["${aws_security_group.SecurityGroup_main.id}"]
	iam_instance_profile = "${aws_iam_instance_profile.Consul_IAM_Profile.name}"
	
	tags = {
	Name = "Jenkins_Node-TerraBuild"
	}
	
	user_data = "${file(var.jenkins_node_user_data_script)}"
}

resource "aws_instance" "Jenkins_Another" {
	ami           = "${var.ubuntu_image_18-04}"
	instance_type = "t2.micro"
	key_name        = "${var.aws_key_name}"
	subnet_id = "${aws_subnet.Subnet_main.id}"
	vpc_security_group_ids = ["${aws_security_group.SecurityGroup_main.id}"]
	iam_instance_profile = "${aws_iam_instance_profile.Consul_IAM_Profile.name}"
	
	tags = {
	Name = "Jenkins_Another-TerraBuild"
	}
	
	user_data = "${file(var.jenkins_another_user_data_script)}"
}