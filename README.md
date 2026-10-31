# airflow_gcp_spark_snf_car_rental_processing


## Setup

### 1. Google Cloud

1. Enable the Cloud Composer and IAM Service Account Credentials APIs.
2. Create the Composer environment with a service account that has the **Composer Worker** role.
3. Create a `github-deployer` service account with:
   - **Storage Object Admin** on the Composer DAG bucket
   - A role that allows `composer.environments.get`
4. Create a Workload Identity Pool (`github-pool`) and OIDC provider (`github-provider`):
   - Issuer: `https://token.actions.githubusercontent.com`
   - Attribute mapping: `google.subject = assertion.sub`, `attribute.repository = assertion.repository`
   - Condition (allows repos owned by this GitHub account): `assertion.repository_owner == 'Ashutosh1104'`
5. Allow this repository to impersonate `github-deployer`. On the pool, choose **Grant access** > service account impersonation, select `github-deployer`, and set:
   - Attribute name: `repository`
   - Attribute value: `Ashutosh1104/news_api_data_airflow_snowflake_gcp`

   The provider condition only checks the owner. This per-repo binding is what restricts which repository can use the service account.

#### Adding another repository

Reuse the same pool and provider. No new pool is needed.

1. Repeat step 5 with the new repo's name (`Ashutosh1104/<new-repo>`).
2. Use `github-deployer` if it deploys to the same Composer environment. Otherwise create a separate service account with only the roles it needs, and bind the new repo to that.
3. Add the same `WIF_PROVIDER` secret and the matching `GCP_SA_EMAIL` secret in the new repo.

### 2. GitHub

Add these repository secrets (Settings > Secrets and variables > Actions):

| Secret | Value |
|---|---|
| `WIF_PROVIDER` | `projects/<PROJECT_NUMBER>/locations/global/workloadIdentityPools/github-pool/providers/github-provider` |
| `GCP_SA_EMAIL` | `github-deployer@<PROJECT_ID>.iam.gserviceaccount.com` |

Optional repository variables: `COMPOSER_ENV`, `COMPOSER_REGION` (defaults are set in the workflow).