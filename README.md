Ansible Role: Nextcloud for Kubernetes
======================================

Ansible role to install [Nextcloud](https://nextcloud.com/) on Kubernetes.

Role Variables
--------------

```yaml
# Namespace
kubernetes_nextcloud_namespace: "default"
# App name (used as selector)
kubernetes_nextcloud_app: "nextcloud"
# Deployment name
kubernetes_nextcloud_deployment: "nextcloud-deployment"
# Cronjob name
kubernetes_nextcloud_cronjob: "nextcloud-cron"
# Configmap name
kubernetes_nextcloud_configmap: "nextcloud"
# Service name
kubernetes_nextcloud_service: "nextcloud"

# Number of replicas
kubernetes_nextcloud_replicas: 1
kubernetes_nextcloud_revision_history: 1

# Node selector
kubernetes_nextcloud_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_nextcloud_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_nextcloud_deployment_annotations: {}

kubernetes_nextcloud_resources:
  limits:
    memory: "512Mi"
  requests:
    memory: "256Mi"

kubernetes_nextcloud_nginx_resources:
  limits:
    memory: "256Mi"
  requests:
    memory: "64Mi"

# Cronjob period
kubernetes_nextcloud_cron_period: "*/15 * * * *"

# Useful when nextcloud restrict the requests to some hostnames
# IF NOT SET CORRECTLY, CRON AND HEALTHCHECKS CAN BREAK
kubernetes_nextcloud_hostname: "nextcloud.example.com"

# Setup ingress for nextcloud
kubernetes_nextcloud_setup_ingress: false
kubernetes_nextcloud_ingress:
  name: "nextcloud-ingress"
  host: "nextcloud.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

## Required volumes ##

# Nextcloud data volume, mounted as /var/www/html/data (see examples for more
# details)
kubernetes_nextcloud_data_volume:
  definition:

# Nextcloud config volume, mounted as /var/www/html/config (see examples for
# more details)
kubernetes_nextcloud_data_volume:
  definition:

## Optional volumes ##

# Next volumes are optional, and are setup as emptyDir by default. They are
# useful to customize Nextcloud, and also to share the same apps between
# multiple Nextcloud instances.

# Nextcloud base directory, mounted as /var/www/html, useful to share apps
# between multiple instances
kubernetes_nextcloud_base_volume:
  definition:
    emptyDir: {}

# Nextcloud custom-apps directory, mounted as /var/www/html/custom_apps
kubernetes_nextcloud_custom_apps_volume:
  definition:
    emptyDir: {}

# Nextcloud theme directory, mounted as /var/www/html/themes
kubernetes_nextcloud_theme_volume:
  definition:
    emptyDir: {}
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_nextcloud_config_volume:
      subPath: config
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: nextcloud
          readOnly: false

    kubernetes_nextcloud_data_volume:
      subPath: data
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: nextcloud
          readOnly: false

    kubernetes_nextcloud_base_volume:
      subPath: basedir
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: nextcloud
          readOnly: false

    kubernetes_nextcloud_setup_ingress: true
    kubernetes_nextcloud_ingress:
      name: "nextcloud-ingress"
      host: "nextcloud.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "nextcloud-ingress-tls"
          hosts:
            - "nextcloud.example.com"

    kubernetes_nextcloud_hostname: "{{ kubernetes_nextcloud_ingress.host }}"
  roles:
    - role: Anthony25.kubernetes-nextcloud
```

Use `run_once` to run the role on only one available master in the cluster.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
