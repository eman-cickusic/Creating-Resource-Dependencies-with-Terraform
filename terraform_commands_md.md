# Terraform commands cheatsheet for this project

## Initialization
```bash
# Initialize Terraform working directory
terraform init
```

## Planning
```bash
# Generate execution plan
terraform plan

# Save execution plan to file
terraform plan -out=tfplan
```

## Applying Changes
```bash
# Apply configuration
terraform apply

# Apply specific saved plan
terraform apply tfplan

# Auto-approve apply without confirmation prompt
terraform apply -auto-approve
```

## Destroying Resources
```bash
# Destroy all resources
terraform destroy

# Auto-approve destroy without confirmation prompt
terraform destroy -auto-approve
```

## State Management
```bash
# List resources in state
terraform state list

# Show details of a specific resource
terraform state show google_compute_instance.vm_instance

# Remove resource from state
terraform state rm google_compute_instance.vm_instance
```

## Dependency Visualization
```bash
# Generate dependency graph in DOT format
terraform graph

# Generate and save dependency graph as PNG (requires graphviz)
terraform graph | dot -Tpng > graph.png
```

## Variables
```bash
# Set variables via command line
terraform apply -var="instance_name=myvm" -var="instance_zone=us-central1-a"

# Use variable file
terraform apply -var-file="prod.tfvars"
```

## Validation
```bash
# Validate configuration syntax
terraform validate
```

## Formatting
```bash
# Format configuration files
terraform fmt
```

## Providers
```bash
# List providers used in configuration
terraform providers

# Update providers
terraform init -upgrade
```