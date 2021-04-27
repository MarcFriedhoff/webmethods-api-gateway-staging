# About API Gateway Staging

The API Gateway Staging solution allows to extract API Gateway assets from a local development environment or a central DEV environment, add them to the Azure DevOps repository (Git) and automatically promote them to STAGE and PROD environments, controlled by Azure DevOps build pipelines. During the promotion, the assets are first imported on a BUILD environment where they are automatically tested (based on Postman collections) and specifically prepared for the intended target environment (also based on Postman collections): The pipeline will automatically remove all applications which are not intended for the target environment (with names not ending with _STAGE or _PROD), activate (unsuspend) all other applications and add API tags to all APIs indicating the build ID, the build name and the pipeline name (for auditability). After that procedure, the assets are exported again from BUILD environment and imported on the target environment (STAGE or PROD).

The solution also includes a script for automatically extracting the general configuration of API Gateway instances (local development environment, DEV, BUILD, STAGE and PROD). The configuration includes the configuration of the API Gateway destination (event types, performance metrics and audit log data to be stored in the internal Elasticsearch database), the configuration of an outbound proxy server (incl. bypass addresses) and the Transaction logging global policy. The configuration can be added to the repository and imported on API Gateway instances using Azure DevOps build pipelines for quickly setting up new API Gateway instances or for updating existing instances. After importing the base configuration, the build pipelines will also add environment-specific configuration items like loadbalancer URL and the OAuth2 configuration (authorization code and access token expiration interval) in the local OAuth2 Authorization Server alias.

This solution is based on https://github.com/thesse1/webmethods-api-gateway-devops which is by itself a fork of https://github.com/SoftwareAG/webmethods-api-gateway-devops.

## Some background

As each organization builds APIs using API Gateway for easy consumption and monetization, the continuous integration and delivery are integral part of the API Gateway solutions to meet the consumer demands. We need to automate the management of APIs and policies to speed up the deployment, introduce continuous integration concepts and place API artifacts under source code management. As new apps are deployed, the API definitions can change and those changes have to be propagated to other external products like API Portal. This requires the API owner to update the associated documentation and in most cases this process is a tedious manual exercise. In order to address this issue, it is a key to bring in DevOps style automation to the API life cycle management process in API Gateway. With this, enterprises can deliver continuous innovation with speed and agility, ensuring that new updates and capabilities are automatically, efficiently and securely delivered to their developers and partners in a timely fashion and without manual intervention. This enables a team of API Gateway policy developers to work in parallel developing APIs and policies to be deployed as a single API Gateway configuration.

![GitHub Logo](/images/api.png)

In addition to the functionality of the original webmethods-api-gateway-devops template, this solution includes an automatic adjustment of API Gateway assets for the deployment on different stages. It implements the following requirements: APIs should have separate sets of applications (with different identifiers) on different stages. The correct deployment of these applications should be enforced automatically. All applications are created on a local development environment or the central DEV environment with names ending with "_DEV", "_STAGE" or "_PROD" indicating their intended usage. All applications should be exported and managed in VCS, but only the _STAGE and _PROD applications should be imported on the respective STAGE and PROD environments. This is implemented by manipulating the assets on a dedicated BUILD environment: Initially, all assets (including all applications) are imported on the BUILD environment. Then all applications except for _STAGE or _PROD, respectively, are automatically deleted from the BUILD environment using the API Gateteway Appication Management API. Finally, the API project is exported again from the BUILD environment (now only including the right applications for the target environment) and imported on the target environment.

![GitHub Logo](/images/devops_flow.png)

## webMethods API Gateway assets and configurations

The following API Gateway assets and configurations can be moved across API Gateway stages:
 - Gateway APIs 
 - Policy Definitions/Policy Templates/Global Policies
 - Applications
 - Aliases
 - Plans
 - Packages
 - Subscriptions
 - Users/Groups/ACLs/Teams
 - General Configurations like Load balancer, Extended settings
 - Security configurations
 - Destination configurations
 - External accounts configurations
 
## Devops and CI/CD in webMethods API Gateway

The CI/CD and devops flow can be achieved in multiple ways.

### Using webMethods Deployer and Asset Build Environment

API Gateway asset binaries can be build using Asset Build Environment and promoted across stages using WmDeployer. More information on this way of CI/CD and DevOps automation can be found at https://tech.forums.softwareag.com/t/staging-promotion-and-devops-of-api-gateway-assets/237040.

### Using Promotion Management APIs

The promotion APIs that are exposed by API Gateway can be used for the DevOps automation. More information on these APIs can be found at https://github.com/SoftwareAG/webmethods-api-gateway/blob/master/apigatewayservices/APIGatewayPromotionManagement.json.

### Directly using the API Gateway Archive API for exporting and importing asset definitions

This approach is followed in this solution.

## About this repository

This repository provides assets/scripts for implementing the CI/CD solution for API Gateway assets and general configurations. The artifacts in this repository use the API Gateway Archive API for automation of the DevOps flow.

The repository has the following folders:
  - apis: Contains projects with the API Gateway assets exported from DEV environment along with the definition of the projects' asset sets and API tests (Postman collections)
  - bin: Windows batch script that exports a defined set of API Gateway assets from DEV environment and stores the asset definition in file system
  - configuration: Contains folders with the API Gateway configuration assets exported from DEV, BUILD, STAGE and PROD environments along with the definition of the exported asset sets
  - environments: Postman environment definitions for API Gateway DEV, BUILD, STAGE and PROD environments
  - pipelines: Contains the Azure DevOps pipelines for deploying API Gateway assets on DEV, BUILD, STAGE and PROD environments
  - utilities: Contains Postman collections for importing API Gateway assets, for preparing (cleaning) the BUILD environment, for preparing the API Gateway assets on BUILD for the target environment and for initializing API Gateway instances with environment-specific configurations

