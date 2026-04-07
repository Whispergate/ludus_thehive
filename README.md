# Ansible Role: TheHive 5 ([Ludus](https://ludus.cloud))

An Ansible Role that installs [TheHive 5](https://strangebee.com/thehive/) on Debian/Ubuntu hosts with Cassandra, Elasticsearch, and optional integrations for [MISP](https://www.misp-project.org/), [Wazuh](https://wazuh.com/), and [OpenCTI](https://www.opencti.io/).

This role installs and configures a complete standalone TheHive instance including:
- Java 11 (Amazon Corretto)
- Apache Cassandra 4.1 (database)
- Elasticsearch 7.17 (index engine)
- TheHive 5.4 (incident response platform)

## Requirements

- Debian 11/12 or Ubuntu 20.04/22.04
- Minimum 4 CPU cores and 8 GB RAM recommended
- 100 GB disk space minimum

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### TheHive

    # TheHive version
    ludus_thehive_version: "5.4.5-1"

    # HTTP port for the web UI
    ludus_thehive_port: 9000

    # Secret key (auto-generated if empty)
    ludus_thehive_secret: ""

### Cassandra

    ludus_thehive_cassandra_version: "4.1"
    ludus_thehive_cassandra_listen_address: "127.0.0.1"
    ludus_thehive_cassandra_cluster_name: "thp"
    ludus_thehive_cassandra_keyspace: "thehive"
    ludus_thehive_cassandra_thehive_user: "thehive"
    ludus_thehive_cassandra_thehive_password: "thehive"

### Elasticsearch

    ludus_thehive_elasticsearch_version: "7.17"
    ludus_thehive_elasticsearch_host: "127.0.0.1"
    ludus_thehive_elasticsearch_port: 9200
    ludus_thehive_elasticsearch_heap_size: "1g"

### File Storage

    # "local" or "s3"
    ludus_thehive_storage_type: "local"
    ludus_thehive_storage_local_path: "/opt/thp/thehive/files"

    # S3/MinIO settings (only when storage_type is "s3")
    ludus_thehive_storage_s3_endpoint: "http://127.0.0.1:9100"
    ludus_thehive_storage_s3_bucket: "thehive"
    ludus_thehive_storage_s3_access_key: "minioadmin"
    ludus_thehive_storage_s3_secret_key: "minioadmin"

### Cortex Integration

    ludus_thehive_cortex_enabled: false
    ludus_thehive_cortex_url: "http://127.0.0.1:9001"
    ludus_thehive_cortex_api_key: ""

### MISP Integration

    ludus_thehive_misp_enabled: false
    ludus_thehive_misp_servers:
      - name: "MISP"
        url: "https://misp.example.com"
        auth_key: "your-misp-api-key"
        organisation: "default"
        max_age_days: 7
        purpose: "ImportAndExport"

### Wazuh Integration

    ludus_thehive_wazuh_enabled: false
    ludus_thehive_wazuh_alert_level: 6
    ludus_thehive_wazuh_api_key: ""

### OpenCTI Integration

    ludus_thehive_opencti_enabled: false
    ludus_thehive_opencti_api_key: ""

> [!NOTE]
> OpenCTI connects to TheHive via the [OpenCTI TheHive connector](https://github.com/OpenCTI-Platform/connectors). Configure the connector on the OpenCTI side using TheHive's URL and an API key.

## Dependencies

None.

## Example Playbook

```yaml
- hosts: thehive_hosts
  roles:
    - Whispergate.ludus_thehive
  vars:
    ludus_thehive_misp_enabled: true
    ludus_thehive_misp_servers:
      - name: "MISP"
        url: "https://10.2.99.50"
        auth_key: "your-misp-api-key"
        ws_ssl_verify: false
```

## Example Ludus Range Config

```yaml
ludus:
  - vm_name: "{{ range_id }}-thehive"
    hostname: "{{ range_id }}-thehive"
    template: debian-12-x64-server-template
    vlan: 20
    ip_last_octet: 50
    ram_gb: 8
    cpus: 4
    roles:
      - Whispergate.ludus_thehive
    role_vars:
      ludus_thehive_misp_enabled: true
      ludus_thehive_misp_servers:
        - name: "MISP"
          url: "https://10.2.99.51"
          auth_key: "your-misp-api-key"
          ws_ssl_verify: false
      ludus_thehive_cortex_enabled: true
      ludus_thehive_cortex_url: "http://10.2.99.52:9001"
      ludus_thehive_cortex_api_key: "your-cortex-api-key"
```

### With Wazuh Integration

If deploying alongside a Wazuh manager, enable the Wazuh integration to forward alerts:

```yaml
role_vars:
  ludus_thehive_wazuh_enabled: true
  ludus_thehive_wazuh_api_key: "your-thehive-api-key"
  ludus_thehive_wazuh_alert_level: 6
```

> [!WARNING]
> The Wazuh integration deploys a script to `/var/ossec/integrations/`. This requires the target host to have Wazuh manager installed. You must also add the integration block to your Wazuh `ossec.conf` manually or via a separate role.

### With OpenCTI

OpenCTI connects to TheHive using its own connector. On the OpenCTI side, configure the `opencti/connector-thehive` Docker container with TheHive's URL and API key. This role sets `ludus_thehive_opencti_enabled` as a flag for documentation purposes.

## Default Credentials

After installation, TheHive is accessible at `http://<host_ip>:9000` with default credentials:
- **Username:** `admin@thehive.local`
- **Password:** `secret`

> [!WARNING]
> Change the default credentials immediately after first login.

## License

MIT

## Author Information

This role was created by [Whispergate](https://github.com/Whispergate), for [Ludus](https://ludus.cloud/).
