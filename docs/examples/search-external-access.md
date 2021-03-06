# Enable Alfresco Search Services External Access
This example demonstrates how to enable Alfresco Search Services (`/solr`) for external access.

## Prerequisites

You will need to make sure the [ACS Ingress controller](../helm-deployment-aws_kops.md#deploying-the-ingress-for-alfresco-content-services-community) is installed.

## Deployment

By default, the Alfresco Search Services endpoint (`/solr`) is disabled for external access due to security reasons.  But, this can be enabled should you wish.  You may need to adjust the configuration settings as per [acs-community-deployment configuration table](../../helm/alfresco-content-services-community#configuration).

See [ACS Community Deployment](../helm-deployment-aws_kops.md#deploying-alfresco-content-services-community) for full reference to ENVIRONMENT variables used in the following snippets.

### Install ACS Helm Chart with Search external access

Below is the snippet for installing ACS Community with Search external access.

```bash
helm install alfresco-incubator/alfresco-content-services-community \
--set externalProtocol="https" \
--set externalHost="$EXTERNALHOST" \
--set externalPort="443" \
--set repository.adminPassword="$ALF_ADMIN_PWD" \
--set postgresql.postgresPassword="$ALF_DB_PWD" \
--set alfresco-infrastructure.persistence.efs.enabled=true \
--set alfresco-infrastructure.persistence.efs.dns="$EFS_SERVER" \
--set alfresco-search.resources.requests.memory="2500Mi",alfresco-search.resources.limits.memory="2500Mi" \
--set alfresco-search.environment.SOLR_JAVA_MEM="-Xms2000M -Xmx2000M" \
--set alfresco-search.ingress.enabled=true \
--set alfresco-search.ingress.basicAuth="YWRtaW46JGFwcjEkVVJqb29uS00kSEMuS1EwVkRScFpwSHB2a3JwTDd1Lg==" \
--set alfresco-search.ingress.whitelist_ips="0.0.0.0/0" \
--set postgresql.persistence.subPath="$DESIREDNAMESPACE/alfresco-content-services-community/database-data" \
--set persistence.repository.data.subPath="$DESIREDNAMESPACE/alfresco-content-services-community/repository-data" \
--set persistence.solr.data.subPath="$DESIREDNAMESPACE/alfresco-content-services-community/solr-data" \
--namespace=$DESIREDNAMESPACE
```

### Upgrade ACS Helm Chart with Search external access

Below is the snippet for upgrading ACS with Search external access (where it was previously disabled).

```bash
helm upgrade \
--set alfresco-infrastructure.persistence.efs.enabled=true \
--set alfresco-infrastructure.persistence.efs.dns="$EFS_SERVER" \
--set alfresco-search.ingress.enabled=true \
--set alfresco-search.ingress.basicAuth="YWRtaW46JGFwcjEkVVJqb29uS00kSEMuS1EwVkRScFpwSHB2a3JwTDd1Lg==" \
--set alfresco-search.ingress.whitelist_ips="0.0.0.0/0" \
$ACSRELEASE alfresco-incubator/alfresco-content-services-community
```

**Note:** There are known issues when upgrading a Helm chart relating to Helm cache.
- https://github.com/kubernetes/helm/issues/3275
- https://github.com/kubernetes/helm/issues/1193
- https://github.com/kubernetes/helm/pull/4146

If your `helm upgrade` fails due to any of these example errors:
```bash
Error: UPGRADE FAILED: no Secret with the name "nosy-tapir-alfresco-search-solr" found
(or)
Error: UPGRADE FAILED: no Ingress with the name "nosy-tapir-alfresco-search-solr" found
```

Then, simply delete that resource.  Below is an example:
```bash
kubectl delete secret nosy-tapir-alfresco-search-solr --namespace=$DESIREDNAMESPACE
(or)
kubectl delete ingress nosy-tapir-alfresco-search-solr --namespace=$DESIREDNAMESPACE
```

And re-try above Upgrade ACS Helm Chart steps which will also re-create the above deleted resource.
