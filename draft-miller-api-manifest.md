---
title: "API Manifest"
category: std
docname: draft-miller-api-manifest-latest
date: {DATE}
category: info

ipr: trust200902
submissiontype: IETF
v: 3
area: General
wg: Internet Engineering Task Force
keyword:
 - API
 - Http
 - OpenAPI
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "darrelmiller/api-manifest"
  latest: "https://darrelmiller.github.io/api-manifest/draft-miller-api-manifest.html"

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: D. Miller
    name: Darrel Miller
    organization: Microsoft
    email: darrel.miller@microsoft.com

normative:
  URITEMPLATE: RFC6570
  JSON: RFC7159
  RAR: RFC9396

informative:


--- abstract
This document defines an "api manifest" as a way to declare the dependencies that an application has on one or more HTTP APIs. It contains characteristics of those dependencies including links to API descriptions, specifics of the types of HTTP API requests made by the application and related authorization information. It can optionally include information that target specific existing applications to enhance their ability to dynamically interact with an API without a priori knowledge.

--- middle

# Introduction

Applications frequently rely on HTTP APIs to provide functionality to users. Currently, there are limited options for developers to be able to describe those dependencies and the options that do exist do not have sufficiently detailed information to enable some of the desired scenarios. By contrast, there does exist declarative, machine readable files, that describe the dependencies that applications have on code libraries and packages. These files have enabled an ecosystem of tooling related to checking, adding, updating and reporting on dependencies. This specification defines a machine processable format to enable a programming language agnostic tooling ecosystem be built around the dependencies applications have on HTTP APIs.

An API manifest such as described in this document could enable a number of scenarios:

- generate a minimal set of client code that can be used to access the specified resources
- define API subsets for API gateways
- identify the scopes or roles that an application must be granted to be able to access those resources
- use as Signed Statement in Trustworthy and Transparent Digital Supply Chains
- perform dependency checks for updates to APIs in a similar way Dependabot tooling does for package dependencies
- provide security alerts for APIs that have announced discovered vulnerabilities
- describe the capabilities of a skill/plugin for a chat-based system

It is common for the person who consents to an application to be used, and therefore access data and functionality of HTTP APIs, not be capable of reviewing application source code to understand the details of what an application does. The API manifest can be used to create admin friendly descriptions of application capabilities to simplify the process of application consent.

There are no guarantees that an API manifest accurately describes capabilities and dependencies of an application. There remains an element of trust. It is not in itself a security artifact. However, it can play a role in enabling tooling as part of a secure supply chain.

By creating an API manifest format independent of the application programming language tooling that consumes the API manifest can be created in any programming language. Language specific tooling could be created to generate API manifests by introspecting application code.  Tooling could be created to produce API manifests to support design first methodologies, or integration centric scenarios.

Not all applications have static dependencies on HTTP APIs. Products have been build that can dynamically discover how to call APIs based on API descriptions. An API Manifest can be used as an input to these kinds of products to teach them how to consume new APIs at runtime.

# Schema

## Api Manifest {#api-manifest}

The Api Manifest document contains information about an application that consumes HTTP APIs. The canonical model for an API Manifest document is a JSON object. When serialized as JSON it can be identified by the `application/api-manifest` media type.

An API manifest document SHOULD contain a `publisher` property that has a value described by the publisher {{publisher}} and MUST contain a JSON object that contains of zero or more mappings from a string key to Api Dependency {{api-dependency}} objects.  The API Manifest object MUST contain an `applicationName` string property to uniquely identify the application to users of the API Manifest.

## Publisher Object {#publisher}

The publisher object MUST contain a `name` property that is a JSON string. This string contains a value representing the organization or individual responsible for the application that this api manifest belongs to.  The `contactEmail` property MUST provide an email address to communicate information to the publisher of the application being described.

## API Dependency Object {#api-dependency}

