Terraform Resource Dependencies in GCP
This repository demonstrates how to create resource dependencies with Terraform in Google Cloud Platform (GCP). The project covers both implicit and explicit dependencies between resources.

Overview
In this project, we create two VM instances in the default network:

The first VM instance with a static IP address (demonstrating implicit dependency)
The second VM instance with a dependency on a Cloud Storage bucket (demonstrating explicit dependency)
We use variables to define VM attributes at runtime and output values to display resource attributes.

Prerequisites
Google Cloud Platform account
Terraform installed (v1.0.0+)
Google Cloud SDK installed
Project Structure
terraform-resource-dependencies/
├── provider.tf    # GCP provider configuration
├── variables.tf   # Variable definitions
├── instance.tf    # VM instance with implicit dependency (static IP)
├── exp.tf         # VM instance with explicit dependency (storage bucket)
├── outputs.tf     # Output definitions
└── README.md      # This file
Files Description
provider.tf
Configures Google Cloud as the provider, including project ID, region, and zone.

terraform
provider "google" {
  project = "PROJECT_ID"  # Replace with your actual GCP project ID
  region  = "REGION"      # Replace with your preferred region
  zone    = "ZONE"        # Replace with your preferred zone
}
variables.tf
Defines variables for the VM instance configuration:

instance_name: Name of the Google Compute instance
instance_zone: Zone for the Google Compute instance
instance_type: Machine type of the Google Compute instance (default: e2-medium)
terraform
variable "instance_name" {
  type        = string
  description = "Name for the Google Compute instance"
}

variable "instance_zone" {
  type        = string
  description = "Zone for the Google Compute instance"
}

variable "instance_type" {
  type        = string
  description = "Disk type of the Google Compute instance"
  default     = "e2-medium"
}
instance.tf
Creates a VM instance and assigns a static IP address to it. This demonstrates an implicit dependency, where Terraform automatically identifies that the VM depends on the static IP address.

terraform
resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}

resource "google_compute_instance" "vm_instance" {
  name         = "${var.instance_name}"
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  
  network_interface {
    network = "default"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
}
exp.tf
Creates a VM instance with an explicit dependency on a Cloud Storage bucket using the depends_on attribute. This demonstrates how to handle dependencies that are not automatically detected by Terraform.

terraform
resource "google_compute_instance" "another_instance" {
  name         = "terraform-instance-2"
  machine_type = "e2-micro"
  
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  
  network_interface {
    network = "default"
    access_config {
    }
  }
  
  # Tells Terraform that this VM instance must be created only after the
  # storage bucket has been created.
  depends_on = [google_storage_bucket.example_bucket]
}

# New resource for the storage bucket our application will use.
resource "google_storage_bucket" "example_bucket" {
  name     = "UNIQUE-BUCKET-NAME"  # Replace with a globally unique bucket name
  location = "US"
  
  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }
}
outputs.tf
Defines output values to display after resource creation:

network_IP: The internal IP address of the VM instance
instance_link: The URI of the created VM instance
terraform
output "network_IP" {
  value       = google_compute_instance.vm_instance.instance_id
  description = "The internal ip address of the instance"
}

output "instance_link" {
  value       = google_compute_instance.vm_instance.self_link
  description = "The URI of the created resource."
}
Usage
Clone this repository:
bash
git clone https://github.com/YOUR-USERNAME/terraform-resource-dependencies.git
cd terraform-resource-dependencies
Update the configuration:
In provider.tf: Replace placeholders with your GCP project ID, region, and zone
In exp.tf: Replace UNIQUE-BUCKET-NAME with a globally unique name for your GCS bucket
Initialize Terraform:
bash
terraform init
Review the execution plan:
bash
terraform plan
When prompted, enter values for instance_name and instance_zone.

Apply the configuration:
bash
terraform apply
When prompted, enter values for instance_name and instance_zone, then confirm by typing yes.

Verify resource creation in the Google Cloud Console:
Navigate to Compute Engine > VM instances to view the created instances
Navigate to VPC network > IP addresses > External IP addresses to view the static IP
Navigate to Cloud Storage > Buckets to view the created bucket
View the dependency graph:
bash
terraform graph | dot -Tpng > graph.png
Note: This requires the graphviz package to be installed.

Understanding Dependencies
Implicit Dependencies
In the instance.tf file, the VM instance has an implicit dependency on the static IP address. Terraform automatically determines this dependency because the VM's network interface refers to the static IP's address property.

When applying the configuration, Terraform creates the static IP before creating the VM instance, regardless of the order in which they are defined in the configuration.

Explicit Dependencies
In the exp.tf file, the VM instance has an explicit dependency on the Cloud Storage bucket, defined using the depends_on attribute. This tells Terraform that the VM instance should only be created after the bucket is created, even though there's no reference to the bucket in the VM configuration.

This type of dependency is useful when resources have a relationship that's not expressed in the configuration itself, such as when an application running on the VM needs to access the bucket.

Cleaning Up
To avoid incurring charges for the created resources, destroy them when they're no longer needed:

bash
terraform destroy
When prompted, confirm by typing yes.

License
This project is licensed under the MIT License - see the LICENSE file for details.

