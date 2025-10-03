`# Terraform-google-template-instance
# Terraform Google Cloud Template-Instance Module

## Table of Contents

- [Introduction](#introduction)
- [Usage](#usage)
- [Examples](#examples)
- [License](#license)
- [Author](#author)
- [Inputs](#inputs)
- [Outputs](#outputs)


## Introduction

This Terraform module provides infrastructure configuration for creating a Template-Instance in Google Cloud Platform (GCP) along with vpc subnets, firewall rules . It's designed to be used for managing network resources in a GCP environment.


## Usage

To use this module, include it in your Terraform configuration. Below is an example of how to call the template-instance module and its dependencies.
### Examples

## Example: simple
```hcl
module "simple_template" {
  source               = "git@github.com:pankajyadavdevops/terraform-google-template-instance.git"
  version              = "1.0.2"
  name                 = "template"
  environment          = "test"
  stack_type           = "IPV4_ONLY"
  region               = "asia-northeast1"
  source_image         = "ubuntu-2204-jammy-v20230908"
  source_image_family  = "ubuntu-2204-lts"
  source_image_project = "ubuntu-os-cloud"
  disk_size_gb         = "20"
  subnetwork           = module.subnet.subnet_id
  instance_template    = true
  service_account      = null
  enable_public_ip     = true ## public IP if enable_public_ip is true
  metadata = {
    ssh-keys = <<EOF
        dev:ssh-rsa AAAAB3NzaC1yc2EAA/3mwt2y+PDQMU= vinod@vinod
      EOF
  }
}
```

## Example: alias_ip_range
```hcl
module "instance_template" {
  source            = "git@github.com:pankajyadavdevops/terraform-google-template-instance.git"
  version           = "1.0.2"
  name              = "alias-ip-range"
  environment       = "test"
  instance_template = true
  subnetwork        = module.subnet.subnet_id

  service_account = {
    email  = module.service-account.account_email
    scopes = ["https://www.googleapis.com/auth/cloud-platform"] # Example scopes
  }