Each API dependency object represents an HTTP API that the target application consumes. The API dependency object MAY contain a `apiDescriptionUrl` that references an API description document such as an [OpenAPI](https://spec.openapis.org/oas/latest.html) description. The `apiDeploymentBaseUrl` member MAY contain the base URL to use in combination with request info{{requestInfo}} `uriTemplate` properties. When the base URL is used, it MUST end with a trailing slash.
The combined base URL with the request UriTemplate SHOULD match a resource identified in the API description. The `apiDescriptionVersion` member can contain the version of the API Description used by the application. This member enables tooling to detect if the referenced API description is updated. The `authorizationRequirements` property contains the requirements for the target application to authorize a call to the HTTP API. The `requests` property contains an array of `requestInfo` objects.

## Authorization Requirements Object {#authRequirements}

The Authorization Requirements object contains information that is required to authorize the application to perform the requests listed in the Api Dependency `requests` property. The `clientIdentifier` property is a JSON string value used to identify the application to an OAuth2 authorization server for APIs that use OAuth2 for authorization. The `access` property is a JSON object that has the structure and semantics of the `authorization_details` defined in {{RAR}} that are required to perform the complete set of requests defined in the Api Dependency {{api-dependency}}. The Api Manifest does not attempt to correlate which permission is required for a specific request. It is assumed that the application must be granted the complete set of permissions in order to perform its function.

## Request Info Object {#requestInfo}

Each Request Info object MUST contain a `uriTemplate` {{URITEMPLATE}} and a corresponding HTTP `method`. The values are used to identify an operation defined in the API description referenced in the Api Dependency {{api-dependency}}. If the API Dependency {{api-dependency}} contains a `apiDeploymentBaseUrl` then uriTemplate values that resolve to a relative reference MUST be relative to the `apiDeploymentBaseUrl`. The `dataClassification` property is a list of URIs used to indicate privacy classifications of the data being transmitted via the HTTP request.

Request Info objects can optionally contain a `runtimes` property which is a JSON object that represents a map of Runtime objects {{runtime}}. Each Runtime object provides a set of additional information to guide specifically identified runtime engines about when and how they can make HTTP requests. Each key in the map SHOULD be registered in the IANA API Manifest Runtime registry that is created by this specification.

## Runtime Object {#runtime}

Runtime objects contain three optional properties that are JSON arrays. If the property is present the array MUST contain at least one value.

### constraints property

The `constraints` property has an array of Request Constraint objects {{requestConstraint}} that the runtime engine can use to determine when it is allowed to make the request.

### inputTransforms property

The `inputTranforms` property has an array of Data Transform objects {{dataTransform}} that the runtime engine can use to manipulate input values into a format compatible with the API request.

### outputTransforms property

The `ouputTranforms` property has an array of Data Transform objects {{dataTransform}} that the runtime engine can use to manipulate HTTP response representations into a format suitable for consumption by the runtime engine.

## Request Constraint Object {#requestConstraint}

The Request Constraint Object contains a `type` property that is a JSON String. The value SHOULD be understood by the runtime engine in order to limit when the runtime engine will make a HTTP request to the API. If the property `mandatory` is set to `true` then the runtime engine MUST understand the value of the `type` property and respect the semantics of the constraint.  Additional configuration information can be provided in the `config` property to assist the runtime engine in evaluating the constraint.

## Data Transform Object {#dataTransform}

Data Transform objects contain a `type` property that identifies a data transformation process that the runtime engine SHOULD understand.  The `config` property is used to provide additional information for the runtime engine to perform the transformation of either the API request or response.

## Extensibility

The API Manifest object and API Dependency object can be extended with additional properties. The `extensions` member is a map of properties whose values can be any valid JSON member.

## CDDL Schema

~~~ cddl

apiManifest = {
    applicationName: text
    ? publisher: publisher
    apiDependencies: {* text => apiDependency}
    extensibility
}

; Identification of the application developer / organization
publisher = {
    name: text
    contactEmail: text
}

; Declaration of application dependencies on HTTP API
apiDependency = {
    ? apiDescriptionUrl: text
    ? apiDescriptionVersion: text
    ? apiDeploymentBaseUrl: text
    authorizationRequirements: authorizationRequirements
    requests: [+ requestInfo]
    extensibility
}

; Permissions required by client application for the described dependency
authorizationRequirements = {
    ? clientIdentifier: text
    ? access: [+accessRequest] | [+text]
}

extensibility = (
    ? extensions => {* text => any }
)
accessRequest = {
    type : text ;
    * text => any;
}

; Details of a resource request
requestInfo = {
    method: text
    uriTemplate: text
    ? dataClassification: [* text]
    runtimes : {* text => runtime }
}

runtime = {
    ? constraints: [+ requestConstraint] ; constraints that MUST/SHOULD be met before performing request
    ? inputTransforms: [+ dataTransform]
    ? outputTransforms: [+ dataTransform]
}

requestConstraint = {
    name: text
    mandatory: bool ; This property allows new constraints to be introduced in a non breaking way.
}

dataTransform = {
    type: text;
    config: any;
    examples: [* text]
}

~~~

Example:

~~~ json

{
    "publisher": {
        "name": "Alice",
        "contactEmail": "alice@example.org"
    },
    "apiDependencies": {
        "example": {
            "apiDescriptionUrl": "https://example.org/openapi.json",
            "apiDescriptionVersion": "1.2",
            "apiDeploymentBaseUrl": "https://example.org/",
            "authorizationRequirements": {
                "clientIdentifier": "some-uuid-here",
                "access": [
                    {
                        "type": "delegated",
                        "actions": [
                                    "resourceA.ReadWrite",
                                    "resourceB.ReadWrite"
                                ]
                    },
                    {
                        "type": "application",
                        "actions": [
                                    "resourceB.Read"
                                ]
                    }
                ]
            },
            "requests": [
                {
                    "method": "GET",
                    "uriTemplate": "/api/resourceA"
                },
                {
                    "method": "POST",
                    "uriTemplate": "/api/resourceB",
                    "runtime": {
                        "someWorkflowEngine": {
                            "constraints": [
                                {
                                    "type": "weekendsOnly"
                                }
                            ],
                            "outputTransforms": [
                                {
                                    "type": "xslt",
                                    "config": {
                                        "transformFile": "toHtml.xslt"
                                    }
                                }
                            ]
                        }
                    }
                }
            ]
        }
    }
}
~~~


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document document registers the `application/api-manifest` media type.

Type name:  application

Subtype name:  api-manifest

Required parameters:  n/a

Optional parameters:  n/a

Encoding considerations:  Encoding considerations are identical to those specified for the "application/json" media type.  See {{JSON}}.

Security considerations:  TBD.

Interoperability considerations:  TBD.

Published specification:  This document is the specification for this media type.

Applications that use this media type:

Additional information:

    Magic number(s):  n/a

    File extension(s):  TBD

    Macintosh file type code(s):  n/a

Person & email address to contact for further information:  See Authors' Addresses section.

Intended usage:  COMMON

Restrictions on usage:  n/a

Author:  See Authors' Addresses section.

Change controller:  Internet Engineering Task Force (mailto:iesg@ietf.org).

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

# Appendix
{:numbered="false"}

## Example for Microsoft Graph API
{:numbered="false"}

~~~ json

{
    "publisher": {
        "name": "Alice",
        "contactEmail": "alice@example.org"
    },
    "apiDependencies": {
        "graph": {
            "apiDescriptionUrl": "https://raw.githubusercontent.com/microsoftgraph/msgraph-metadata/master/openapi/v1.0/openapi.yaml",
            "apiDeploymentBaseUrl": "https://graph.microsoft.com/v1.0/",
            "authorizationRequirements": {
                "clientIdentifier": "some-uuid-here",
                "access": [
                    {
                        "type": "openid",
                        "claims": {
                            "scp": {
                                "essential": true,
                                "values": [
                                    "User.Read",
                                    "Mail.ReadWrite.All"
                                ]
                            }
                        }
                    },
                    {
                        "type": "openid",
                        "claims": {
                            "roles": {
                                "essential": true,
                                "values": [
                                    "User.Read.All"
                                ]
                            }
                        }
                    }
                ]
            },
            "requests": [
                {
                    "method": "GET",
                    "uriTemplate": "me"
                },
                {
                    "method": "GET",
                    "uriTemplate": "users/{userId}/messages"
                },
                {
                    "method": "GET",
                    "uriTemplate": "users"
                }
            ]
        }
    }
}
~~~

## Example for dynamic usage
{:numbered="false"}

~~~ json

{
    "publisher": {
        "name": "Alice",
        "contactEmail": "alice@example.org"
    },
    "apiDependencies": {
        "graph": {
            "apiDescriptionUrl": "https://raw.githubusercontent.com/microsoftgraph/msgraph-metadata/master/openapi/v1.0/openapi.yaml",
            "apiDeploymentBaseUrl": "https://graph.microsoft.com/v1.0/",
            "authorizationRequirements": {
                "clientIdentifier": "some-uuid-here",
                "access": [
                    {
                        "type": "openid",
                        "claims": {
                            "scp": {
                                "essential": true,
                                "values": [
                                    "User.Read",
                                    "Mail.ReadWrite.All"
                                ]
                            }
                        }
                    },
                    {
                        "type": "openid",
                        "claims": {
                            "roles": {
                                "essential": true,
                                "values": [
                                    "User.Read.All"
                                ]
                            }
                        }
                    }
                ]
            },
            "requests": [
                {
                    "method": "GET",
                    "uriTemplate": "me"
                },
                {
                    "method": "GET",
                    "uriTemplate": "users/{userId}/messages"
                },
                {
                    "method": "GET",
                    "uriTemplate": "users"
                },
                {
                    "method": "POST",
                    "uriTemplate": "users/{userId}/sendmail",
                    "runtimes": {
                        "powerAutomate": {
                            "constraints": [
                                {
                                    "name": "userConfirmation",
                                    "mandatory": true
                                }
                            ],
                            "inputTransforms": [
                                {
                                    "type": "jsonSchemaForm",
                                    "config": {}
                                },
                            ],
                            "outputTransforms": [
                                {
                                    "type": "adaptiveCard",
                                    "config": {}
                                }
                            ]
                        }
                    }
                }
            ]
        }
    }
}
~~~
