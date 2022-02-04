# Create-VPC-with-Terraform
Provisioning a vpc with terraform


### Vpc Module Creation

```mkdir -p /var/terraform/modules/vpc```

 ```tree /var/terraform/```
/var/terraform/
└── modules
    └── vpc

2 directories, 0 files

```touch /var/terraform/modules/vpc/{main.tf,output.tf,variables.tf,datasource.tf}```

``` tree /var/terraform/```
/var/terraform/
└── modules
    └── vpc
        ├── datasource.tf
        ├── main.tf
        ├── output.tf
        └── variables.tf

2 directories, 4 files

### variables.tf
```vim /var/terraform/modules/vpc/variables.tf```


variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "project" {
   default = "example"
}

variable "env" {
  default = "test"
}

### datasource.tf
```vim /var/terraform/modules/vpc/datasource.tf```

data "aws_availability_zones" "az" {
  state = "available"
}

### main.tf

 
### **VPC Creation**

```
resource "aws_vpc" "vpc"  {

  cidr_block = var.vpc_cidr
  instance_tenancy = "default"
  enable_dns_support = true
  enable_dns_hostnames = true

    tags = {
       Name = "${var.project}-vpc-${var.env}"
       project = var.project
       environment = var.env
  }
}
```


### **InterNet GateWay Creation**

```
resource "aws_internet_gateway" "igw"  {
  vpc_id = aws_vpc.vpc.id
   
    tags = {
    Name = "${var.project}-igw-${var.env}"
    project = var.project
    environment = var.env
  }
}
```


### **Public Subnet 1**

```
resource "aws_subnet" "public1"  {
  vpc_id = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr,"3", 0)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[0]
 
tags = {
    Name = "${var.project}-public1-${var.env}"
    project = var.project
    environment = var.env
 }
}
```

 ### **Public Subnet 2**

```
resource "aws_subnet" "public2"  {
  vpc_id = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr,"3", 1)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[1]

tags = {
    Name = "${var.project}-public2-${var.env}"
    project = var.project
    environment = var.env
 }
}
```


### **Public Subnet 3**


```
resource "aws_subnet" "public3"  {
  vpc_id = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr,"3", 2)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[2]

tags = {
    Name = "${var.project}-public3-${var.env}"
    project = var.project
    environment = var.env
 }
}
```




### **Private Subnet 1**


```
resource "aws_subnet" "private1"  {
  vpc_id = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr,"3", 3)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[0]

tags = {
    Name = "${var.project}-private1-${var.env}"
    project = var.project
    environment = var.env
 }
}
```


### **Private Subnet 2**


```
resource "aws_subnet" "private2"  {
  vpc_id = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr,"3", 4)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[1]

tags = {
    Name = "${var.project}-private2-${var.env}"
    project = var.project
    environment = var.env
 }
}
```


### **Private Subnet 3**


```
resource "aws_subnet" "private3"  {
  vpc_id = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr,"3", 5)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[2]

tags = {
    Name = "${var.project}-private3-${var.env}"
    project = var.project
    environment = var.env
 }
}
```

### **ElasticIp for NatGateway**

```
resource "aws_eip" "nat"  {
  vpc = true
  
    tags = {
    Name = "${var.project}-nat-${var.env}"
    project = var.project
    environment = var.env
  }
}
```


### **NatGateway  Creation**

```
resource "aws_nat_gateway" "nat"  {
  allocation_id = aws_eip.nat.id
  subnet_id	= aws_subnet.public1.id
   
  tags = {
    Name = "${var.project}-nat-${var.env}"
    project = var.project
    environment = var.env
  }
  depends_on = [aws_internet_gateway.igw]
}
```


###  **Public RouteTable**

```
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.vpc.id

  route {
  cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
   Name = "${var.project}-public-${var.env}"
   project = var.project
   environment = var.env
   }
}
```

###  **Private RouteTable**

```
resource "aws_route_table" "private" {
    
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
  
  tags = {
    Name = "${var.project}-private-${var.env}"
    project = var.project
    environment = var.env
  }
}
```

###  **Public RouteTable association**

```
resource "aws_route_table_association" "public1" {
  subnet_id    = aws_subnet.public1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public2" {
  subnet_id    = aws_subnet.public2.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public3" {
  subnet_id    = aws_subnet.public3.id
  route_table_id = aws_route_table.public.id
}
```


###  **Private RouteTable association**

```
resource "aws_route_table_association" "private1" {
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private2" {
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private3" {
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private.id
}
```


### terraform init

[root@ip-172-31-42-108 zomato-project]# vim /var/terraform/modules/vpc/main.tf 
[root@ip-172-31-42-108 zomato-project]# ``` terraform init ``` 
```
Initializing modules...

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v3.74.0...
- Installed hashicorp/aws v3.74.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

[root@ip-172-31-42-108 zomato-project]# ``` tree -A -a ```
.
```
├── main.tf
├── output.tf
├── provider.tf
├── .terraform
│   ├── modules
│   │   ├── modules.json
│   │   └── vpc -> /var/terraform/modules/vpc
│   └── providers
│       └── registry.terraform.io
│           └── hashicorp
│               └── aws
│                   └── 3.74.0
│                       └── linux_amd64
│                           └── terraform-provider-aws_v3.74.0_x5
├── .terraform.lock.hcl
└── variables.tf

9 directories, 7 files
```

### Validate the terraform code 
[root@ip-172-31-42-108 zomato-project]# ``` terraform validate ```
Success! The configuration is valid.

### terraform plan
```Plan: 24 to add, 0 to change, 0 to destroy.```

```
Changes to Outputs:
  + bastion_public_ip    = (known after apply)
  + database_private_ip  = (known after apply)
  + webserver_private_ip = (known after apply)
  + webserver_public_ip  = (known after apply)
  ```
  
  ### Terraform Apply (-auto-approve)
  
 ```terraform apply -auto-approve```
 
 Outputs:

bastion_public_ip = "3.110.154.158"
database_private_ip = "172.20.100.54"
webserver_private_ip = "172.20.25.134"
webserver_public_ip = "65.1.109.22"

 
 