  alias_ip_range = {
    ip_cidr_range         = "/24"
    subnetwork_range_name = module.subnet.subnet_name
  }
}
```

## Example: additional_disks
```hcl
module "instance_template" {
  source            = "git@github.com:pankajyadavdevops/terraform-google-template-instance.git"
  version           = "1.0.2"
  name              = "additional-disks"
  environment       = "test"
  subnetwork        = module.subnet.subnet_id
  instance_template = true
  service_account   = null
  additional_disks = [
    {
      disk_name    = "disk-0"
      device_name  = "disk-0"
      disk_size_gb = 10
      disk_type    = "pd-standard"
      auto_delete  = "true"
      boot         = "false"
      disk_labels  = {}
    },
    {
      disk_name    = "disk-1"
      device_name  = "disk-1"
      disk_size_gb = 10
      disk_type    = "pd-standard"
      auto_delete  = "true"
      boot         = "false"
      disk_labels  = { "foo" : "bar" }
    },
    {
      disk_name    = "disk-2"
      device_name  = "disk-2"
      disk_size_gb = 10
      disk_type    = "pd-standard"
      auto_delete  = "true"
      boot         = "false"
      disk_labels  = { "foo" : "bar" }
    },
  ]
}
```
This example demonstrates how to create various GCP resources using the provided modules. Adjust the input values to suit your specific requirements.

## Examples
For detailed examples on how to use this module, please refer to the [Examples](https://github.com/pankajyadavdevops/terraform-google-template-instance/tree/master/examples) directory within this repository.

## License
This Terraform module is provided under the **MIT** License. Please see the [LICENSE](https://github.com/pankajyadavdevops/terraform-google-template-instance/blob/master/LICENSE) file for more details.

## Author
Your Name
Replace **MIT** and **pankajyadavdevops** with the appropriate license and your information. Feel free to expand this README with additional details or usage instructions as needed for your specific use case.

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >=1.9.5 |
| <a name="requirement_google"></a> [google](#requirement\_google) | >=6.1.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_google"></a> [google](#provider\_google) | >=6.1.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_labels"></a> [labels](#module\_labels) | git@github.com:pankajyadavdevops/terraform-google-labels.git?ref=v1.0.2 | 1.0.2 |

## Resources

| Name | Type |
|------|------|
| [google_compute_instance_template.tpl](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance_template) | resource |
| [google_client_config.current](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/client_config) | data source |
| [google_compute_zones.available](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/compute_zones) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_additional_disks"></a> [additional\_disks](#input\_additional\_disks) | List of maps of additional disks. See https://www.terraform.io/docs/providers/google/r/compute_instance_template#disk_name | <pre>list(object({<br>    disk_name    = string<br>    device_name  = string<br>    auto_delete  = bool<br>    boot         = bool<br>    disk_size_gb = number<br>    disk_type    = string<br>    disk_labels  = map(string)<br>  }))</pre> | `[]` | no |
| <a name="input_additional_networks"></a> [additional\_networks](#input\_additional\_networks) | Additional network interface details for GCE, if any. | <pre>list(object({<br>    network            = string<br>    subnetwork         = string<br>    subnetwork_project = string<br>    network_ip         = string<br>    access_config = list(object({<br>      nat_ip       = string<br>      network_tier = string<br>    }))<br>    ipv6_access_config = list(object({<br>      network_tier = string<br>    }))<br>  }))</pre> | `[]` | no |
| <a name="input_alias_ip_range"></a> [alias\_ip\_range](#input\_alias\_ip\_range) | An array of alias IP ranges for this network interface. Can only be specified for network interfaces on subnet-mode networks.<br>ip\_cidr\_range: The IP CIDR range represented by this alias IP range. This IP CIDR range must belong to the specified subnetwork and cannot contain IP addresses reserved by system or used by other network interfaces. At the time of writing only a netmask (e.g. /24) may be supplied, with a CIDR format resulting in an API error.<br>subnetwork\_range\_name: The subnetwork secondary range name specifying the secondary range from which to allocate the IP CIDR range for this alias IP range. If left unspecified, the primary range of the subnetwork will be used. | <pre>object({<br>    ip_cidr_range         = string<br>    subnetwork_range_name = string<br>  })</pre> | `null` | no |
| <a name="input_auto_delete"></a> [auto\_delete](#input\_auto\_delete) | Whether or not the boot disk should be auto-deleted | `string` | `"true"` | no |
| <a name="input_automatic_restart"></a> [automatic\_restart](#input\_automatic\_restart) | (Optional) Specifies whether the instance should be automatically restarted if it is terminated by Compute Engine (not terminated by a user). | `bool` | `true` | no |
| <a name="input_can_ip_forward"></a> [can\_ip\_forward](#input\_can\_ip\_forward) | Enable IP forwarding, for NAT instances for example | `string` | `"false"` | no |
| <a name="input_confidential_instance_type"></a> [confidential\_instance\_type](#input\_confidential\_instance\_type) | Defines the confidential computing technology the instance uses. If this is set to "SEV\_SNP", var.min\_cpu\_platform will be automatically set to "AMD Milan". See https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance#confidential_instance_type. | `string` | `null` | no |
| <a name="input_description"></a> [description](#input\_description) | The template's description | `string` | `""` | no |
| <a name="input_disk_encryption_key"></a> [disk\_encryption\_key](#input\_disk\_encryption\_key) | The id of the encryption key that is stored in Google Cloud KMS to use to encrypt all the disks on this instance | `string` | `null` | no |
| <a name="input_disk_labels"></a> [disk\_labels](#input\_disk\_labels) | Labels to be assigned to boot disk, provided as a map | `map(string)` | `{}` | no |
| <a name="input_disk_size_gb"></a> [disk\_size\_gb](#input\_disk\_size\_gb) | Boot disk size in GB | `string` | `"20"` | no |
| <a name="input_disk_type"></a> [disk\_type](#input\_disk\_type) | Boot disk type, can be either pd-ssd, local-ssd, or pd-standard | `string` | `""` | no |
| <a name="input_enable_confidential_vm"></a> [enable\_confidential\_vm](#input\_enable\_confidential\_vm) | Whether to enable the Confidential VM configuration on the instance. Note that the instance image must support Confidential VMs. See https://cloud.google.com/compute/docs/images | `bool` | `false` | no |
| <a name="input_enable_nested_virtualization"></a> [enable\_nested\_virtualization](#input\_enable\_nested\_virtualization) | Defines whether the instance should have nested virtualization enabled. | `bool` | `false` | no |
| <a name="input_enable_public_ip"></a> [enable\_public\_ip](#input\_enable\_public\_ip) | public IP if enable\_public\_ip is true for the instance. | `bool` | `false` | no |
| <a name="input_enable_shielded_vm"></a> [enable\_shielded\_vm](#input\_enable\_shielded\_vm) | Whether to enable the Shielded VM configuration on the instance. Note that the instance image must support Shielded VMs. See https://cloud.google.com/compute/docs/images | `bool` | `false` | no |
| <a name="input_environment"></a> [environment](#input\_environment) | Environment (e.g. `prod`, `dev`, `staging`). | `string` | `""` | no |
| <a name="input_extra_tags"></a> [extra\_tags](#input\_extra\_tags) | Additional tags for the resource. | `map(string)` | `{}` | no |
| <a name="input_gpu"></a> [gpu](#input\_gpu) | GPU information. Type and count of GPU to attach to the instance template. See https://cloud.google.com/compute/docs/gpus more details | <pre>object({<br>    type  = string<br>    count = number<br>  })</pre> | `null` | no |
| <a name="input_instance_description"></a> [instance\_description](#input\_instance\_description) | Description of the generated instances | `string` | `""` | no |
| <a name="input_instance_template"></a> [instance\_template](#input\_instance\_template) | Instance template self\_link used to create compute instances | `bool` | `false` | no |
| <a name="input_ipv6_access_config"></a> [ipv6\_access\_config](#input\_ipv6\_access\_config) | IPv6 access configurations. Currently a max of 1 IPv6 access configuration is supported. If not specified, the instance will have no external IPv6 Internet access. | <pre>list(object({<br>    network_tier = string<br>  }))</pre> | `[]` | no |
| <a name="input_label_order"></a> [label\_order](#input\_label\_order) | Label order, e.g. sequence of application name and environment `name`,`environment`,'attribute' [`webserver`,`qa`,`devops`,`public`,] . | `list(any)` | <pre>[<br>  "name",<br>  "environment"<br>]</pre> | no |
| <a name="input_labels"></a> [labels](#input\_labels) | Labels, provided as a map | `map(string)` | `{}` | no |
| <a name="input_machine_type"></a> [machine\_type](#input\_machine\_type) | Machine type to create, e.g. n1-standard-1 | `string` | `"e2-small"` | no |
| <a name="input_maintenance_interval"></a> [maintenance\_interval](#input\_maintenance\_interval) | Specifies the frequency of planned maintenance events | `string` | `null` | no |
| <a name="input_managedby"></a> [managedby](#input\_managedby) | ManagedBy, eg 'pankajyadavdevops'. | `string` | `"ipankajyadavdevops"` | no |
| <a name="input_metadata"></a> [metadata](#input\_metadata) | Metadata, provided as a map | `map(string)` | `{}` | no |
| <a name="input_min_cpu_platform"></a> [min\_cpu\_platform](#input\_min\_cpu\_platform) | Specifies a minimum CPU platform. Applicable values are the friendly names of CPU platforms, such as Intel Haswell or Intel Skylake. See the complete list: https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform | `string` | `null` | no |
| <a name="input_name"></a> [name](#input\_name) | Name of the resource. Provided by the client when the resource is created. | `string` | `"test"` | no |
| <a name="input_network"></a> [network](#input\_network) | The name or self\_link of the network to attach this interface to. Use network attribute for Legacy or Auto subnetted networks and subnetwork for custom subnetted networks. | `string` | `""` | no |
| <a name="input_network_ip"></a> [network\_ip](#input\_network\_ip) | Private IP address to assign to the instance if desired. | `string` | `""` | no |
| <a name="input_nic_type"></a> [nic\_type](#input\_nic\_type) | Valid values are "VIRTIO\_NET", "GVNIC" or set to null to accept API default behavior. | `string` | `null` | no |
| <a name="input_on_host_maintenance"></a> [on\_host\_maintenance](#input\_on\_host\_maintenance) | Instance availability Policy | `string` | `"MIGRATE"` | no |
| <a name="input_preemptible"></a> [preemptible](#input\_preemptible) | Allow the instance to be preempted | `bool` | `false` | no |
| <a name="input_region"></a> [region](#input\_region) | Region where the instance template should be created. | `string` | `null` | no |
| <a name="input_repository"></a> [repository](#input\_repository) | Terraform current module repo | `string` | `"https://github.com/pankajyadavdevops/terraform-google-template-instance"` | no |
| <a name="input_resource_policies"></a> [resource\_policies](#input\_resource\_policies) | (Optional) A list of short names or self\_links of resource policies to attach to the instance. Modifying this list will cause the instance to recreate. Currently a max of 1 resource policy is supported. | `list(string)` | `[]` | no |
| <a name="input_service_account"></a> [service\_account](#input\_service\_account) | Service account to attach to the instance. See https://www.terraform.io/docs/providers/google/r/compute_instance_template#service_account. | <pre>object({<br>    email  = string<br>    scopes = set(string)<br>  })</pre> | n/a | yes |
| <a name="input_shielded_instance_config"></a> [shielded\_instance\_config](#input\_shielded\_instance\_config) | Not used unless enable\_shielded\_vm is true. Shielded VM configuration for the instance. | <pre>object({<br>    enable_secure_boot          = bool<br>    enable_vtpm                 = bool<br>    enable_integrity_monitoring = bool<br>  })</pre> | <pre>{<br>  "enable_integrity_monitoring": true,<br>  "enable_secure_boot": true,<br>  "enable_vtpm": true<br>}</pre> | no |
| <a name="input_source_image"></a> [source\_image](#input\_source\_image) | Source disk image. If neither source\_image nor source\_image\_family is specified, defaults to the latest public CentOS image. | `string` | `""` | no |
| <a name="input_source_image_family"></a> [source\_image\_family](#input\_source\_image\_family) | Source image family. If neither source\_image nor source\_image\_family is specified, defaults to the latest public CentOS image. | `string` | `""` | no |
| <a name="input_source_image_project"></a> [source\_image\_project](#input\_source\_image\_project) | Project where the source image comes from. The default project contains CentOS images. | `string` | `""` | no |
| <a name="input_spot"></a> [spot](#input\_spot) | Provision a SPOT instance | `bool` | `false` | no |
| <a name="input_spot_instance_termination_action"></a> [spot\_instance\_termination\_action](#input\_spot\_instance\_termination\_action) | Action to take when Compute Engine preempts a Spot VM. | `string` | `"STOP"` | no |
| <a name="input_stack_type"></a> [stack\_type](#input\_stack\_type) | The stack type for this network interface to identify whether the IPv6 feature is enabled or not. Values are `IPV4_IPV6` or `IPV4_ONLY`. Default behavior is equivalent to IPV4\_ONLY. | `string` | `null` | no |
| <a name="input_startup_script"></a> [startup\_script](#input\_startup\_script) | User startup script to run when instances spin up | `string` | `""` | no |
| <a name="input_subnetwork"></a> [subnetwork](#input\_subnetwork) | The name of the subnetwork to attach this interface to. The subnetwork must exist in the same region this instance will be created in. Either network or subnetwork must be provided. | `string` | `""` | no |
| <a name="input_subnetwork_project"></a> [subnetwork\_project](#input\_subnetwork\_project) | The ID of the project in which the subnetwork belongs. If it is not provided, the provider project is used. | `string` | `""` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | Network tags, provided as a list | `list(string)` | `[]` | no |
| <a name="input_threads_per_core"></a> [threads\_per\_core](#input\_threads\_per\_core) | The number of threads per physical core. To disable simultaneous multithreading (SMT) set this to 1. | `number` | `null` | no |
| <a name="input_total_egress_bandwidth_tier"></a> [total\_egress\_bandwidth\_tier](#input\_total\_egress\_bandwidth\_tier) | Egress bandwidth tier setting for supported VM families | `string` | `"DEFAULT"` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_available_zones"></a> [available\_zones](#output\_available\_zones) | List of available zones in region |
| <a name="output_id"></a> [id](#output\_id) | An identifier for the resource with format |
| <a name="output_metadata_fingerprint"></a> [metadata\_fingerprint](#output\_metadata\_fingerprint) | An identifier for the resource with format |
| <a name="output_name"></a> [name](#output\_name) | An identifier for the resource with format |
| <a name="output_self_link"></a> [self\_link](#output\_self\_link) | An identifier for the resource with format |
| <a name="output_self_link_unique"></a> [self\_link\_unique](#output\_self\_link\_unique) | A special URI of the created resource that uniquely identifies this instance template with the following format: |
| <a name="output_tags_fingerprint"></a> [tags\_fingerprint](#output\_tags\_fingerprint) | The unique fingerprint of the tags. |
<!-- END_TF_DOCS -->