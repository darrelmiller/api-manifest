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

An API manifest that identifies the resources required by an application could be used, along with an API description, to generate client code that can be used to access those resources.  Tooling could also use that information to identify the scopes or roles that an application must be granted to be able to access those resources.  The information contained in an API manifest could be used as input to Software Bill Of Materials documents to support secure supply chain efforts.

It is common for the the person who consents an application to be used, and therefore access data and functionality of HTTP APIs, not be capable of reviewing application source code to understand the details of what  an application does. The API manifest can be used to create admin friendly descriptions of application capabilities to simplify the process of application consent.

There are no guarantees that an API manifest accurately describes that capabilities and dependencies of an application. There remains an element of trust. It is not in itself a security artifact. However, it can play an role in enabling tooling as part of a secure supply chain.

By creating an API manifest format independent of the application programming language tooling that consumes the API manifest can be created in any programming language. Language specific tooling could be created to generate API manifests by introspecting application code.  Tooling could be created to produce API manifests to support  design first methodologies, or integration centric scenarios.


# Schema

```

apiManifest = {
    ? appPublisher: publisherDetails
    apiDependency : [* apiDependency]
}

; Identification of the application developer / organization
publisherDetails = {
    name: tstr
}

;  Declaration of application dependencies on HTTP API
apiDependency = {
    apiDescription: tstr
    auth: authDetails
    requests: [+ requestDetails]
}

; Permissions required by client application for the described dependency
authDetails = {
    ? clientId: tstr
    ? permissions: [+ tstr]
}

; sdsd
requestDetails = {
    method: tstr
    resourceIdentifierTemplate: tstr
}

```

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
