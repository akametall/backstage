# Backstage deployment

# Create SSL 
Go to ****** and create an SSL certificate for URL created in previous step. Download certificate and private key, they will be needed in next steps.

# Get  Root CA certificate
Go to ****** and download Root CA certificate, it will be used in the next step.

# Create Gitlab Token
The token need at least `api`, `read_repository` and `write_repository` scopes.

# Edit `secret.yaml` to add env for backsatge and its plugins
Values must be encoded in BASE64

Example:

```yaml
data:
  GITLAB_TOKEN: ==
  ARGOCD_AUTH_TOKEN: ==
  NODE_EXTRA_CA_CERTS: ==
  SONARQUBE_TOKEN: ==
  root-ca.crt: ==
  VAULT_TOKEN: ==
```

# Edit `configmap.yaml` to configure backstage. For example to add some entrypoints for plugins.
Need to change values in `backend` section.

Example
```yaml
    backend:
      baseUrl: https://backstage.smdomain.com
```

`integrations` section.

Example
```yaml
      gitlab:
        - apiBaseUrl: 'https://gitlab.***.ru/api/v4'
          baseUrl: 'https://gitlab.***.ru'
          host: 'gitlab.***.ru'
          token: ${GITLAB_TOKEN}
```

`proxy` section

Example
```yaml
      '/gitlabci':
          target: 'https://gitlab.***.ru/api/v4'
          allowedMethods: ['GET']
          headers:
            PRIVATE-TOKEN: '${GITLAB_TOKEN}'
```

`catalog` section

Example
```yaml
      locations:
        - type: url
          target: https://gitlab.***.ru/catalog/-/blob/master/catalog.yaml
```
Location target ***always*** must be `blob` to prevent errors

For more additional info read https://backstage.io/docs/conf/

# Edit `ingress.yaml`
Add domain name into `hosts` and `host` fields

# (Optional) Change or put catalog file
Location of catalog files is https://gitlhub.ru/catalog

***WARN!*** if you create new catalog file with different name or location you must actualize `catalog` section of `configmap.yaml`

# Edit `backstage.yaml` in argocd applications directory
Directory address https://github.ru/argocd-apps-int/-/tree/master/applications/backstage/in-cluster/backstage
