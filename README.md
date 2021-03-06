# ApiConfigTool

Java app that uses the Exchange, API Manager 2.0 and Access Manager APIs to perform a complete configuration of an API for API Management...meant to be used in Maven using exec-maven-plugin. This is a fork of the ApiDeployTool project.

## Overview

The ApiConfigTool was created primarily to provide a coding example for how to use the various Anypoint Platform APIs to automate the API registration for deploying an API within a Maven script. Note that the ApiConfigTool will re-use pieces that have already been configured, for instance, if the API is already defined in Exchange, it will use that entry and continue running the rest of the steps.

The ApiConfigTool can be used in its current form to register APIs, although it makes many assumptions about naming conventions and expected usage of the API that may not fit with a specific set of customer requirements. When finished running, the specified API will be created in Exchange and an instance for the specified environment will be created in API Manager. In addition to the API instance, a client application will be registered for the API which is intended to be used for inter-API governance.

The API manager autodiscovery values will be stored in the src/main/resources directory in the environment's -config.properties (or -config.yaml) depending upon which file exists in the directory. This results in an environment specific configuration file that can be used to identify the API Manager defined characteristics of the API. Here is a sample of an DEV-config.properties file contents:

```
\#Mon Jan 07 13:18:22 PST 2019

api.id=15607288
api.name=groupId\:50228d0d-2c1f-4548-9c8a-90c0ebb480b9\:assetId
\:50228d0d-2c1f-4548-9c8a-90c0ebb480b9_pdd-exp_1.0.0
api.version=1.0.0\:15607288
my.client_id=da183b8d12cf4c06814a81a5574e95bb
my.client_name=PDD-EXP_SANDBOX
my.client_secret=39e75F05e151453C86dAFd1EB8dC14BB
```

Here is an example of a yaml file:

```
api:
  id: "15607289"
  name: 
"groupId:50228d0d-2c1f-4548-9c8a-90c0ebb480b9:assetId:50228d0d-2c1f-4548-9c8a-90c0ebb480b9_pdd-exp_1.0.0"
  version: "1.0.0:15607289"
my:
  client_name: "PDD-EXP_SANDBOX"
  client_id: "da183b8d12cf4c06814a81a5574e95bb"
  client_secret: "39e75F05e151453C86dAFd1EB8dC14BB"
```

This document will explain the current tool and how it uses the command line values to perform the registration.

## The configureProjectResourceFile Command

The ApiConfigTool is a java program. To register a Mule 3 API use this command:

```
java -jar target/ApiConfigTool.jar configureProjectResourceFile myAnypointUser MyAnypointPassword "businessGroupName" myApi v1 "myEnvironmentName" my-policies.json my‑clients.json sla-tiers.json
```

**configureProjectResourceFile** is the operation to execute. This operation configures the API in Anypoint Exchange and creates the API Manager instance for the environment. If the API Exchange or API Manager instance already exists, then the current settings are used.

**myAnypointUser** is the Anypoint user that will be used to perform all the registration steps. Note that this tool will create any client applications that are listed in the my‑clients.json file. In doing so, the user specified here becomes the owner of record for the application and its client credentials...no other users will be able to see these applications except the master org owner. Using a consistent user name here is important in order to have consistent visibility of the credentials for all automated API registrations.

**MyAnypointPassword** is the password for the user specified above.

**businessGroupName** is the Anypoint business group Exchange where the API will be registered.

**myApi** is the name of the API that will be registered in Exchange.

**v1** is the version to use in Exchange. If the version of the API already exists in Exchange, then the existing version will be used.

**myEnvironmentName** is the environment within the business group that the API instance will be registered into.

**my-policies.json** is a file that contains the list of policies that will be applied to the API Instance being registered. See the following section on "Defining Policies" for more information on this file.

**sla-tiers.json** is a file that contains the list of SLA tiers to apply to the API Instance. See the section on "Defining SLA tiers" for more information on this file.

The file name specified here must either be in the current running directory or on the Java classpath as a resource file. The default file is distributed in the project as client‑credentials‑policy which applies client credential enforcement using HTTP headers client\_id and client\_secret.

