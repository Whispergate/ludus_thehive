# Ansible Role: TheHive 5 ([Ludus](https://ludus.cloud))

An Ansible Role that deploys [TheHive 5](https://strangebee.com/thehive/) on Debian/Ubuntu using the official [StrangeBee docker-compose stack](https://github.com/StrangeBeeCorp/docker). Supports optional integrations for [MISP](https://www.misp-project.org/), [Wazuh](https://wazuh.com/), and [OpenCTI](https://www.opencti.io/).

The role:
- Installs Docker Engine + Compose plugin
- Clones the canonical StrangeBee docker repo
- Generates `.env`, `index.conf`, `secret.conf`, and TLS certificates non-interactively
- Brings up Cassandra, Elasticsearch, TheHive, and nginx via `docker compose`
- Waits for TheHive's API to report healthy

## Requirements

- Debian 11/12 or Ubuntu 20.04/22.04/24.04
- **At least 8 GB RAM** (the role will fail fast if less — Cassandra and Elasticsearch each want several GB heap)
- At least 4 CPUs recommended
- 100+ GB disk for production use

## Role Variables

See `defaults/main.yml` for the full list. Key variables:

```yaml
# Minimum RAM the role will allow (in MB)
ludus_thehive_min_ram_mb: 7168

# Where to clone the StrangeBee docker repo and run the stack
ludus_thehive_install_dir: /opt/thehive

# Profile: "prod1-thehive" (standard, ~10GB RAM) or "prod2-thehive" (high-perf, ~25GB RAM)
ludus_thehive_profile: prod1-thehive

# Pin the StrangeBee repo to a specific commit for reproducibility
ludus_thehive_git_version: main

# TheHive web UI port (host side)
ludus_thehive_port: 9000

# Expose nginx TLS reverse proxy on 443
ludus_thehive_enable_nginx: true
ludus_thehive_server_name: "thehive.local"

# Auto-generated if empty
ludus_thehive_elasticsearch_password: ""
ludus_thehive_play_secret: ""
```

> [!NOTE]
> MISP, Cortex, and OpenCTI integrations are enabled in the image by default. Configure actual server connections after first login via **Platform Management > Connectors** in the TheHive UI.

## Dependencies

None (Docker is installed by the role itself).

## Example Playbook

```yaml
- hosts: thehive_hosts
  roles:
    - Whispergate.ludus_thehive
  vars:
    ludus_thehive_server_name: "hive.icsrange.internal"
```

## Example Ludus Range Config

```yaml
ludus:
  - vm_name: "{{ range_id }}-thehive"
    hostname: "{{ range_id }}-thehive"
    template: ubuntu-24.04-x64-server-template
    vlan: 10
    ip_last_octet: 4
    ram_gb: 8
    cpus: 4
    linux: true
    dns_rewrites:
      - hive.icsrange.internal
    roles:
      - Whispergate.ludus_thehive
    role_vars:
      ludus_thehive_server_name: "hive.icsrange.internal"
      ludus_thehive_misp_enabled: true
```

## Default Credentials

After deploy, TheHive is at `http://<host_ip>:9000` (and `https://<host_ip>` if nginx is enabled with a self-signed cert):

- **Username:** `admin@thehive.local`
- **Password:** `secret`

> [!WARNING]
> Change the default credentials immediately on first login.

## Operating the stack

The stack lives under `{{ ludus_thehive_install_dir }}/{{ ludus_thehive_profile }}` on the host. Standard compose commands work from that directory:

```bash
cd /opt/thehive/prod1-thehive
docker compose ps
docker compose logs -f thehive
docker compose restart thehive
```

## Wazuh Integration

Setting `ludus_thehive_wazuh_enabled: true` deploys a Python integration script to `/var/ossec/integrations/` on the same host. This only makes sense if the target is also a Wazuh manager. You still need to add the `<integration>` block to Wazuh's `ossec.conf` manually or via a separate role.

## License

MIT

## Author Information

This role was created by [Whispergate](https://github.com/Whispergate), for [Ludus](https://ludus.cloud/).
