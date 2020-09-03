---
title: "Using the Anchore API"
linkTitle: "Using the API"
weight: 4
---

The Anchore Enterprise API is a combination of the open-source Anchore Engine external API as well as enterprise-only extensions. The has been documented using the OpenAPI Specification (Swagger) and the source can be found in the [swagger.yaml](https://github.com/anchore/anchore-engine/blob/master/anchore_engine/services/apiext/swagger/swagger.yaml) document within the external API service.  There are also a variety of ways in which the API specification can be accessed.

## Online Specifications

The API specifications for this release of Anchore Enterprise are available from the documentation.

## Enterprise Service APIs

This service, typically called `api` or `apiext` in deployment templates has two specifications that are combined to provide the full API for the service:

The primary external-facing API is combination of the [Engine API](specs/engine_api.yaml) and its [Enterprise Extensions API](specs/enterprise_api_swagger.yaml)

[Reporting Service API](specs/reports_swagger.yaml)

[RBAC Service API](specs/rbac_manager_swagger.yaml)

[Notifications Service API](specs/notifications_swagger.yaml)

[Feed Service API](specs/feeds_swagger.yaml)


### Swagger JSON from a Deployment

#### Engine-Compatible API

The JSON definition for the Engine API specification for your specific instance of Anchore can be downloaded from a running Anchore Enterprise service at the following URI:

http://{servername:port}/v1/swagger.json

e.g.

http://localhost:8228/v1/swagger.json

This API is compatible with Anchore Engine to ensure an easy transition from using Engine to Enterprise. Additional features and extensions are available from other service endpoints (e.g. reporting and notifications) or via
the Enterprise extension specification below.

### Enterprise Extensions

Anchore Enterprise includes additional API calls and features not available in Anchore Engine. These use a distinct base route and separate specification file. The file is available at:

http://{servername:port}/v1/enterprise/swagger.json

e.g.

http://localhost:8228/v1/enterprise/swagger.json

### Local Swagger UI

Finally, Anchore includes the ability to deploy a sidecar set of containers that provide a local Swagger UI, allowing for the API to be viewed and tested within a web browser. The Swagger UI is accessed by first enabling an nginx proxy service and swagger-ui service that run alongside your anchore engine deployment, and then directing your browser at the nginx proxy service.

To enable in a quickstart deployment (using docker-compose), simply uncomment the two service sections (the 'anchore-swagger-ui-nginx' and 'anchore-swagger-ui' services) at the end of the default docker-compose.yaml before starting up anchore, according to the regular [Quickstart Guide]({{< ref "/docs/quickstart" >}}).  Once enabled, you can then direct your browser at your server, port 8080, and start browsing and testing API calls.

http://{servername}:8080/

e.g.

http://localhost:8080/
