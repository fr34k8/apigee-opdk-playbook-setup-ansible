> Ansible playbook framework for automating the full lifecycle of Apigee Edge Private Cloud (OPDK) — from OS prerequisites and controller setup through multi-node planet installation, expansion, upgrade, and disaster recovery.

> [!NOTE]
> For expertise demonstrated, capabilities analysis, and related projects, see [SKILLS-ASSESSMENT.md](./SKILLS-ASSESSMENT.md).

## What the playbook actually does

This repository is the top-level orchestrator for the `apigee-opdk-*` Ansible role ecosystem. It composes ~30+ Galaxy roles into runnable playbooks that cover every phase of an Apigee Private Cloud deployment:
> [!NOTE]
> Engineering portfolio note — this project demonstrates Apigee OPDK lifecycle orchestration. See the [skills assessment →](SKILLS-ASSESSMENT.md) for the expertise applied.

1. **Controller setup** (`setup/`) — Bootstraps the Ansible control node: installs Galaxy role dependencies, creates the `~/.ansible`, `~/.apigee`, `~/.apigee-secure`, and `~/.apigee-workspace` directory conventions, and configures `ansible.cfg` with inventory and SSH defaults.

2. **Infrastructure** (`infrastructure/`) — Prepares target environments:
   - **SSH access** — Configure key-based SSH login and bastion host proxy commands.
   - **Offline / air-gapped installs** — Mirror creation (`mirror/`), offline package download, and offline Ansible controller setup.
   - **Response file generation** — Dynamically generate Apigee silent-installation config files from host properties.
   - **Port validation** — Verify required ports are open across the planet.
   - **GCE provisioning** — Terraform modules and Ansible playbooks for spinning up Apigee topologies on Google Compute Engine (AIO, 5-node, multi-data-center).
   - **Backup** — Back up and restore the Ansible controller configuration.

3. **Installations** (`installations/`) — Deploy Apigee components by topology:
   - **AIO** — All-in-one single-node installation.
   - **Multi-node / multi-region** — Full HA planet installation with tagged plays for OS prep (`os`), bootstrap, Cassandra/ZooKeeper (`ds`), Management Server (`ms`), Router (`r`), Message Processor (`mp`), Qpid (`qpid`), Postgres master/standby (`pg`/`pgmaster`/`pgstandby`), and org onboarding (`org`).
   - **Edge Microgateway** — Microgateway-specific installation.
   - **Developer Portal** — Drupal-based portal installation.
   - Every installation supports `--tags` for partial execution (e.g., `--tags=os` for OS-only, `--tags=ds` for data-store only).

4. **Post-installation** (`post-installation/`) — Ongoing operations:
   - **Add components** — Scale routers, message processors, Postgres standbys.
   - **Remove components** — Decommission analytics, Cassandra, routers, message processors, ZooKeeper, entire environments, pods, or virtual hosts.
   - **Expand regions** — Add data centers to an existing planet.
   - **Upgrade** — In-place Apigee version upgrades.
   - **Backup & restore** — Component-level backup and restore.
   - **Cassandra rebuild** — Rebuild dead or replaced Cassandra nodes.
   - **Validations** — Smoke-test Cassandra, ZooKeeper, LDAP, Qpid, Router+MP, and analytics.
   - **Operational** — Download logs, restart components, debug mode, user management, monetization setup, virtual host management, analytics scope updates.

## Architecture

```
setup/                          # Ansible controller bootstrap
  setup.yml                       → apigee-opdk-setup-ansible-controller
  requirements.yml                → pulls Galaxy roles
infrastructure/                 # Environment preparation
  configure-ssh-login/
  ssh-bastion-host/
  ssh-tunnels/
  mirror/                          → apigee-opdk-setup-mirror-nginx + offline workflows
  download-offline-packages/
  setup-ansible-offline/
  response-file-generator/
  port-requirements/
  gce-management/                  → Terraform + Ansible for GCE
  backup-ansible-controller/
  clean-ansible/
  bastion-host-proxy/              → nginx + Let's Encrypt
installations/                  # Planet deployment
  aio/                             → single-node install
  multi-node/                      → HA planet (single or multi-region)
  devportal/                       → Drupal portal
  edge-microgateway/
post-installation/              # Lifecycle operations
  add/  remove/  upgrade/  backup/
  expand-planet-regions/  cassandra-rebuild/
  validations/  debug-mode/  ...
```

Roles are consumed via `ansible-galaxy install -r requirements.yml` and composed into multi-play playbooks. Property files in `~/.apigee/` and `~/.apigee-secure/` drive per-environment configuration.

## Usage

```bash
# 1. Set up the Ansible controller
cd setup/
ansible-galaxy install -r requirements.yml -f
ansible-playbook setup.yml

# 2. Configure inventory and credentials (see README-ansible-configuration.md, README-credentials.md)

# 3. Install a planet (example: multi-node)
cd installations/multi-node/
ansible-galaxy install -r requirements.yml -f
ansible-playbook install.yml

# 4. Run a subset (OS prerequisites only)
ansible-playbook install.yml --tags=os

# 5. Post-install: validate Cassandra
cd post-installation/validations/cassandra/
ansible-playbook validate.yml
```

## Provenance

Originally authored by Carlos Frias during tenure on the Apigee Edge Private Cloud team at Google. Forked from the `apigee/ansible-opdk-accelerator` open-source release and extended with additional infrastructure, offline, GCE, and operational playbooks.

<!-- BEGIN Google Required Disclaimer -->
This is not an officially supported Google product.
<!-- END Google Required Disclaimer -->

## License

Apache License 2.0 — see [LICENSE](./LICENSE).