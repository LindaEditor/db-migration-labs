# Using Terraform to Create Networks and Firewalls

## Overview

In this lab, you will create a secure network infrastructure for you database migration projects. You will create both public and private VPCs with appropriate firewall rules in each. You will add virtual machines to each network and test the commnication between them. You will do all of this using Terraform to demonstrate a more real-world workflow that you can use in your migration projects. 

### Objectives

In this lab, you will learn how to perform the following tasks:

*   Use Terraform to automate network creation
*   Use Terraform to create firewall rules
*   Use Terraform to create virtual machines
*   Use Terraform to create a private network


## Task 0. Lab Setup

In this task, you use Qwiklabs and perform initialization steps for your lab.

### Access Qwiklabs

![[/fragments/startqwiklab]]

After you complete the initial sign-in steps, the project dashboard appears.

![GCP Project Dashboard](img/gcpprojectdashboard.png)

Click __Select a project__, highlight your _GCP Project ID_, and click
__OPEN__ to select your project.

![[/fragments/cloudshell]]

## Task 1. Use Terraform to automate network creation

1.  In the Navigation menu ( ![Menu](img/menu.png) ), click on **Home**.

1.  In the **Project info** section, find your Project ID and copy and paste it into a text file. You will need it later.

1.  Click on the **Activate Cloud Shell** icon in the upper right of the console. ![Cloud Shell](img/CloudShell.png) <p>The Cloud Shell terminal will open in a pane at the bottom.</p>

1.  Make a directory called `terraform-networks` and change to it. 

```
mkdir terraform-networks
cd terraform-networks
```

1.  Use the following commands to create the required Terraform files for this lab. 

```
touch provider.tf
touch terraform.tfvars
touch test-server-linux.tf
touch variables.tf
touch vpc-network-public.tf
touch vpc-firewall-rules-public.tf
touch public-test-server-linux.tf
touch random-id-generator.tf
```

1.  Type `ls` to verify your files were created in the **terraform-networks** folder.

1.  Click on the **Open Editor** button in Cloud Shell. <p>Select the **terraform-networks** folder and open the `provider.tf` file. Enter the following code to configure the Google Cloud Terraform provider.</p>

```
terraform {
  required_version = ">= 0.12"
}

provider "google" {
  project = var.project_id
  region  = var.gcp_region_1
  zone    = var.gcp_zone_1
}
```

1.  Notice the variables in the above code. You will create those and some other variables now. Open the `variables.tf` file and enter the following code.

```
# GCP Project ID 
variable "project_id" {
  type = string
  description = "GCP Project ID"
}

# Region to use for Subnet 1
variable "gcp_region_1" {
  type = string
  description = "GCP Region"
}

# Zone used for VMs
variable "gcp_zone_1" {
  type = string
  description = "GCP Zone"
}

# Define subnet for public network
variable "subnet_cidr_public" {
  type = string
  description = "Subnet CIDR for Public Network"
}
```

1.  In the previous file you defined the variables, you set the variables in another file. Open the `terraform.tfvars` file and add the following code.<p>***Enter your project ID where indicated.***

```
# GCP Settings
project_id    = "your-project-id" 
gcp_region_1  = "us-central1"
gcp_zone_1    = "us-central1-a"

# GCP Network Variables
subnet_cidr_public  = "10.1.1.0/24"
```

1.  Now open the `vpc-network-public.tf` file and add the following to it. 

```
resource "google_compute_network" "public-vpc" {
  name                    = "public-vpc"
  auto_create_subnetworks = "false"
  routing_mode            = "GLOBAL"
}

resource "google_compute_subnetwork" "public-subnet_1" {
  name          = "public-subnet-1"
  ip_cidr_range = var.subnet_cidr_public
  network       = google_compute_network.public-vpc.name
  region        = var.gcp_region_1
}
```

<aside><p><strong>Note: </strong>The above Terraform code creates a VPC with one subnet. In the subnet, notice the two variables for `ip_cidr_range` and `region`. Also notice how the  `network` property refers back the the VPC created before the subnet.</p></aside>


1.  Let's see if this works up to this point. In the Navigation menu of the Console ( ![Menu](img/menu.png) ), click on **VPC network**. At this point you should have one network named `default`

1.  In Cloud Shell, click on the ***Open Terminal*** button. Make sure you are in the right folder.

```
cd ~/terraform-networks
```

1.  Enter the following code to initialize Terraform.

```
terraform init
```
<aside><p><strong>Note: </strong>It should indicate that "Terraform has been successfully initialized!".</p></aside>

1.  Enter the following code to build the Terraform plan. Make sure there are no errors and take a look at what resources will be created. 

```
terraform plan
```

<aside><p><strong>Note: </strong>The plan should tell you that 2 resources will be created, a network and a subnetwork.</p></aside>

1.  If all looks good, run the follow command (*the -auto-approve parameter simply runs the script with prompting you*).
```
terraform apply -auto-approve
```

1.  Wait for the script to complete. Then in the console, click the **Refresh** button in the VPC networks toolbar. You should see your new network and subnet.

1.  You're not done configuring the network. Delete what you just created with the following command. 

```
terraform destroy -auto-approve
```
1.  Click **Refresh** again to verify the network was deleted.

## Review

