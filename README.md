# Automating AWS Infrastructure Using Terraform

After successfully building AWS infrastructure manually for 2 websites, we will now automate the entire process using Terraform. This automation will ensure consistency, repeatability, and easier maintenance of our infrastructure.

## **Step 1: Understanding the Goal**  
The objective is to automate AWS infrastructure creation using Terraform, an Infrastructure as Code (IaC) tool. Instead of manually creating resources in AWS (like a VPC, subnets, and security groups), we will write Terraform code to automate the process.

---

## **Step 2: Set Up Your Work Environment**  

### **1. Install Required Tools**  
Ensure you have the following installed on your computer:  
✅ **Terraform**: Download and install from [Terraform official website](https://developer.hashicorp.com/terraform/downloads). 

To download Terraform I used the the chocolatey on Windows option as downloading the zip did not work for me. find screenshots below
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/download%20terraform.PNG)

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/verify%20terraform%20installation.PNG)

✅ **AWS CLI**: Install it by following [AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html). 

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/installing%20AWS%20CLI.PNG)

✅ **Python & Boto3**: Install Python (if not installed) and use `pip install boto3` to install Boto3.  
I already have python installed so I will go ahead and install boto
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/python%20version.PNG)
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/installing%20boto.PNG)

✅ **Visual Studio Code (VS Code)**: A code editor where you'll write Terraform scripts.  
Click on this website to download ![Download vscode](https://code.visualstudio.com/docs/setup/windows)


---

## **Step 3: Create AWS IAM User for Terraform**  

### **1. Log into AWS Console**  
1. Go to [AWS Console](https://aws.amazon.com/console/).  
2. Search for "IAM" in the search bar and open **IAM (Identity & Access Management)**.  
3. Click **Users** > **Add users**.  

### **2. Create a New User**  
1. **User Name**: `terraform`  
2. **Access Type**: Check **Programmatic access**.  
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20iam%20in%20aws.PNG)

### **3. Set Permissions**  
1. Choose **Attach policies directly**.  
2. Search for `AdministratorAccess` and attach it.  
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20iam%20in%20aws1.PNG)

3. Click **Next**, then **Create user**.  

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20iam%20in%20aws2.PNG)

### Create Access Keys:

After user creation, create access keys
Store these securely - they will be used for AWS CLI configuration

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20access.PNG)
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20access1.PNG)
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20access2.PNG)

### **4. Copy Access Keys**  
1. Copy the **Access Key ID** and **Secret Access Key** and save them in a Notepad (you’ll use them later). 
![(screenshot) ](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20access3.PNG)

### Security Best Practices:

Never commit access keys to version control
Rotate access keys regularly
Use the principle of least privilege when assigning permissions
Enable MFA for the IAM user if possible

---

## **Step 4: Configure AWS CLI**  
Open your terminal (Git Bash for Windows, Terminal for Mac). Run:  
```bash
aws configure
```
Enter the following when prompted:  
- **AWS Access Key ID**: (paste the key from step 3.4)  
- **AWS Secret Access Key**: (paste the key)  
- **Region**: `us-east-1` (or any region closest to you)  
- **Output format**: `json`  
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/configure%20aws%20cli%20on%20git%20bash.PNG)

To verify, run:
```bash
aws s3 ls
```
If it returns a list of buckets or no errors, your AWS CLI is correctly configured.

---
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/configure%20aws%20cli%20on%20git%20bash1.PNG)

## **Step 5: Create an S3 Bucket for Terraform State File**  
Terraform keeps track of infrastructure changes in a **state file**. To store it securely, create an S3 bucket.  

1. Run this command (replace `yourname` with your actual name):  
   ```bash
   aws s3 mb s3://yourname-dev-terraform-bucket
   ```
2. To confirm the bucket is created, run:  
   ```bash
   aws s3 ls
   ```

---
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/creating%20s3%20bucket.PNG)

## **Step 6: Verify AWS Connection with Python Boto3**  
Open **Python (or VS Code terminal)** and run:  
```python
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```
If it prints your S3 bucket name, you're good to go.

---
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/Verify%20AWS%20Connection%20with%20Python%20Boto3.PNG)

### ⚠️ S3 Backend Considerations:

Enable versioning on the S3 bucket
Configure server-side encryption
Use DynamoDB for state locking
Implement appropriate bucket policies

## **Step 7: Set Up Terraform Project**  
### **1. Create a New Project Directory**  
1. Open **VS Code**.  
2. Create a new folder `PBL`.  
3. Inside `PBL`, create a file **main.tf**.  

---
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/Create%20folder%20PBL%20and%20main%20tf.PNG)

Make sure to also download Terraform and AWS extension that are compatible with VSCODE

## **Step 8: Write Terraform Code**  

### **1. Define the AWS Provider and VPC in `main.tf`**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "172.16.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/AWS%20Provider%20and%20VPC.PNG)
✅ **Explanation**:  
- `provider "aws"` tells Terraform we are working with AWS.  
- `aws_vpc` creates a **Virtual Private Cloud (VPC)**, which is like a private network for your AWS resources.  