**my-clients.json** is a file that contains the list of client applications that will be registered as consumers of the API. As noted earlier, these client applications will be created if they do not already exist. Do to the current structure of Anypoint Access Management, the client application will fail if it already exists in another part of the Master Org that the specified Anypoint user performing the registration does not have access to. See the following section on "Defining a Client List" for more information on this file contents.

The file name specified here must either be in the current running directory or on the Java classpath as a resource file. The default file is distributed in the project as empty‑client‑access‑list which is an empty list resulting in no client applications being registered to use the API Instance.

## The mule4ConfigureProjectResourceFile Command

Similarly, register an API running in Mule 4 with this command:

```
java -jar target/ApiConfigTool.jar mule4ConfigureProjectResourceFile myAnypointUser MyAnypointPassword "businessGroupName" myApi v1 "myEnvironmentName" my-policies.json my‑clients.json sla-tiers.json
```

## Defining Policies

The policies are defined in a file that is named when the ApiConfigTool is executed. The file that can be in current directory where ApiConfigTool is running, or in the Java classpath. The current directory is searched first.

The file is in json format and lists the policies that should be applied to the API Instance. There are several examples of policy definitions in the resources directory of the ApiConfigTool:

- client-credentials-policy
- ip-whitelist-policy
- simple-basic-auth-policy

To determine more policies, use the "Developer view" of a Chrome browser when adding policies through API Management to determine what properties and names to use. These are not really documented anywhere.

Here is an example of client-id-enforcement policy for Mule 3:

```
[{
        "policyTemplateId": "client-id-enforcement",
        "configurationData": {
                "credentialsOrigin": "customExpression",
                "clientIdExpression": "#[message.inboundProperties['client_id']]",
                "clientSecretExpression": "#[message.inboundProperties['client_secret']]"
        }
}]
```

Here is an example of basic authentication with a simple authentication manager for Mule 3:

```
[{
        "policyTemplateId": "simple-security-manager",
        "configurationData": {
                "username": "username",
                "password": "password"
        }
}, {
        "policyTemplateId": "http-basic-authentication",
        "configurationData": {}
}]
```
Note that policies for Mule 3 runtimes are different from Mule 4 policies. Here is an example of client-id-enforcement policy for Mule 4:

```
[	{
		"configurationData": {
			"credentialsOriginHasHttpBasicAuthenticationHeader": "customExpression",
			"clientIdExpression": "#[attributes.headers['client_id']]",
			"clientSecretExpression": "#[attributes.headers['client_secret']]"
		},
		"policyTemplateId": 294,
		"groupId": "68ef9520-24e9-4cf2-b2f5-620025690913",
		"assetId": "client-id-enforcement",
		"assetVersion": "1.1.2"
	}
]
```
 
## Defining a Client List

The client applications are defined in a file that can be in the current directory where ApiConfigTool is running, or in the Java classpath. The current directory is searched first.

The file is in json format and lists the client applications that should be created (if they don't already exist) and then registered to use the API Instance.

The registration assumes no SLA's are configured for the API. Here is an example of registering two applications:

```
[{
                "applicationName": "auto-api-registration"
        },
        {
                "applicationName": "my-web-app"
        }
]
```

## Defining SLA Tiers

SLA Tiers are defined in a file that is named when the ApiConfigTool is executed. The file that can be in current directory where ApiConfigTool is running, or in the Java classpath. The current directory is searched first.

The file is in json format and lists the SLA Tiers that should be applied to the API Instance. Two SLA Tiers are predefined:

- empty-sla-tiers-list
- manual-approval-required

To determine more SLA Tiers, use the "Developer view" of a Chrome browser when adding SLA Tiers through API Management to determine what properties and names to use. These are not really documented anywhere.

Here is an example of manual-approval-required SLA for Mule 4:

```
[{
	"status": "ACTIVE",
	"autoApprove": false,
	"limits": [{
		"visible": true,
		"timePeriodInMilliseconds": 1000,
		"maximumRequests": 1000
	}],
	"name": "manual-approval-required"
}]
```

## Revisions
|version|who|description|
| --- | --- | --- |
|1.0.4|pdd|Jackson databind security release fix. Add my.client_ variables to environment configuration file (DEV-config.properties for instance).|	
|1.0.5|pdd|create a -config.properties if no other config file is found|	
|1.0.7|pdd|add SLA Tiers feature|

	
