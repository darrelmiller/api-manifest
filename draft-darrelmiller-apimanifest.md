---
title: "API Manifest"
category: info

docname: draft-darrelmiller-apimanifest-latest
submissiontype: independent  # also: "IETF","independent", "IAB", or "IRTF"
number:
date:
consensus: false
v: 3
# area: ART
# workgroup: WG Working Group
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
  latest: "https://darrelmiller.github.io/api-manifest/draft-darrelmiller-apimanifest.html"

author:
 -
    fullname: Darrel Miller
    organization: Microsoft
    email: darrel.miller@microsoft.com

normative:

informative:


--- abstract
This document defines an "api manifest" as a way to capture the dependencies that an application has on HTTP APIs. It contains characteristics of those dependencies including links to API descriptions, specifics of the types of HTTP API requests made by the application and related authorization information.

--- middle

# Introduction

Applications frequently rely on HTTP APIs to provide functionality to users. Currently, there are limited options for developers to be able to describe those dependencies and the options that do exist do not have sufficiently detailed information to enable some scenarios. By contrast, there does exist declarative, machine readable files, that describe the dependencies that applications have on code libraries and packages. These files have enabled an ecosystem of tooling related to checking, adding, updating and reporting on dependencies. This specification defines a machine processable format to enable a programming language agnostic tooling ecosystem be built around the dependencies applications have on HTTP APIs.

An API manifest such as described in this document could enable a number of scenarios:

- generate client code that can be used to access those resources
- identify the scopes or roles that an application must be granted to be able to access those resources.
- use as input to Software Bill Of Materials documents to support secure supply chain efforts.
- perform dependency checks for updates to APIs in a similar way Dependabot tooling does for package dependencies
- provide security alters for APIs that have announced discovered vulnerabilities

It is common for the the person who consents an application to be used, and therefore access data and functionality of HTTP APIs, not be capable of reviewing application source code to understand the details of what an application does. The API manifest can be used to create admin friendly descriptions of application capabilities to simplify the process of application consent.

There are no guarantees that an API manifest accurately describes that capabilities and dependencies of an application. There remains an element of trust. It is not in itself a security artifact. However, it can play an role in enabling tooling as part of a secure supply chain.

By creating an API manifest format independent of the application programming language tooling that consumes the API manifest can be created in any programming language. Language specific tooling could be created to generate API manifests by introspecting application code.  Tooling could be created to produce API manifests to support  design first methodologies, or integration centric scenarios.


# Schema

## Api Manifest {#api-manifest}

The Api Manifest document contains information about a target application that consumes HTTP APIs. The canonical model for an API Manifest document is a JSON object. When serialized as JSON it can be identified by the `application/api-manifest` media type.

An API manifest document contains a `appPublisher` property that has a value described by the {{publisher}} and an array of zero or more {{api-dependency}} objects.

## Publisher Object {#publisher}

The publisher object contains a `name` property that is a JSON string. This string contains a value representing the organization or individual responsible for the application that this api manifest belongs to.  The `contactEmail` provides a mechanism to communicate information to the publisher.

## Api Dependency Object {#api-dependency}

Each Api dependency object represents a HTTP API that the target application consumes. The `apiDescriptionUrl` references an API description document such as an OpenAPI description. The `auth` property contains the requirements for the target application to authorize a call to the HTTP API. The `requests` property contains a array of `requestInfo` objects.

## Authorization Details Object {#authDetails}

## Security Scheme Object {#securityScheme}

## Request Info {#requestInfo}

~~~ cddl
{::include manifest.cddl }
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
                    "delegated": ["user.read"],
                    "application": []
                }
            },
            "requests": [{
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

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
