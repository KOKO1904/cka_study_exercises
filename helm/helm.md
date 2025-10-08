# Helm

### Your company requires the installation of Apache HTTP Server version 8.5.70 in their Kubernetes cluster using the official Helm chart from the Bitnami repository: https://charts.bitnami.com/bitnami.

Please complete the following tasks:

- Create a Helm template to install Apache HTTP Server v8.5.70 with the mod_status module enabled. Locate the apacheExtraModules section and add the module there.

- Proceed to install Apache HTTP Server using this template.

---

<details>
<summary>Show commands / answers</summary>
<p>


```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami | grep apache

# This is the result
bitnami/apache      11.4.29         2.4.65          Apache HTTP Server is an open-source HTTP serve...

helm show values bitnami/apache --version 8.5.70 > apache-values.yaml

# Search for this section
apacheExtraModules: []

# Add the status_module to enable serverStatus
apacheExtraModules:
  - mod_status

# Generate the manifests using Helm template
helm template apache bitnami/apache --version 8.5.70 -f apache-values.yaml
```

</p>
</details>