At this point you have used Terraform to create a network and subnet, next you will create some firewall rules.

## Task 2. Use Terraform to create firewall rules

1.  In Cloud Shell, click on the **Open Editor** button. Open the `vpc-firewall-rules-public.tf` file in the `terraform-networks` folder. 

1. First, add a firewall rule that will allow SSH into machines in this network. Paste the code below.

```
# allow ssh
resource "google_compute_firewall" "public-allow-ssh" {
  name    = "${google_compute_network.public-vpc.name}-allow-ssh"
  network = google_compute_network.public-vpc.name
  allow {
    protocol = "tcp"
    ports    = ["22"]
  }
  source_ranges = [
    "0.0.0.0/0"
  ]
  target_tags = ["allow-ssh"] 
}
```

<aside><p><strong>Note: </strong>This rule allows SSH from everywhere, but only to machines that have the "allow-ssh" tag.</p></aside>

1.  Windows machines require RDP, not SSH. Add the following RDP rule below the SSH rule. 

```
# allow rdp
resource "google_compute_firewall" "public-allow-rdp" {
  name    = "${google_compute_network.public-vpc.name}-allow-rdp"
  network = google_compute_network.public-vpc.name
  allow {
    protocol = "tcp"
    ports    = ["3389"]
  }
  source_ranges = [
    "0.0.0.0/0"
  ]
  target_tags = ["allow-rdp"] 
}

```

1.  Ping is useful for testing. Add the following rule to enable it below the previous rules. 

```
# allow ping only from everywhere
resource "google_compute_firewall" "public-allow-ping" {
  name    = "${google_compute_network.public-vpc.name}-allow-ping"
  network = google_compute_network.public-vpc.name
  allow {
    protocol = "icmp"
  }
  source_ranges = [
    "0.0.0.0/0"
  ]
}
```

1.  As you did before, switch the the Terminal and run the following command to check for errors and see what will be created. 

```
terraform plan
```


1.  Run the following Terraform command and verify that the network, subnet and firewall rules are all being created using the Console.

```
terraform apply -auto-approve
```


## Review

Now you have a network and some firewall rules. Next you will add a test server to the network and see if the firewall rules work.

## Task 3. Use Terraform to create virtual machines

1.  Open the Cloud Shell code editor again.<p> Open the file `random-id-generator.tf` and add the following code. *This Terraform plug-in is used to generate a unique name for VMs added programatically*.</p>

```
# Terraform plugin for creating random ids
resource "random_id" "instance_id" {
 byte_length = 4
}
```

1.  Open the `test-server-linux.tf` file and add the following code to create a Virtual Machine in the public network.

```
# Create Test Server in Public VPC
resource "google_compute_instance" "test-server-linux" {
  name         = "public-test-server-linux-${random_id.instance_id.hex}"
  machine_type = "f1-micro"
  zone         = var.gcp_zone_1
  tags         = ["allow-ssh"]

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }

  metadata_startup_script = "sudo apt-get update;"

  network_interface {
    network        = google_compute_network.public-vpc.name
    subnetwork     = google_compute_subnetwork.public-subnet_1.name
    access_config { } 
  }
} 


output "test-server-linux" {
  value = google_compute_instance.test-server-linux.name
}

output "test-server-linux-external-ip" {
  value = google_compute_instance.test-server-linux.network_interface.0.access_config.0.nat_ip
}

output "test-server-linux-internal-ip" {
  value = google_compute_instance.test-server-linux.network_interface.0.network_ip
}
```

<aside><p><strong>Note: </strong>The output variables will return the name, and internal and external IP addresses for the machine created. Also, this machine is tagged with the "allow-ssh" tag, so you can connect to it. Lastly, take a look at the code in the network_interface section that configures this machine to be in the public network your created earlier.</p></aside>

1.  Run the Terraform plan and apply commands as you did earlier to create this machine. If an errors appears you may need to run terraform ini again before running the other two commands.

1. When the commands complete you should see the VM name, and internal and external IP addresses. From the Cloud Shell terminal make sure you can ping the external IP address of that machine. 

1.  In the Console, go to the Compute Engine service. You should see the VM you just created. Click the SSH button to make sure your firewall rule works. 

1.  Run the Terraform destroy command to delete everything you created thus far. 


## Task 4. Use Terraform to create a private network

1. Use the configuration of the Public network as a guide and create a second Private network. <p>In the `variables.tf` file, add a variable for the private subnet IP CIDR range, and set its value in the `terraform.tfvars` file. </p><p>Duplicate the `vpc-network-public.tf` file and change the names and variables appropriately. </p>

1.  Using the public firewall rules as a guide, add firewall rules for the private network. In the `source_ranges` section, don't allow traffic from all sources, only allow traffic from the public subnet IP CIDR range.

1.  Create a test server in the private network using the public one as a guide.

1.  Once you have everything created you should be able to SSH into the public test server. The server in the private network won't be accesible yet. You will fix that in the next lab.



<aside><p><strong>Congratulations!</strong>You have created a secure network infrastructure for your database migration projects. You created both public and private VPCs with appropriate firewall rules in each. You added virtual machines to each network and tested the commnication between them. You did all of this using Terraform. </p></aside>


![[/fragments/endqwiklab]]


![[/fragments/copyright]]
