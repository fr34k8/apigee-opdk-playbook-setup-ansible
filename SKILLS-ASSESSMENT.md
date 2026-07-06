> Infrastructure-as-code automation for enterprise API gateway platforms — specifically Ansible-based orchestration of Apigee Edge Private Cloud (OPDK) across the full deployment lifecycle: provisioning, configuration, installation, expansion, upgrade, and disaster recovery.

## Why this project is notable

This is the orchestrator playbook at the center of a **30+ role ecosystem** that automates every phase of Apigee Private Cloud — one of the most complex on-premises API management platforms available. Deploying Apigee OPDK manually requires coordinating OS hardening, port matrices, Java runtimes, Cassandra clusters, ZooKeeper ensembles, multi-region data replication, and a specific startup ordering across dozens of nodes. This playbook encodes all of that institutional knowledge into reproducible, tagged Ansible plays that can install an entire planet from bare metal, expand it across regions, validate it, and recover it from failure — each phase executable independently via `--tags`.

## Expertise demonstrated

| Expertise Area | Evidence |
|---|---|
| **Ansible role architecture & composition** | Decomposed a multi-product private cloud into ~30+ composable Galaxy roles with consistent tagging (`os`, `ds`, `ms`, `rmp`, `qpid`, `pg`, `org`, `expansion`, etc.) enabling surgical partial execution across hundreds of plays |
| **Multi-region distributed systems orchestration** | Multi-node `install.yml` manages serial data-store bootstrapping, free-strategy Router/MP fan-out, Postgres master-standby promotion, and cross-region Cassandra registration — all with correct ordering constraints |
| **Air-gapped / offline deployment** | Full offline workflow: local mirror creation (nginx), package archive assembly, offline Ansible controller bootstrap, and response file generation without external network access |
| **Disaster recovery & operational automation** | Playbooks for Cassandra dead-node recovery, planet-wide component restart, backup/restore, environment removal, virtual host management, analytics scope updates, and user account unlock |
| **Infrastructure provisioning (GCE + AWS)** | Terraform modules for 5-node, AIO, and multi-DC GCE topologies; Ansible playbooks for AWS security groups and EC2 instance management |
| **Idempotent configuration management** | Property-driven (`~/.apigee/`, `~/.apigee-secure/`) silent-installation config generation that adapts per-node and per-region, with caching layers to avoid redundant re-computation |
| **Bastion host & SSH tunneling** | ProxyCommand configuration, bastion host setup with nginx reverse proxy and Let's Encrypt TLS termination, and SSH key deployment for secure multi-hop access |

## How this shows the expertise

- **Role composition over monolithic scripts**: The `requirements.yml` files pull individual `apigee-opdk-*` roles that are developed, versioned, and tested independently. The playbook layer composes them — the same `apigee-opdk-setup-component` role handles `ds`, `ms`, `r`, `mp`, `qs`, and `ps` profiles depending on inventory group membership and tags.

- **Correct startup ordering for distributed state**: The `install.yml` playbooks use `serial: 1` for ZooKeeper/Cassandra/DataStore nodes and Postgres master/standby, while using `strategy: free` for stateless Router and Message Processor fan-out. This reflects deep understanding of distributed consensus bootstrapping.

- **Tag-driven partial execution**: Every role is tagged with the phase it belongs to (`os`, `bootstrap`, `common`, `config`, `ds`, `ms`, `rmp`, `org`, `validate`). Operators can run `--tags=ds` to reinstall only the data-store tier without touching anything else — critical for production troubleshooting.

- **Property layering for multi-environment parity**: Runtime attributes are loaded from `~/.apigee/custom-properties.yml` and `~/.apigee-secure/credentials.yml`, enabling the same playbook to target dev, staging, and production planets by swapping property files rather than rewriting plays.

- **Full lifecycle coverage**: The same framework handles initial installation, horizontal scaling (add Router/MP/Qpid nodes), vertical scaling (add Postgres standbys), geographic expansion (add data centers), in-place upgrades, and decommissioning — all using the same role set and property conventions.

## Related expertise

| Repo | Domain | Relationship |
|---|---|---|
| [apigee-opdk-modules](https://github.com/carlosfrias/apigee-opdk-modules) | Ansible module library | Core custom modules used by every playbook in this framework |
| [apigee-opdk-setup-silent-installation-config](https://github.com/carlosfrias/apigee-opdk-setup-silent-installation-config) | Configuration generation | Generates per-node response files from property layers |
| [apigee-opdk-setup-ansible-controller](https://github.com/carlosfrias/apigee-opdk-setup-ansible-controller) | Controller bootstrap | Role consumed by `setup/setup.yml` |
| [apigee-opdk-playbook-installation-single-region](https://github.com/carlosfrias/apigee-opdk-playbook-installation-single-region) | Single-region install | Alternative topology playbook for single-region planets |
| [apigee-opdk-playbook-installation-aio](https://github.com/carlosfrias/apigee-opdk-playbook-installation-aio) | AIO install | Standalone all-in-one playbook |
| [apigee-opdk-cassandra-rebuild](https://github.com/carlosfrias/apigee-opdk-cassandra-rebuild) | Cassandra recovery | Role for rebuilding dead Cassandra nodes |
| [apigee-opdk-playbook-maintenance-cassandra](https://github.com/carlosfrias/apigee-opdk-playbook-maintenance-cassandra) | Cassandra maintenance | Maintenance playbook for repair, replication factor changes, and validation |
| [apigee-opdk-setup-mirror-nginx](https://github.com/carlosfrias/apigee-opdk-setup-mirror-nginx) | Offline mirror | Role for creating nginx-based local package mirrors |
| [apigee-ansible-configurations](https://github.com/carlosfrias/apigee-ansible-configurations) | Ansible config samples | Reference inventory and configuration examples |

## Provenance

Originally authored by Carlos Frias during tenure on the Apigee Edge Private Cloud team at Google. Forked from the `apigee/ansible-opdk-accelerator` open-source release and extended with additional infrastructure, offline, GCE, and operational playbooks.

<!-- BEGIN Google Required Disclaimer -->
This is not an officially supported Google product.
<!-- END Google Required Disclaimer -->

## License

Apache License 2.0 — see [LICENSE](./LICENSE).