---

### **2. Initialize Terraform**  
Run:  
```bash
terraform init
```
This downloads necessary Terraform plugins.

---

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terraform%20innit.PNG)

### **3. Plan and Apply Infrastructure**  
1. Run to see what Terraform will create:  
   ```bash
   terraform plan
   ```
   ![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terraform%20plan.PNG)
2. Apply the changes:  
   ```bash
   terraform apply
   ```

3. Type `yes` to confirm.  

---
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terraform%20apply.PNG)
 
We need to **create six subnets** inside our **VPC (Virtual Private Cloud)**:  
✅ **2 public subnets** (for resources that need internet access).  
✅ **2 private subnets for web servers** (internal use, no direct internet access).  
✅ **2 private subnets for database storage** (highly secure, no direct internet access).  

Terraform allows us to **automate** this process instead of creating them manually in the AWS console.


## **📌 Step 9: Define Subnets in Terraform (Basic Version)**  
We'll start by **hardcoding** the subnets before optimizing the code.

### **1️⃣ Add Public Subnets to `main.tf`**
📌 **Open `main.tf` and add this code:**
```hcl
# Create Public Subnet 1
resource "aws_subnet" "public1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "172.16.0.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "eu-central-1a"
}

# Create Public Subnet 2
resource "aws_subnet" "public2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "172.16.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "eu-central-1b"
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/public%20subnets.PNG)
🔹 **Explanation:**  
- Each `aws_subnet` resource **creates a subnet** inside the VPC.  
- `cidr_block` defines **the IP range** assigned to each subnet.  
- `map_public_ip_on_launch = true` means EC2 instances launched here get **public IPs**.  

### **2️⃣ Add Private Subnets for Web Servers**
📌 **Add this below the public subnets:**
```hcl
# Private Subnet 1 for Web Server
resource "aws_subnet" "private_web1" {
  vpc_id           = aws_vpc.main.id
  cidr_block       = "172.16.2.0/24"
  availability_zone = "eu-central-1a"
}

# Private Subnet 2 for Web Server
resource "aws_subnet" "private_web2" {
  vpc_id           = aws_vpc.main.id
  cidr_block       = "172.16.3.0/24"
  availability_zone = "eu-central-1b"
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/private%20subnets.PNG)
🔹 **Explanation:**  
- These **do not** have `map_public_ip_on_launch = true` because web servers are private.  
- They are located in **different availability zones** for **high availability**.  

### **3️⃣ Add Private Subnets for Database Layer**
📌 **Add this below the private web subnets:**
```hcl
# Private Subnet 1 for Database
resource "aws_subnet" "private_db1" {
  vpc_id           = aws_vpc.main.id
  cidr_block       = "172.16.4.0/24"
  availability_zone = "eu-central-1a"
}

# Private Subnet 2 for Database
resource "aws_subnet" "private_db2" {
  vpc_id           = aws_vpc.main.id
  cidr_block       = "172.16.5.0/24"
  availability_zone = "eu-central-1b"
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/database%20subnets.PNG)
🔹 **Explanation:**  
- These are meant for **databases** like MySQL or PostgreSQL.  
- They are kept **private for security** (databases should not be exposed to the internet).  

---

## **🛠 Step 10: Run Terraform Commands**  
Now, let’s **test and deploy** the subnets.

### **1️⃣ Initialize Terraform**
Run this command **inside the `PBL` folder**:  
```bash
terraform init
```
✅ **What this does:** It downloads required AWS plugins.

### **2️⃣ Preview the Changes**
Run:  
```bash
terraform plan
```
✅ **What this does:** It **checks for errors** before applying changes.

### **3️⃣ Apply the Changes**
Run:  
```bash
terraform apply
```
Terraform will ask for confirmation. Type **`yes`** and press **Enter**.  
✅ **What this does:** It **creates the subnets** inside AWS.

---

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/our%20subnet%20created%20in%20vs%20code.PNG)
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/the%20subnets%20in%20aws%20console.PNG)

### step 11 VPC and Networking Configuration (SoftCoded - Introducing Variable - VERSION)
Now, we will run terraform destroy to remove all the resources we have created in our aws account through terraform so that we can redo it, this time around, in a more flexible, dynamic way.
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terraform%20destroy.PNG)

Create a file called variables.tf, in this file is where we will declare our variables to use in the main.tf file. We could declare these variables in the main.tf file, but for the sake of making our work orderly and easy to read, we are separating the files for their respective purpose.

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/variable%20tf%20created.PNG)

Variable Definition:

```
# variables.tf
variable "region" {
  default = "us-east-1"
}

variable "vpc_cidr" {
  default = "172.16.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "preferred_number_of_public_subnets" {
  default = null
}

variable "vpc_tags"{
 description = "Tags to be applied to the VPC"
 type        = map(string)
 default = {
     Environment = "production"
     Terraform   = "true"
     Project     = "PBL"
 }
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/content%20od%20variable%20tf.PNG)

