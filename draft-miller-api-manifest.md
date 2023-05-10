---
title: "API Manifest"
category: std
docname: draft-miller-api-manifest-latest
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
  latest: "https://darrelmiller.github.io/api-manifest/draft-miller-apimanifest.html"

author:
 -
    ins: D. Miller
    name: Darrel Miller
    organization: Microsoft
    email: darrel.miller@microsoft.com

normative:

informative:


--- abstract
This document defines an "api manifest" as a way to declare the dependencies that an application has on HTTP APIs. It contains characteristics of those dependencies including links to API descriptions, specifics of the types of HTTP API requests made by the application and related authorization information.

--- middle

# Introduction

Applications frequently rely on HTTP APIs to provide functionality to users. Currently, there are limited options for developers to be able to describe those dependencies and options that do exist do not have sufficiently detailed information to enable some of the desired scenarios. By contrast, there does exist declarative, machine readable files, that describe the dependencies that applications have on code libraries and packages. These files have enabled an ecosystem of tooling related to checking, adding, updating and reporting on dependencies. This specification defines a machine processable format to enable a programming language agnostic tooling ecosystem be built around the dependencies applications have on HTTP APIs.

An API manifest such as described in this document could enable a number of scenarios:

- generate a minimal set of client code that can be used to access the specified resources
- define API subsets for API gateways
- identify the scopes or roles that an application must be granted to be able to access those resources
- use as Signed Statement in Trustworthy and Transparent Digigtal Supply Chains
- perform dependency checks for updates to APIs in a similar way Dependabot tooling does for package dependencies
- provide security alerts for APIs that have announced discovered vulnerabilities

It is common for the the person who consents an application to be used, and therefore access data and functionality of HTTP APIs, not be capable of reviewing application source code to understand the details of what an application does. The API manifest can be used to create admin friendly descriptions of application capabilities to simplify the process of application consent.

There are no guarantees that an API manifest accurately describes that capabilities and dependencies of an application. There remains an element of trust. It is not in itself a security artifact. However, it can play an role in enabling tooling as part of a secure supply chain.

By creating an API manifest format independent of the application programming language tooling that consumes the API manifest can be created in any programming language. Language specific tooling could be created to generate API manifests by introspecting application code.  Tooling could be created to produce API manifests to support  design first methodologies, or integration centric scenarios.

# Schema

## Api Manifest {#api-manifest}

The Api Manifest document contains information about a target application that consumes HTTP APIs. The canonical model for an API Manifest document is a JSON object. When serialized as JSON it can be identified by the `application/api-manifest` media type.

An API manifest document contains a `appPublisher` property that has a value described by the publisher {{publisher}} and an array of zero or more Api Dependency {{api-dependency}} objects.

## Publisher Object {#publisher}

The publisher object contains a `name` property that is a JSON string. This string contains a value representing the organization or individual responsible for the application that this api manifest belongs to.  The `contactEmail` provides a mechanism to communicate information to the publisher.

## Api Dependency Object {#api-dependency}

Each Api dependency object represents a HTTP API that the target application consumes. The `apiDescriptionUrl` references an API description document such as an [OpenAPI](https://spec.openapis.org/oas/latest.html) description. The `auth` property contains the requirements for the target application to authorize a call to the HTTP API. The `requests` property contains a array of `requestInfo` objects.

## Authorization Requirements Object {#authReqirements}

The Authorization Requirements object contains information that is required to authorize the application to perform the requests listed in the Api Dependency `requests` property. The `clientId` property is a JSON string value used to identify the application to an OAuth2 authorization server for APIs that use OAuth2 for authorization. The `permissions` property is a JSON object that map a set of security schemes to an array of permission strings required to perform the complete set of requests defined in the Api Dependency {{api-dependency}}. The Api Manifest does not attempt to correlate which permission is required for a specific request. It is assumed that the application must be granted the complete set of permissions in order to perform its function.

## Request Info Object {#requestInfo}

Each Request Info object contains a `uriTemplate` [RFC6570] and a corresponding HTTP `method`. The values are used to identify one or more operations defined in the API description referenced in the Api Dependency{{api-dependency}}. The  `excludes` property when set to `true` can be used to eliminate specific operations included by another Request Info {{requestInfo}}. The `dataClassification` property is a list of URIs used to indicate privacy classifications of the data being transmitted via the HTTP request.

~~~ cddl

apiManifest = {
    applicationName: tstr
    ? publisher: publisher
    apiDependencies : [* apiDependency]
}

; Identification of the application developer / organization
publisher = {
    name: tstr
    contactEmail: tstr
}

;  Declaration of application dependencies on HTTP API
apiDependency = {
    apiDescriptionUrl: tstr
    authorizationRequirements: authorizationRequirements
    requests: [+ requestInfo]
}

; Permissions required by client application for the described dependency
authorizationRequirements = {
    ? clientId: tstr
    ? permissions: {+ securityScheme => [+ tstr]}
}

; Details of a resource request
requestInfo = {
    method: tstr
    uriTemplate: tstr
    ? exclude: bool
    ? dataClassification: [* tstr]
}

~~~

Example:

~~~ json

{
    "publisher": {
        "name": "Alice",
        "contactEmail": "alice@example.org"
    },
    "apiDependencies": [
        {
            "apiDescripionUrl": "https://example.org/openapi.json",
            "auth": {
                "clientId": "some-uuid-here",
                "permissions": {
                    "delegated": [
                        "resourceA.ReadWrite",
                        "resourceB.ReadWrite"
                    ],
                    "application": [
                        "resourceB.Read"
                    ]
                }
            },
            "requests": [
                {
                    "method": "GET",
                    "uriTemplate": "https://example.org/api/resourceA"
                },
                {
                    "method": "GET",
                    "uriTemplate": "https://example.org/api/resourceB"
                }
            ]
        }
    ]
}
~~~

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document document registers the `application/api-manifest+json` media type.

Type name:  application

Subtype name:  api-manifest+json

Required parameters:  n/a

Optional parameters:  n/a

Encoding considerations:  Encoding considerations are identical to those specified for the "application/json" media type.  See [RFC7159].

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