The repository content can be committed to the Azure DevOps repository (Git), it can be branched, merged, rolled-back like any other code repository. Every commit to any branch in the Azure DevOps repository can be imported back to a local development environment, to the central DEV environment or promoted to STAGE or PROD.

## Develop and test APIs using API Gateway

The most common use case for an API Developer is to develop APIs in their local development environments or the central DEV environment and then export them to a flat file representation such that they can be integrated to any VCS. Also developers need to import their APIs from a VCS (flat file representation) to their local development environments for further updates.

The gateway_import_export_utils.bat under /bin can be used for this. Using this batch script, the developers can export APIs from their local development API Gateway or the central DEV environment to their VCS local repository and vice versa. In addition to that, the gateway_import_export_utils.bat batch script can also be used for exporting or importing a defined set of general configuration assets from/to local development environments, DEV, BUILD, STAGE or PROD.

## gateway_import_export_utils.bat

The gateway_import_export_utils.bat can be used for importing and exporting APIs (projects) in a flat file representation. The export_payload.json file in each project folder under /apis defines which API Gateway assets belong to this project. The assets will be imported/exported into/from their respective project folders under /apis.

| Parameter | README |
| ------ | ------ |
| importapi or exportapi |  To import or export from/to the flat file representation |
| api_name | The name of the API project to import or export |
| apigateway_url |  API Gateway URL to import to or export from |
| username |  The API Gateway username. The user must have the "Export assets" or "Import assets" privilege, respectively, for the --exportapi and --importapi option |
| password | The API Gateway user password |

Sample Usage for importing the API petstore that is present as flat file representation under apis/petstore/assets into API Gateway server at https://daehgcs51570.hycloud.softwareag.com:11000

```sh
bin>gateway_import_export_utils.bat --importapi --api_name petstore --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
```

Sample Usage for exporting the API petstore that is present on the API Gateway server at https://daehgcs51570.hycloud.softwareag.com:11000 as flat file under apis/petstore/assets

```sh
bin>gateway_import_export_utils.bat --exportapi --api_name petstore --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
```

The batch script can also be used for importing and exporting general API Gateway configurations in a flat file representation. The export_payload.json file in each folder under /configuration defines which API Gateway assets belong to this configuration. The assets will be imported/exported into/from their respective folders under /configuration.

| Parameter | README |
| ------ | ------ |
| importconfig or exportconfig |  To import or export from/to the flat file representation |
| environment | The type fo the environment to import or export (DEV, BUILD, STAGE or PROD) |
| apigateway_url |  API Gateway URL to import to or export from |
| username |  The API Gateway username. The user must have the "Export assets" or "Import assets" privilege, respectively, for the --exportconfig and --importconfig option |
| password | The API Gateway user password |

Sample Usage for importing the configuration that is present as flat file representation under configuration/DEV/assets into API Gateway server at https://daehgcs51570.hycloud.softwareag.com:11000

```sh
bin>gateway_import_export_utils.bat --importconfig --environment DEV --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
```

Sample Usage for exporting the configuration that is present on the API Gateway server at https://daehgcs51570.hycloud.softwareag.com:11000 as flat file under configuration/DEV/assets

```sh
bin>gateway_import_export_utils.bat --exportconfig --environment DEV --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
```

## export_payload.json export query for API projects

The set of assets exported by gateway_import_export_utils.bat --exportapi is defined by the export_payload.json in the API project root folder. It must be a JSON document applicable for the API Gateway Archive Service POST /archive request payload, cf. https://api.webmethodscloud.eu/#sagapis/apiDetails/c.restObject.API-Portal._N0usdLdEelRUwr3rpYDZg.-1. It will typically contain a list of asset types ("types") to be exported and a query ("scope") based on the IDs of the selected assets.

The apis folder contains sample API projects with the following export_payload.json files:

### petstore

```
{
  "types": [
    "api"
  ],
  "scope": [
    {
      "attributeName": "id",
      "keyword": "f3d2a3c1-0f83-43ab-a6ec-215b93e2ecf5"
    }
  ]
}
```

This example will select the API with asset ID f3d2a3c1-0f83-43ab-a6ec-215b93e2ecf5 (the petstore API). It will automatically also include all applications defined for this API.

### petstore-versioning

```
{
  "types": [
    "api"
  ],
  "scope": [
    {
      "attributeName": "id",
      "keyword": "4ea2dcf0-66c5-469b-b822-fe4707c6f899|4bd552e3-064f-444a-bc77-7560059c9955"
    }
  ]
}
```

This API project includes two APIs, actually two versions of the same API. They must be configured separately in the export_payload.json of the API project.

### postman-echo

```
{
  "types": [
    "api"
  ],
  "scope": [
    {
      "attributeName": "id",
      "keyword": "d73d5702-6925-41a0-8c23-933326c27b96"
    }
  ]
}
```

