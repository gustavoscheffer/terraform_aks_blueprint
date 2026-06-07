

# Terraform AKS Blueprint

Repository: https://github.com/gustavoscheffer/terraform_aks_blueprint

Terraform configuration to create the `mecha-buda` AKS environment.

## Project layout

- `main.tf` — resource definitions: resource group, ACR, AKS cluster
- `variables.tf` — input variables used by the configuration
- `output.tf` — sensitive outputs (kube config / client cert)
- `terraform.tfvars` — (optional) local overrides for variables

## What this configuration creates

- Resource Group: `mecha-buda-rg-<environment>`
- ACR: `mechabudaacr<environment>` (sku: Basic)
- AKS Cluster: `mecha-buda-aks-<environment>` with:
	- `azure` network plugin (overlay)
	- system-assigned identity
	- default node pool `system` (1 x Standard_D2_v2)

## Variables (from `variables.tf`)

- `environment` (string): target environment name (used in resource names)
- `location` (string): Azure region for resources

Set values via a `terraform.tfvars` file or via CLI (`-var='environment=dev' -var='location=westeurope'`).

## Outputs (from `output.tf`)

- `client_certificate` (sensitive): client certificate from AKS kube config
- `kube_config` (sensitive): raw kubeconfig for the AKS cluster

Note: both outputs are marked sensitive. Avoid printing them in shared logs.

## Quick start

```bash
terraform init
terraform plan -out=tfplan -var='environment=dev' -var='location=westeurope'
terraform apply "tfplan"
```

After apply you can retrieve the kubeconfig with the Terraform output (sensitive) or use `az aks get-credentials`:

```bash
az aks get-credentials --resource-group mecha-buda-rg-dev --name mecha-buda-aks-dev
kubectl get nodes
```

Replace `dev` with your chosen `environment` value.

## Notes and recommendations

- Protect sensitive outputs and avoid committing secrets.
- Use a remote backend (Azure Storage) for Terraform state with locking in team environments.
- Consider increasing the `node_count` and choosing appropriate `vm_size` for production workloads.
- Add logging/monitoring addons (Azure Monitor) and RBAC/AAD integration for production.

## Cleanup

Destroy all resources when no longer needed:

```bash
terraform destroy -var='environment=dev' -var='location=westeurope'
```

