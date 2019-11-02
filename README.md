# terraform-module-k3s

[![MIT Licensed](https://img.shields.io/badge/license-MIT-green.svg)](https://tldrlegal.com/license/mit-license)

Terraform module that creates a [k3s](https://k3s.io/) cluster with all node given nodes. Currently, it only applies the k3s installation script, without any complex settings nor HA cluster (may be in future releases).

## Usage

``` terraform
module "k3s" {
  source  = "xunleii/k3s/module"

  k3s_version = "v0.10.2"

  cluster_name = "my.k3s.local"
  cluster_cidr = "10.0.0.0/16"
  cluster_service_cidr = "10.1.0.0/16"
  additional_tls_san = ["k3s.my.domain.com"]
  
  master_node = {
      # This IP will be used as k3s master node IP.... if you want to use a public
      # address for the connection, use connection.host instead
      ip = "10.123.45.67"

      # Connection uses Terraform connection syntax
      connection = {
        host = "203.123.45.67"
        user = "ubuntu"
      }
  }
  minion_nodes = [
      {
          ip = "10.123.45.68"
          connection = {
              type = "ssh"
              user = "root"
              bastion_host = "10.123.45.67"
              bastion_user = "ubuntu"
          }
      },
      {
          ip = "10.123.45.69"
          connection = {
              type = "ssh"
              user = "root"
              bastion_host = "10.123.45.67"
              bastion_user = "ubuntu"
          }
      },
  ]
}
```

### Connection

The `connection` object can use all SSH [Terraform connection arguments](https://www.terraform.io/docs/provisioners/connection.html#argument-reference).

> Currently, only SSH type is allowed for this module (depends on SSH command to provide the `node-token` file from the master to a minion).  
> If you encounter problems during this step (you will see `[NOTE]` log prefix juste before), feel free to open an issue.  
> This will be resolved when a provider that can get file remotely will be available.

### Kubeconfig

Because Terraform doesn't allow us to get file remotely, you need to get it manually (with `external` data for example).

## Known issues

### Removing a minion node(s)

Currently, you can't removing a minion directly from the list module for the following reasons:

- **K3s side**: The node is not drained, so all workflow on it will be unvailable
- **K3s side**: If we reload `k3s_minions_installer` a removed node, the node will be seen as `NotReady`
- **Terraform side**: If you not remove the last one, Terraform will try to recreate all nodes after the deleted
  - This is due to the `triggers`: because the position in the list will changed, the `IP` of the minion will changed too, activating the trigger.
- **Terraform side**: If all nodes are removed, Terraform will crashed

## License

terraform-module-k3s is released under the MIT License. See the bundled LICENSE file for details.
