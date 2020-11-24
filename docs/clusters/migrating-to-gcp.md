# Migrating to GCP

### Why migrate our application\(s\)?

* Access to self-service [buckets](../persistence/buckets.md) and [Postgres databases](../persistence/postgres.md). 
* Access to Google Cloud features.
* [Zero Trust security model](../appendix/zero-trust.md) instead of FSS/SBS zone model.
* [Built-in call tracing](https://istio.io/docs/tasks/observability/distributed-tracing/) similar to AppDynamics.
* Cost efficient and future proof.

### Prerequisites

The team needs to update their ROS and PVK analysis to migrate to GCP. Refer to [Google Cloud Platform's ROS and PVK section](./gcp.md#ros-and-pvk).

#### Basic setup

Follow the Getting started's [Access from laptop](../basics/access.md) instructions, and make sure to pay attention to the [GCP](../basics/access.md#google-cloud-platform-gcp) section.

#### Security

Our GCP clusters use a zero trust security model, implying that the application must specify both incoming and outgoing connections in order to receive or send traffic at all. This is expressed using the [access policy spec](../nais-application/access-policy.md).

The access policy also enables zone traversal and cross-cluster communication. This must be implemented in both applications, by using and accepting tokens from [TokenX](../security/auth/tokenx.md).

#### Deploy

The same deployment mechanism is leveraged for both on-premise and GCP K8s clusters. See [deployment](../deployment/README.md) section of the documentation for how to leverage the _NAIS deploy tool_.

#### Ingress

See [GCP clusters](gcp.md).

#### Privacy

Google is cleared to be a data processor for personally identifiable information \(PII\) at NAV. However, before your team moves any applications or data to GCP the following steps should be taken:

1. Verify that you have a valid and up-to-date PVK for your application. This document should be [tech stack agnostic](../legal/app-pvk.md) and as such does not need to be changed to reflect the move to GCP.
2. If the application stores any data in GCP, update [Behandlingskatalogen](https://behandlingskatalog.nais.adeo.no/) to reflect that Google is a data processor.

#### ROS

The ROS analysis for the team's applications need to be updated to reflect any changes in platform components used. For example, if your team has any specific measures implemented to mitigate risks related to "Kode 6 / 7 users", you should consider if these measures still apply on the new infrastructure or if you want to initiate any additional measures. When updating the ROS, please be aware that the GCP components you are most likely to use [have already undergone risk assessment by the nais team](../legal/nais-ros.md) and that you can refer to these ROS documents in your own risk assessment process.

#### Roles and responsibilites

As with applications in our on-premises clusters, the operation, security and integrity of any application is the responsibility of the team that owns that particular application. Conversely, it is the responsiblity of the nais platform team to handle the operation, security and integrity of the nais application platform and associated components. At GCP, Google is responsible for operating infrastructure underlying the nais platform, as well as any cloud services not consumed through the nais abstraction layer. Service exceptions reported by either Google or the nais team will be announced in the \#nais slack channel.

If your application stores personally identifiable information in any GCP data store, Google is effectively a data processor \("databehandler"\) for this data, and your documentation needs to reflect this fact.

### FAQ

#### What do we have to change?

* Cluster name: All references to cluster name. \(Logs, grafana, deploy, etc.\)
* Secrets: are now stored as native secrets in the cluster, rather than externally in vault.
* Namespace: If your application is in the `default` namespace, you will have to move to team namespace
* Storage: Use `GCS-buckets` instead of `s3` in GCP. Buckets, and access to them, are expressed in your [application manifest](../nais-application/nais.yaml/full-example.md)
* Ingress: There are some domains that are available both on-prem and in GCP, but some differ, make sure to verify before you move.
* Postgres: A new database \(and access to it\) is automatically configured when expressing `sqlInstance` in your [application manifest](../nais-application/nais.yaml/full-example.md) 

  We're currently investigating the possibility of using on-prem databases during a migration window.

* PVK: Update your existing PVK to include cloud

See [this table](migrating-to-gcp.md#gcp-compared-to-on-premises) for the differences between GCP and on-premises, and which that may apply to your application.

#### What should we change?

Use [TokenX](../security/auth/tokenx.md) instead of API-GW If using automatically configured [buckets](../persistence/buckets.md) or [postgres](../persistence/postgres.md), use [Google APIs](https://cloud.google.com/storage/docs/reference/libraries)

#### What do we not need to change?

You do not have to make any changes to your application code. Ingresses work the same way, although some domains overlap and others are exclusive. Logging, secure logging, metrics and alerts work the same way

#### What can we do now to ease migration to GCP later?

Make sure your PVK is up to date. Deploy your application to your team's namespace instead of `default`

#### What about PVK?

A PVK is not a unique requirement for GCP, so all applications should already have one. See [about security and privacy when using platform services](../README.md#about-security-and-privacy-when-using-platform-services) for details

#### How do we migrate our database?

See [Migrating databases to GCP](migrating-databases-to-gcp.md).

#### Why is there no vault in GCP?

There is native functionality in GCP that overlap with many of the use cases that Vault have covered on-prem. Using these mechanisms removes the need to deal with these secrets at all. Introducing team namespaces allows the teams to manage their own secrets in their own namespaces without the need for IAC and manual routines. For other secrets that are not used by the application during runtime, you can use the [Secret Manager](../security/secrets/google-secrets-manager.md) in each team's GCP project.

#### How do we migrate from vault to secrets manager

Retrieve the secret from vault and store it in a file. `kubectl apply -f <secret-file>`. See the [secrets documentation](../security/secrets/kubernetes-secrets.md) for an example.

#### How do we migrate from filestorage to buckets

Add a bucket to your application spec Copy the data from the filestore using [s3cmd](https://s3tools.org/s3cmd) to the bucket using [gsutil](https://cloud.google.com/storage/docs/gsutil)

#### What are the plans for cloud migration in NAV?

We aim to shut down both sbs clusters by summer 2021 NAVs strategic goal is to shut off all on-prem datacenters by the end of 2023

#### What can we do in our GCP project?

The teams GCP projects are primarily used for automatically generated resources \(buckets and postgres\). We're working on extending the service offering. However, additional access may be granted if required by the team

#### How long does it take to migrate?

A minimal application without any external requirements only have to change a single configuration parameter when deploying and have migrated their application in 5 minutes. See [this table](migrating-to-gcp.md#gcp-compared-to-on-premises) for the differences between GCP and on-premises, and which that may apply to your application.

### We have personally identifiable and/or sensitive data in our application, and we heard about the Privacy Shield invalidation. Can we still use GCP?

**Yes.** [NAV's evaluation of our Data Processor Agreement with Google post-Schrems II](https://navno.sharepoint.com/:w:/r/sites/Skytjenesterforvaltningsregime/_layouts/15/Doc.aspx?sourcedoc=%7BA9562232-BB00-40CB-930D-4EF254A5AD7F%7D&file=2020-10-10%20GCP%20-%20behandling%20og%20avtaler.docx&action=default&mobileredirect=true) is that it still protects us and is valid for use **given that data is stored and processed in data centers located within the EU/EEA**. If your team uses resources provisioned through NAIS, this is guaranteed by the nais team. If your team uses any other GCP services the team is responsible for ensuring that only resources within EU/EES are used \(as well as for evaluating the risk of using these services\).

See [Laws and regulations/Application PVK](../legal/app-pvk.md) for details.

## GCP compared to on-premises

| Feature | on-prem | gcp | Comment |
| :--- | :--- | :--- | :--- |
| Deploy | ✔️ | ✔️ | different clustername when deploying |
| Logging | ✔️ | ✔️ | different clustername in logs.adeo.no |
| Metrics | ✔️ | ✔️ | same mechanism, different datasource |
| Nais app dashboard | ✔️ | ✔️ | new and improved in GCP |
| Alerts | ✔️ | ✔️ | identical |
| Secure logs | ✔️ | ✔️ | different clustername in logs.adeo.no |
| Kafka | ✔️ | ✔️ | identical |
| Secrets | vault | Secret manager |  |
| Team namespaces | ✔️ | ✔️ |  |
| Shared namespaces | ✔️ | ✖️ | Default namespace not available for teams in GCP |
| Health checks | ✔️ | ✔️ | identical |
| Ingress | ✔️ | ✔️ | see [GCP](gcp.md) and [on-premises](on-premises.md) for available domains |
| Storage | Ceph | Buckets |  |
| Postgres | ✔️ \(IAC\) | ✔️ \(self-service\) |  |
| Laptop access | ✔️ | ✔️ |  |
| domain: dev.nav.no | ✔️ \(IAC\) | ✔️ \(Automatic\) | Wildcard DNS points to GCP load balancer |
| domain: dev.adeo.no | ✔️ \(IAC\) | ✔️ \(Automatic\) | Wildcard DNS points to GCP load balancer |
| Access to FSS services | ✔️ | ✔️ | Identical \(either API-gw or [TokenX](../security/auth/tokenx.md) |
| OpenAM | ✔️ | ✖️ | OpenAM is EOL, use [TokenX](../security/auth/tokenx.md) |
| NAV truststore | ✔️ | ✔️ |  |
| PVK required | ✔️ | ✔️ | amend to cover storage in cloud |
| Security | Zone Model | [zero-trust](../appendix/zero-trust.md) |  |