This example features the Postman Echo API (https://learning.postman.com/docs/developer/echo-api/) which is often used for demonstrating API Management features. For each request type (POST, GET, DELETE), it extracts the header, query and path parameters and the request body from the request and echoes them back in the response payload.

### multiple-tenants

```
{
  "types": [
    "api"
  ],
  "scope": [
    {
      "attributeName": "id",
      "keyword": "5c4f8c51-1fc3-4767-a3cd-30bdb9a04ca6|9be25215-20c6-4e8c-803a-d9ddac3394cc|ffcf4a33-e4ce-401f-b349-9056fe88abf4|328ae890-452f-43f1-9501-b708c7afc83f|0e235520-0121-4b44-a086-750f4422072f|0245c29f-e583-4ebd-824a-a4762ad85fbf"
    }
  ]
}
```

This API project demonstrates two ways of implementing multi-tenancy in API Gateway. It contains four APIs which are specifically designed for one tenant each:
 - Tenant1_Petstore
 - Tenant1_PostmanEcho
 - Tenant2_Petstore
 - Tenant2_PostmanEcho

In order to simulate different endpoints per tenant, the Tenant1 APIs add a tenant=Tenant1 query parameter to the native service endpoint, and the Tenant2 APIs add a tenant=Tenant2 query parameter to the native service endpoint. In the Postman Echo API, this is reflected in the response from the native service.

And the API project contains two APIs which can be used by both Tenant1 and Tenant2:
 - Common_Petstore
 - Common_PostmanEcho

These APIs will automatically identify the tenant by the incoming API key, and then they will "route" the request to the right endpoint by adding tenant=Tenant1 or tenant=Tenant2 to the native endpoint in a Context-based Routing policy.

### postman-echo-oauth2

```
{
  "types": [
    "api", "gateway_scope"
  ],
  "scope": [
    {
      "attributeName": "id",
      "keyword": "4ceba5ec-f0bc-42c7-a431-40ec951aeecf|69fb0cfe-d549-4572-92d9-cb1131d45e61"
    }
  ]
}
```

This API is using OAuth2 for inbound authentication. Therefore, the developer must also include the scope mapping ("gateway_scope") in the export set.

> Note: The local OAuth2 scope itself cannot be promoted from stage to stage using this mechanism, because it is configured in the general "local" system alias for the internal OAuth2 Authorization Server. OAuth2 scopes needed for the API(s) in an API project can be configured in the API project's scopes.json file, see below.

## scopes.json configuration of OAuth2 scopes

Each API project can include one scopes.json file in the API project root folder specifying the OAuth2 scopes needed by the API(s) in the API project. The file will be parsed right before importing the other API Gateway assets of the API project and the scopes are injected into the local OAuth2 Authorization Server configuration. ("UPSERT": Existing scope definitions with the same name will be overwritten, new scope definitions with new names will be added.)

The postman-echo-oauth2 sample project includes the following scopes.json file configuring the scope name and description user by the API:

```
[
    {
        "name": "postman-echo",
        "description": "postman-echo scope definition"
    }
]
```

The JSON array can include multiple scope definitions.

## export_payload.json export query for API Gateway configurations

The set of assets exported by gateway_import_export_utils.bat --exportconfig is defined by the export_payload.json in the configuration root folder. It must be a JSON document applicable for the API Gateway Archive Service POST /archive request payload, cf. https://api.webmethodscloud.eu/#sagapis/apiDetails/c.restObject.API-Portal._N0usdLdEelRUwr3rpYDZg.-1. It will typically contain a list of asset types ("types") to be exported and a query ("scope") based on the IDs of the selected assets.

The configuration folder contains sample configurations for DEV, BUILD, STAGE and PROD environments, for example:

### DEV

```
{
  "types": [
    "policy", "outbound_proxy", "proxy_bypass", "administrator_setting"
  ],
  "scope": [
    {
      "attributeName": "id",
      "keyword": "GlobalLogInvocationPolicy"
    },
    {
      "attributeName": "entityId",
      "keyword": "HTTPS_Proxy|proxyBypass"
    },
    {
      "attributeName": "configId",
      "keyword": "gatewayDestinationConfig"
    }
  ],
  "condition" : "or"
}
```

This configuration includes the Transaction logging global policy, the API Gateway destination (local Elasticsearch database) events configuration and the HTTPS proxy configuration incl. proxy bypass addresses.

The Transaction logging global policy is configured differently on the four environments:

| Environment | Configuration of Transaction logging global policy |
| ------ | ------ |
| DEV | Always (on success and on failure) incl. HTTP headers and payloads |
| BUILD | Always (on success and on failure) incl. HTTP headers and payloads |
| STAGE | Always (on success and on failure) excl. HTTP headers and payloads |
| PROD | Always (on success and on failure) excl. HTTP headers and payloads |

More configuration assets can be added later.

## APITest.json Postman test collection

The next common scenario for an API developer is to assert the changes made to the APIs do not break their customer scenarios. This is achieved using Postman test collections, cf. https://learning.postman.com/docs/getting-started/introduction/. In a Postman test collection, the developer can group test requests that should be executed against the API under test every time a change is to be propagated to STAGE and to PROD. The collection can be defined and executed in a local instance of the Postman REST client, cf. https://learning.postman.com/docs/sending-requests/intro-to-collections/. The requests in a test collection should include scripted test cases asserting that the API response is as expected (response status, payload elements, headers etc.), cf. https://learning.postman.com/docs/writing-scripts/test-scripts/. Test scripts can also extract values from the response and store them in Postman variable for later use, https://learning.postman.com/docs/sending-requests/variables/. For example, the first request might request and get an OAuth2 access token and store it in a Postman variable; later requests can use the token in the variable for authenticating against their API. Test collections can even define request workflows including branches and loops, cf. https://learning.postman.com/docs/running-collections/building-workflows/. The automatic execution of Postman collections can be tested in the Postman REST client itself, cf. https://learning.postman.com/docs/running-collections/intro-to-collection-runs/.

Each API project must include one Postman test collection under the name APITest.json in its root folder. This test collection will be executed automatically on the BUILD environment for every deployment on STAGE and PROD. It can be created by exporting a test collection in the Postman REST client and storing it directly in the API project's root folder under the name APITest.json.

> Note: The test requests in the Postman collection must use the following environment variables for addressing the API Gateway. Otherwise the requests will not work in the automatic execution on the BUILD environment. Developers can import and use the environment definition for the central DEV environment in the Postman REST client at /environments/dev_environment_hycloud.json.

| Environment variable | README |
| ------ | ------ |
| {{ip}} |  IP address of the API Gateway, must be used in the URL line of the test requests, e.g. https://{{ip}}:{{port}}/gateway/SwaggerPetstore/1.0/pet/123 |
| {{port}} |  Port number of the API Gateway, must be used in the URL line of the test requests, e.g. https://{{ip}}:{{port}}/gateway/SwaggerPetstore/1.0/pet/123 |
| {{hostname}} | Hostname of the API Gateway, must be used in the Host header of the test requests, e.g. Host: {{hostname}} |

The apis folder contains sample API projects with the following test collections:

### petstore

The petstore test collection sends POST, GET and DELETE requests against the SwaggerPetstore API. It contains tests validating the response code and the petId returned in the response body.

### petstore-versioning

The petstore-versioning test collection invokes POST, GET and DELETE requests for both API versions.

### postman-echo

The postman-echo test collection sends POST, GET and DELETE requests against the Postman Echo API. It contains tests validating the response code and the echoed request elements (payload and query parameter) in the response body.

### multiple-tenants

The multiple-tenants test collection invokes all six APIs for Tenant1 and Tenant2, respectively. The Postman Echo API requests contain tests for the correct tenant query parameter echoed in the response body.

### postman-echo-oauth2

The postman-echo-oauth2 test collection invokes the API Gateway pub.apigateway.oauth2/getAccessToken service to retrieve an OAuth2 token. It stores the token in a Postman variable and uses it in the subsequent POST, GET and DELETE requests against the API.

## Pipelines for API projects

The key to proper DevOps is continuous integration and continuous deployment. Organizations use standard tools such as Jenkins and Azure to design their integration and assuring continuous delivery.

The API Gateway Staging solution includes ten Azure DevOps build pipelines for deploying API projects from the Azure DevOps repository to DEV, STAGE and PROD environments. In each pipeline, the API Gateway assets configured in the API project will be imported on the BUILD environment (after cleaning it from remnants of the last deployment). For a deployment to STAGE and PROD, it will then execute the API tests configured in the API project's APITest.json Postman test collection. If one of the tests fail, the deployment will be aborted. (No tests will be executed for deployments to DEV.)

For a deployment to STAGE and PROD, the pipeline will now manipulate the assets on the BUILD environment (using API Gateway's own APIs) to prepare them for the target environment:
- All applications with names not ending with _STAGE or _PROD, respectively, will be removed
- The remaining applications will be unsuspended (if necessary) to make sure they can be used on the target environment
- Three API tags will be added to every API indicating the build ID, the build name and the pipeline name. These tags can later be used in the API Gateway UI on the target environments to understand when and how (and by whom) every API was promoted to the environment

More manipulations or tests (e.g., enforcement of API standards) can be added later.

Finally, the manipulated API Gateway assets will be exported from the BUILD environment and imported on the target environment (DEV, STAGE or PROD).

> Note: If the imported assets already exist on the target environment (i.e., assets with same IDs), they will be overwritten for the following asset types: APIs, policies, policy actions, applications, scope mappings, users, groups and teams. Existing aliases (and assets of any other types, like configuration items) will not be overwritten, so we can configure different alias values and other settings on different environments.

Every pipeline will publish the following artifacts:
- BUILD_import: The API Gateway asset archive (ZIP file) containing the assets initially imported on the BUILD environment
- BUILD_export_for_DEV, BUILD_export_for_STAGE, BUILD_export_for_PROD: The API Gateway asset archive (ZIP file) containing the assets initially exported from the BUILD environment (after manipulations)
- DEV_import, STAGE_import, PROD_import: The API Gateway asset archive (ZIP file) containing the assets imported on DEV/STAGE/PROD. These artifacts should be identical with BUILD_export_for_DEV, BUILD_export_for_STAGE, BUILD_export_for_PROD, respectively.
- test_results: The results of the Postman tests in junitReport.xml form

These artifacts will be stored by Azure DevOps for some time. They will enable auditing and bugfixing of pipeline builds.

In addition to that, the test results are published into the Azure DevOps test results framework.

All pipelines must be triggered manually by clicking on `Queue`. No triggers are defined to start the pipelines automatically.

> Note: Only one API Gateway Staging pipeline may run at one point in time. Parallel running builds might interfere while using the BUILD environment at the same time. Before starting an API Gateway Staging pipeline, make sure that there is no API Gateway Staging pipeline currently executing. If you want to promote an API to STAGE and PROD, use the wm_test_apigw_staging_deploy_to_stage_and_prod pipeline (or wm_test_apigw_staging_deploy_to_stage_and_prod_separate_jobs) instead of queuing a STAGE build and a PROD build in one go.

The API Gateway Staging solution was developed for Azure DevOps Server 2019. This version offers no simple way to prevent parallel invocations of build pipelines. In later versions, this could be accomplished using Environments and Exclusive Locks.

Each pipeline comes in two versions: In the basic version, all steps are executed in one job on one agent. The agent must be able to access the API Gateway BUILD environment and the target environment (DEV/STAGE/PROD). In the extended version of the pipeline, the steps are divided into separate jobs which can be executed on different agents. Each job only contains steps connecting the agent with one API Gateway (either BUILD or DEV/STAGE/PROD). This version of the pipeline can be executed in distributed deployments in which different agents must be used for accessing the different API Gateway environments.

### wm_test_apigw_staging_deploy_to_stage and wm_test_apigw_staging_deploy_to_stage_separate_jobs

This pipeline will propagate the APIs and other API Gateway assets in the selected API project to the STAGE environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see below. By default, the pipeline will import the HEAD of the selected branch |
| apiProject | Case-sensitive name of the API project to be propagated |

### wm_test_apigw_staging_deploy_to_prod and wm_test_apigw_staging_deploy_to_prod_separate_jobs

This pipeline will propagate the APIs and other API Gateway assets in the selected API project to the PROD environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see below. By default, the pipeline will import the HEAD of the selected branch |
| apiProject | Case-sensitive name of the API project to be propagated |

### wm_test_apigw_staging_deploy_to_stage_and_prod and wm_test_apigw_staging_deploy_to_stage_and_prod_separate_jobs

This pipeline will propagate the APIs and other API Gateway assets in the selected API project to the STAGE environment and then to the PROD environment. It will execute all tasks (including the tests and the target-specific preparation of assets on BUILD) twice - once for the STAGE target environment and once for the PROD target environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see below. By default, the pipeline will import the HEAD of the selected branch |
| apiProject | Case-sensitive name of the API project to be propagated |

### wm_test_apigw_staging_deploy_to_dev and wm_test_apigw_staging_deploy_to_dev_separate_jobs

This pipeline will import the APIs and other API Gateway assets in the selected API project to the DEV environment. It will not execute any tests, and it will not prepare the assets for the target environments (no deletion of applications, no unsuspending of applications, no API tagging). The purpose of this pipeline is to reset the DEV environment to a defined earlier state.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see below. By default, the pipeline will import the HEAD of the selected branch |
| apiProject | Case-sensitive name of the API project to be propagated |

### wm_test_apigw_staging_deploy and wm_test_apigw_staging_deploy_separate_jobs

This pipeline will promote/import the APIs and other API Gateway assets in the selected API project to the selected target environment. No tests or asset manipulation if DEV is selected.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see below. By default, the pipeline will import the HEAD of the selected branch |
| apiProject | Case-sensitive name of the API project to be propagated |
| target_type | DEV, STAGE or PROD |

### Selecting a specific commit to be deployed

When queuing a build pipeline, you can select the specific commit that should be checked out on the build agent, i.e., the configuration of the API Gateway assets to be imported to the BUILD environment. You have to provide the commit's full SHA which can be found out like this:
- In the repository history identify the selected commit and click on ``More Actions...``

![GitHub Logo](/images/More_Actions.png)

- Select ``Copy full SHA``

![GitHub Logo](/images/Copy_full_SHA.png)

- Go back to the pipeline and click on ``Queue``. Paste the value from the clipboard into the Commit form entry field

![GitHub Logo](/images/Paste.png)

![GitHub Logo](/images/SHA_pasted.png)

> Note: It will not work with the commit ID displayed in the UI. You have to use the "full SHA".

### Drop-down list for apiProject and target_type

In later versions of Azure DevOps Server, it will be possible to configure the apiProject as pipeline parameter (vs. pipeline variable). It will then be possible to configure a drop-down list which lets the user select the API project to be deployed from a configurable list of candidates which will be more convenient and less error-prone than having to type the full name of the API project correctly in the form entry field.

The same applies to the target_type variable in the generic wm_test_apigw_staging_deploy pipeline. In later versions of Azure DevOps Server, the text entry field can be replaced by a drop-down list with values DEV, STAGE and PROD to select from.

## Example 1: Change an existing API

Let's consider this example:

  - An API developer wants to make a change to the petstore API. All of the APIs of the organization are available in VCS in the /apis folder. This flat file representation of the APIs should be converted and imported into the developer's local development API Gateway environment or the central DEV environment for changes to be made. The developer uses the /bin/gateway_import_export_utils.bat Windows batch script to do this and import this API (and related assets like applications) to the local development environment or the central DEV environment.

  ```sh 
bin>gateway_import_export_utils.bat --importapi --api_name petstore --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
  ```

  - Alternatively, this could also be achieved for the central DEV environment by executing the wm_test_apigw_staging_deploy_to_dev pipeline for the petstore API project.

  - The API Developer makes the necessary changes to the petstore API on the local development environment or the central DEV environment. 

  - The API developer needs to ensure that the change that was made does not cause regressions. For this, the user needs to run the set of function/regression tests over his change in Postman REST client before the change gets propagated to the next stage.

  - Now this change made by the API developer has to be pushed back to the VCS system such that it propagates to the next stage. The developer uses the /bin/gateway_import_export_utils.bat Windows batch script to prepare this, export the configured API Gateway artifacts for the API project from the the local development environment or the central DEV environment and store the asset definitions to the local repository /apis folder. This can be done by executing the following command.

  ```sh 
bin>gateway_import_export_utils.bat --exportapi --api_name petstore --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
  ```

  - If the developer made any changes to the Postman test collection in the Postman REST client, he/she would now have to export the collection and store it under APITest.json in the API project root folder.

  - Optional, but highly recommended: The developer creates a new feature branch for the change in the VCS.
  
  - After this is done, the changes from the developer's local repository are committed to the VCS.

  - Someone will now propagate the changes by publishing the API project from the feature branch (or the master branch if no feature branch was created) to STAGE using the wm_test_apigw_staging_deploy_to_stage pipeline.

  - The changed API can now be tested on STAGE environment.

  - After successful testing, someone can now merge the feature branch into the master branch and propagate the changes by publishing the API project from the master branch to PROD using the wm_test_apigw_staging_deploy_to_prod pipeline.

## Example 2: Create a new API in an existing API project

Let's consider this example:

  - An API developer wants to create a new API and add it to an existing API project. The developer would first have to update the API Gateway artifacts of the existing API project on the local development environment or the central DEV environment. The developer uses the /bin/gateway_import_export_utils.bat Windows batch script to do this and import the existing API project (and related assets like applications) to the local development environment or the central DEV environment.

  ```sh 
bin>gateway_import_export_utils.bat --importapi --api_name petstore --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
  ```

  - Alternatively, this could also be achieved for the central DEV environment by executing the wm_test_apigw_staging_deploy_to_dev pipeline for the petstore API project.

  - The developer would then create the new API on the local development environment or the central DEV environment.

  - The developer would then import the API project's collection of function/regression tests from the APITest.json file into his/her local Postman REST client and add requests and tests for the new API.

  - The developer will now have to add the ID of the new API to the export_payload.json file in the root folder of the existing API project. The API ID can be extracted from the URL of the API details page in the API Gateway UI.

  - Now this change made by the API developer has to be pushed back to the VCS system such that it propagates to the next stage. The developer uses the /bin/gateway_import_export_utils.bat Windows batch script to prepare this, export the configured API Gateway artifacts for the API project from the the local development environment or the central DEV environment and store the asset definitions to the local repository /apis folder. This can be done by executing the following command.

  ```sh 
bin>gateway_import_export_utils.bat --exportapi --api_name petstore --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
  ```

  - The developer would now export the Postman test collection in the Postman REST client and store it under APITest.json in the API project root folder.

  - Optional, but highly recommended: The developer creates a new feature branch for the change in the VCS.
  
  - After this is done, the changes from the developer's local repository are committed to the VCS.

  - Someone will now propagate the changes by publishing the API project from the feature branch (or the master branch if no feature branch was created) to STAGE using the wm_test_apigw_staging_deploy_to_stage pipeline.

  - The new API can now be tested on STAGE environment.

  - After successful testing, someone can now merge the feature branch into the master branch and propagate the changes by publishing the API project from the master branch to PROD using the wm_test_apigw_staging_deploy_to_prod pipeline.

## Example 3: Create a new API in a new API project

Let's consider this example:

  - An API developer wants to create a new API and add it to a new API project. The developer would create the new API on the local development environment or the central DEV environment.

  - The developer would then create a new collection of function/regression tests for the API project in the local Postman REST client with requests and tests for the new API.

  - The developer will now have to create a new API project folder under /apis with a new export_payload.json file including the ID of the new API. The API ID can be extracted from the URL of the API details page in the API Gateway UI.

  - Now the new API has to be committed to the VCS system such that it propagates to the next stage. The developer uses the /bin/gateway_import_export_utils.bat Windows batch script to prepare this, export the configured API Gateway artifacts for the API project from the the local development environment or the central DEV environment and store the asset definitions to the local repository /apis folder. This can be done by executing the following command.

  ```sh 
bin>gateway_import_export_utils.bat --exportapi --api_name new_api --apigateway_url https://daehgcs51570.hycloud.softwareag.com:11000 --apigateway_username hesseth --apigateway_password ***
  ```

  - The developer would now export the Postman test collection in the Postman REST client and store it under APITest.json in the API project root folder.

  - Optional, but highly recommended: The developer creates a new feature branch for the change in the VCS.
  
  - After this is done, the changes from the developer's local repository are committed to the VCS.

  - Someone will now propagate the changes by publishing the API project from the feature branch (or the master branch if no feature branch was created) to STAGE using the wm_test_apigw_staging_deploy_to_stage pipeline.

  - The new API can now be tested on STAGE environment.

  - After successful testing, someone can now merge the feature branch into the master branch and propagate the changes by publishing the API project from the master branch to PROD using the wm_test_apigw_staging_deploy_to_prod pipeline.

## Pipelines for API Gateway configurations

The API Gateway Staging solution includes four Azure DevOps build pipelines for deploying API Gateway configurations from the Azure DevOps repository to DEV, BUILD, STAGE and PROD environments. In each pipeline, the API Gateway assets configured in the environment configuration folder will be imported on the target environment.

Every pipeline will publish the following artifact:
- DEV_configuration, BUILD_configuration, STAGE_configuration, PROD_configuration: The API Gateway asset archive (ZIP file) containing the assets imported on DEV/BUILD/STAGE/PROD

These artifacts will be stored by Azure DevOps for some time. They will enable auditing and bugfixing of pipeline builds.

After importing the API Gateway assets, the configuration pipelines will execute some steps for initializing the API Gateway:
- Configuration of environment-specific loadbalancer URL
- Configuration of environment-specific OAuth2 configuration parameters in the local Authoritzation Server alias (authorization code and access token expiration interval)

All pipelines must be triggered manually by clicking on `Queue`. No triggers are defined to start the pipelines automatically.

### wm_test_apigw_staging_configure_dev

This pipeline will import the API Gateway configuration assets on the DEV environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see above. By default, the pipeline will import the HEAD of the selected branch |

### wm_test_apigw_staging_configure_build

This pipeline will import the API Gateway configuration assets on the BUILD environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see above. By default, the pipeline will import the HEAD of the selected branch |

### wm_test_apigw_staging_configure_stage

This pipeline will import the API Gateway configuration assets on the STAGE environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see above. By default, the pipeline will import the HEAD of the selected branch |

### wm_test_apigw_staging_configure_prod

This pipeline will import the API Gateway configuration assets on the PROD environment.

The following parameters can/must be provided for this pipeline:

| Parameter | README |
| ------ | ------ |
| Branch |  Select the Git branch from which the assets should be imported |
| Commit | Optional: Select the commit from which the assets should be imported. You must provide the commit's full SHA, see above. By default, the pipeline will import the HEAD of the selected branch |

# Implementation notes

## Pipelines for API projects

The definitions for the ten API Gateway Staging pipelines for API projects can be found in the /pipelines folder. All ten pipelines are using central pipeline templates defined in api-build-template.yml, api-deploy-template.yml, store-build-template.yml and retrieve-build-template.yml for the actual work:

| Template | README |
| ------ | ------ |
| api-build-template.yml | Includes all steps for preparing the deployable on the BUILD environment (including import, test execution, asset manipulation and export) |
| api-deploy-template.yml | Includes all steps for importing the deployable on the DEV/STAGE/PROD environment |
| store-build-template.yml | Stores the deployable in Artifactory |
| retrieve-build-template.yml | Retrieves the deployable from Artifactory |

The basic versions of the pipelines invoke only api-build-template.yml and api-deploy-template.yml sequentially in one job on one agent. There is no need for storing the build result in Artifactory.

The extended versions of the pipelines invoke api-build-template.yml and store-build-template.yml in one job on one agent, and then retrieve-build-template.yml and api-deploy-template.yml in another job (potentially) on another agent.

In order to transport the deployable from the build job to the deploy job, it is stored in Artifactory. In later versions of Azure DevOps Server, it will be possible to use Build Artifacts or Pipeline Artifacts for automatically transporting the build result from one job to the next without having to use an external repository.

wm_test_apigw_staging_deploy_to_stage_and_prod invokes the pipeline templates api-build-template.yml and api-deploy-template.yml twice - once for STAGE and once for PROD - in one job on one agent.

wm_test_apigw_staging_deploy_to_stage_and_prod_separate_jobs invokes the pipeline templates api-build-template.yml, store-build-template.yml, retrieve-build-template.yml and api-deploy-template.yml twice - once for STAGE and once for PROD - in four jobs on (potentially) four different agents.

All four pipeline templates need the following parameters to be set in the calling pipeline:

| Parameter | README |
| ------ | ------ |
| apiProject | Case-sensitive name of the API project to be propagated |
| build_environment | Name of the environment definition file in /environments folder for the BUILD environment, e.g., build_environment_hycloud.json |
| target_environment | Name of the environment definition file in /environments folder for the target environment, e.g., dev_environment_hycloud.json, stage_environment_hycloud.json or prod_environment_hycloud.json |
| target_type | DEV, STAGE or PROD |
| test_condition | Whether to execute the automatic tests (${{true}} or ${{false}}), should be ${{true}} for STAGE and PROD and ${{false}} for DEV |
| prep_condition | Whether to prepare the API Gateway artifacts for the target environment (${{true}} or ${{false}}), should be ${{true}} for STAGE and PROD and ${{false}} for DEV |

The pipeline templates execute the following major steps:

### api-build-template.yml

| Step | README |
| ------ | ------ |
| Create the API Deployable from the flat representation for API project xxx | Using ArchiveFiles@2 Azure DevOps standard task for creating ZIP archives |
| Delete all APIs, applications, strategies, scopes and aliases on API Gateway BUILD (except for the system aliases "ServiceConsulDefault", "EurekaDefault", "OKTA", "PingFederate" and "local") | Executing the Prepare_BUILD.json Postman collection in /utilities/prepare |
| Import the Deployable To API Gateway BUILD | Executing the ImportAPI.json Postman collection in /utilities/import |
| Run tests on API Gateway BUILD (if test_condition is ${{true}}) | Executing the APITest.json Postman collection in the API project's root folder |
| Prepare assets: Delete all non-STAGE/PROD applications on API Gateway BUILD, unsuspend all remaining applications, add build details as tags to APIs (if prep_condition is ${{true}}) | Executing the Prepare_for_STAGE.json or Prepare_for_PROD.json Postman collection in /utilities/prepare |
| Export the Deployable from API Gateway BUILD | Using a bash script calling curl to invoke the API Gateway Archive API |

### api-deploy-template.yml

| Step | README |
| ------ | ------ |
| Import the Deployable To API Gateway DEV/STAGE/PROD | Executing the ImportAPI.json Postman collection in /utilities/import |

### store-build-template.yml

| Step | README |
| ------ | ------ |
| Artifactory Build Upload | Using ArtifactoryGenericUpload@2 Artifactory plug-in task |

### retrieve-build-template.yml

| Step | README |
| ------ | ------ |
| Artifactory Build Download | Using ArtifactoryGenericDownload@3 Artifactory plug-in task |

The status and logs for each step can be inspected on the build details page in Azure DevOps Server. The imported API Gateway archives (import to BUILD and import to DEV/STAGE/PROD) and the test results can be inspected by clicking on `Artifacts`. The test results can be inspected in the `Tests` tab.

The Postman collections are executed using the Postman command-line execution component Newman, cf. https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman/.

## Pipelines for API Gateway configurations

The definitions for the four API Gateway Staging pipelines for API Gateway configurations can also be found in the /pipelines folder. All four pipelines are using a central pipeline template defined in api-configure-template.yml for the actual work.

The pipeline template needs the following parameters to be set in the calling pipeline:

| Parameter | README |
| ------ | ------ |
| type | Case-sensitive name of the environment to be configured or updated (DEV, BUILD, STAGE or PROD) |
| environment | Name of the environment definition file in /environments folder for the target environment, e.g., dev_environment_hycloud.json, build_environment_hycloud.json, stage_environment_hycloud.json or prod_environment_hycloud.json |

The pipeline template executes the following major steps:

| Step | README |
| ------ | ------ |
| Create the API Deployable from the flat representation for DEV/BUILD/STAGE/PROD configuration | Using ArchiveFiles@2 Azure DevOps standard task for creating ZIP archives |
| Import the Deployable To API Gateway DEV/BUILD/STAGE/PROD | Executing the ImportConfig.json Postman collection in /utilities/import |
| Initialize API Gateway DEV/BUILD/STAGE/PROD | Executing the Initialize_DEV.json, Initialize_BUILD.json, Initialize_STAGE.json or Initialize_PROD.json Postman collection in /utilities/initialize |

## Variable group

All pipelines are using the variables in the wm_test_apigw_staging variable group.

| Variable | README |
| ------ | ------ |
| exporter_user | User for exporting assets from API Gateway, e.g., Exporter. The user must have the "Export assets" privilege |
| exporter_password | The API Gateway password for the exporter user |
| importer_user | User for importing assets in API Gateway, e.g., Importer. The user must have the "Import assets" privilege |
| importer_password | The API Gateway password for the importer user |
| preparer_user | User for preparing assets on API Gateway BUILD, e.g., Preparer. The user must have the "Manage APIs", "Activate / Deactivate APIs", "Manage applications", "Manage aliases" and "Manage scope mapping" privileges |
| preparer_password | The API Gateway password for the preparer user |
| initializer_user | User for initializing the API Gateway, e.g., Initializer. The user must have the "Manage general administration configurations" privilege |
| initializer_password | The API Gateway password for the initializer user |
| agent_pool_dev, agent_pool_build, agent_pool_stage, agent_pool_prod | The Azure DevOps agent pool to be used for jobs accessing the DEV, BUILD, STAGE or PROD environments, respectively, e.g., Azure Pipelines for Microsoft-hosted agents |
| agent_pool_common | The Azure DevOps agent pool to be used for jobs accessing multiple API Gateway environments, e.g., Azure Pipelines for Microsoft-hosted agents |
| agent_pool_vmImage | The VM image for Microsoft-hosted agents in the Azure Pipelines pool, e.g., ubuntu-latest. Leave blank for self-hosted agents |
| artifactoryService | Name of the Artifactory service connection in Azure |
| artifactoryFolder | Name of the Artifactory base-folder (repository) |
| environment_dev, environment_build, environment_stage, environment_prod | Name of the JSON file in the /environments folder for the API Gateway instances to be used for DEV, BUILD, STAGE and PROD, e.g., dev_environment_hycloud.json, build_environment_hycloud.json, stage_environment_hycloud.json or prod_environment_hycloud.json, respectively |

## Environment configurations

The environments used in the API Gateway Staging solution are configured in the /environments folder. For each environment (DEV, BUILD, STAGE and PROD), there is a Postman environment definition JSON file, for example:

### build_environment_hycloud.json

```
{
  "id": "f313be5f-2640-22f7-36b6-3e79bba3c9e2",
  "name": "BuildEnvironment Demo",
  "values": [
    {
      "enabled": true,
      "key": "hostname",
      "value": "daehgcs51571.hycloud.softwareag.com",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "ip",
      "value": "daehgcs51571.hycloud.softwareag.com",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "port",
      "value": "11000",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "insecureflag",
      "value": "--insecure",
      "type": "text"
    }
  ],
  "timestamp": 1587036498482,
  "_postman_variable_scope": "environment",
  "_postman_exported_at": "2020-04-16T11:32:40.730Z",
  "_postman_exported_using": "Postman/5.5.5"
}
```

Each environment must include values for the hostname, ip, port and insecureflag environment variables.

| Environment variable | README |
| ------ | ------ |
| {{ip}} |  IP address of the API Gateway |
| {{port}} |  Port number of the API Gateway |
| {{hostname}} | Hostname of the API Gateway |
| {{insecureflag}} | Set to --insecure if the API Gateway server does not provide SSL server certificate, otherwise leave blank |

These environment variables are used in the utilities Postman collections and in the "Export the Deployable from API Gateway BUILD" step of the pipeline template (bash script with curl command), and they must also be used in the APITest.json Postman test collections in the API projects.

They are loaded automatically when the Postman collections are executed in the Azure DevOps pipelines, and they can (and should) also be used in the Postman REST client for local API testing and test developments.

The separate configuration of IP address and hostname is necessary in order to support cases in which the agent might not be able to find the API Gateway server by its hostname.