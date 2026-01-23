---
tags:
  - tech
  - infrastructure
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Terraform

## Useful Commands

### State List/Show

[Command: state show - Terraform by HashiCorp](https://www.terraform.io/docs/cli/commands/state/show.html)

```jsx
terraform state list
terraform state show <resource>
ex:
terraform state show google_compute_instance.bastion_instance
```

Google Load Balancer

[用 GCP HTTP(S) Load Balancing 實現 HTTP Redirect HTTPS - Cloud Ace](https://blog.cloud-ace.tw/networking-website/load-balance/how-to-implement-http-redirect-https-through-gcp-https-load-balancing/)

[Terraform examples for external Application Load Balancers  |  Load Balancing  |  Google Cloud](https://cloud.google.com/load-balancing/docs/https/ext-http-lb-tf-module-examples)

[https://github.com/terraform-google-modules/terraform-google-lb-http](https://github.com/terraform-google-modules/terraform-google-lb-http)

[https://github.com/terraform-google-modules/terraform-google-lb-http/blob/HEAD/examples/https-redirect/main.tf](https://github.com/terraform-google-modules/terraform-google-lb-http/blob/HEAD/examples/https-redirect/main.tf)

## Compute Backend

[https://registry.terraform.io/providers/hashicorp/google/3.0.0-beta.1/docs/resources/compute_backend_service](https://registry.terraform.io/providers/hashicorp/google/3.0.0-beta.1/docs/resources/compute_backend_service)

VM

[https://github.com/terraform-google-modules/terraform-google-vm/tree/master/examples/mig/simple](https://github.com/terraform-google-modules/terraform-google-vm/tree/master/examples/mig/simple)

Instance Group Manager

[Updating instances in a GCP Managed Instance Group with Terraform](https://medium.com/@arnaldo.garat/updating-instances-in-a-gcp-managed-instance-group-with-terraform-d2b15a533499)

[https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_region_instance_group_manager](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_region_instance_group_manager)