#### Explicit variable Definitions

Create another file called terraform.tfvars
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/create%20terraform%20tfvars.PNG)

This file sets actual values for the variables declared in variables.tf. It's used to:

Override specific default values
Set environment-specific configurations
Keep sensitive values separate from main code
Find below and replicate the content of the terraform.tfvars file:

```
# terraform.tfvars
region = "us-east-1"
vpc_cidr = "172.16.0.0/16"
enable_dns_support = true
enable_dns_hostnames = true
preferred_number_of_public_subnets = 2
vpc_tags = {
   Name = "Production-VPC"
   Environment = "production"
   Terraform = "true"
   Project = "PBL"
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/content%20terraform%20tfvars.PNG)

#### Main Configuration Updates 

Once you have applied the values in the terraform.tfvars file, we will now proceed to the main.tf file. This file contains the primary infrastructure configuration. It:

References variables using the var. prefix
Defines resources and their relationships
Contains provider configurations

```
# main.tf
provider "aws" {
  region = var.region
}

# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support
  enable_dns_hostnames           = var.enable_dns_hostnames
  tags = {
      Name        = "Production-VPC"
      Environment = "production"
      Terraform   = "true"
      Project     = "PBL"
}
}

# Dynamically create public subnets
resource "aws_subnet" "public" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/content%20of%20main%20tf.PNG)

At the end, our file structure should look somewhat like this:

```
PBL/
├── main.tf           # Main configuration file
├── variables.tf      # Variable declarations
├── terraform.tfvars  # Variable values
```

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/structure%20of%20our%20terraform%20project.PNG)

#### Understanding the Dynamic Subnet Creation
Let us break down the subnet configuration:
```
resource "aws_subnet" "public" {
count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
vpc_id                  = aws_vpc.main.id
cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
map_public_ip_on_launch = true
availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```
i. Key Parts:
Count Parameter

```
count = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
```
This is a conditional expression (ternary operator)
If preferred_number_of_public_subnets is null, it uses the number of available AZs
If not null, it uses the specified number from terraform.tfvars
Controls how many subnets are created


ii. VPC Association
```
vpc_id = aws_vpc.main.id
```
Links the subnet to the VPC we created
Uses interpolation (the method of referencing/assigning values) to reference the VPC's ID

iii. CIDR Block Calculation
```
cidr_block = cidrsubnet(var.vpc_cidr, 4, count.index)
```
cidrsubnet() is a built-in function that calculates subnet CIDR blocks
Parameters:
var.vpc_cidr: The VPC's CIDR block (e.g., "172.16.0.0/16")
4: Number of additional bits for subnet (creates /20 subnets)
count.index: Current subnet number (0, 1, 2, etc.)
Example outputs:
First subnet (index 0): 172.16.0.0/20
Second subnet (index 1): 172.16.16.0/20
Third subnet (index 2): 172.16.32.0/20

iv. Public IP Assignment
```
map_public_ip_on_launch = true
```
Automatically assigns public IPs to instances in this subnet
Makes the subnet "public"

v. Availability Zone Assignment
```
availability_zone = data.aws_availability_zones.available.names[count.index]
```
- Distributes subnets across available AZs
- Uses count.index to cycle through AZs
- Ensures high availability by spreading across zones

This configuration creates a dynamic number of public subnets, each in a different availability zone, with automatically calculated CIDR blocks. It is flexible enough to adapt to different regions and requirements while it strives to maintain best practices for high availability.

Best Practices for VPC Configuration
CIDR Block Selection:

Use private IP ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
Plan for future growth when selecting CIDR ranges
Document your IP addressing scheme
Subnet Design:

Distribute subnets across multiple AZs
Size subnets according to expected workload
Keep some IP ranges in reserve for future use (if and when necessary - as this costs money to store/keep)
Tagging: Multiple tagging is strongly advised

```
tags = {
  Name        = "Production-VPC"
  Environment = "production"
  Terraform   = "true"
  Project     = "PBL"
}
```

### Step 12 Testing Your Configuration
Validate Configuration:
```
terraform validate
```
Review Plan:
```
terraform plan
```
Apply Configuration:
```
terraform apply
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/tearraform%20validate%20and%20plan.PNG)
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terrafrom%20apply%20last.PNG)

##### Verify Resources:
run thr below
```
terraform show
```
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terraform%20show.PNG)
![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/terraform%20show1.PNG)

Now This is the result from our Aws console point of view

![(screenshot)](https://github.com/Prince-Tee/IAC_AWSinfrastructureUsingTerraform/blob/main/Screenshot%20from%20my%20env/saws%20console%20finale.PNG)

Conclusion
Project Achievements
Through this Infrastructure as Code (IaC) implementation with Terraform, we have successfully:

Automated the creation of a production-grade VPC infrastructure
Implemented dynamic subnet allocation across availability zones
Established a maintainable and scalable code structure
Applied AWS best practices for networking and security