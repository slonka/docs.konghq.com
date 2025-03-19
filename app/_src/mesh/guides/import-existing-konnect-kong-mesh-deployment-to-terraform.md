---
title: Import existing Konnect {{site.mesh_product_name}} deployment to Terraform
---

## Introduction

This guide explains how to import an existing Konnect {{site.mesh_product_name}} deployment into Terraform.
It covers setup, importing resources, running Terraform commands, and automating the import process using a provided script.

## Setup

Make sure you completed "Setup" from [Deploy {{site.mesh_product_name}} using Terraform and Konnect](/mesh/{{page.release}}/guides/deploy-kong-mesh-using-terraform-and-konnect).

## Import existing Konnect {{site.mesh_product_name}} deployment to Terraform

In order to import an existing Konnect {{site.mesh_product_name}} deployment to Terraform, you need to add each resource name and ID to the Terraform configuration file.

Below is an example of how to import:
- Mesh Control Plane
- Mesh called `another-name`
- MeshTrafficPermission named `allow-all`
- MeshTrafficPermission named `example-with-tags`

```hcl
import {
  provider = "konnect-beta"
  to = konnect_mesh_control_plane.my_meshcontrolplane
  id = "c9fd8f76-6460-45fb-9a64-a981d8a512d7"
}

import {
    provider = "konnect-beta"
    to = konnect_mesh.another-name
    id = "{ \"cp_id\": \"c9fd8f76-6460-45fb-9a64-a981d8a512d7\", \"name\": \"another-name\"}"
}

import {
  provider = "konnect-beta"
  to = konnect_mesh_traffic_permission.allow-all
  id = "{ \"cp_id\": \"c9fd8f76-6460-45fb-9a64-a981d8a512d7\", \"mesh\": \"another-name\", \"name\": \"allow-all\" }"
}

import {
  provider = "konnect-beta"
  to = konnect_mesh_traffic_permission.example-with-tags
  id = "{ \"cp_id\": \"c9fd8f76-6460-45fb-9a64-a981d8a512d7\", \"mesh\": \"another-name\", \"name\": \"example-with-tags\" }"
}
```

You can find the names and IDs of the resources by navigating the [Mesh Manager UI](https://cloud.konghq.com/us/mesh-manager).
The `id` field in the import block is a JSON string that contains the necessary information to identify the resource.

Next run:

```bash
terraform plan -generate-config-out="generated_resources.tf"
```

Next review the "generated_resources.tf" file and make sure the resources are imported correctly.
Add all the necessary references like `cp_id`, `depends_on`.
After that you can run `terraform apply` to import the resources.

## Automating the import process

{{site.mesh_product_name}} API provides an endpoint to list all resources of a certain type in a specific Mesh Control Plane.
You can use this endpoint to automate the import process.
Below is an example of how to list all `HostnameGenerators` in a Mesh called `another-name` for a Control Plane with id `c9fd8f76-6460-45fb-9a64-a981d8a512d7`:

```bash
curl -s https://us.api.konghq.com/v1/mesh/control-planes/c9fd8f76-6460-45fb-9a64-a981d8a512d7/api/meshes/another-name/hostnamegenerators
```

[Here is a script](https://gist.github.com/slonka/4f53a380acdb1c882d34d168ccf1d526) that automates the import process for all resources in a Mesh Control Plane.
It takes the Control Plane ID and region as arguments and uses the [KPAT](/konnect/org-management/access-tokens/) environment variable to authenticate with the Konnect API.
The script is provided as best effort and may need to be adjusted.

Example invocation:

```bash
./import.sh c9fd8f76-6460-45fb-9a64-a981d8a512d7 us
```

## Next steps

Explore all policies that are available in the [{{site.mesh_product_name}} Policy Catalog](/mesh/{{page.release}}/policies/introduction).
