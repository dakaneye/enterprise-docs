---
title: "Base and Parent Images"
linkTitle: "Base Images"
weight: 1
---

A Docker or OCI image is composed of layers. These layers may be inherited from another image or created during the build of a specific image as defined 
by the instructions in a Dockerfile or other build process.

The ancestry of an image shows the image's parent image(s) and base image. As defined by the [Docker documentation](https://docs.docker.com/develop/develop-images/baseimages/), 
a parent of an image is the image used to start the build of the current image, typically the image identified in the `FROM` directive in the Dockerfile.
If the parent image is `SCRATCH` then the image is considered a base image.

Anchore Enterprise provides an API call to retrieve all parents and bases of an image. This call dynamically computes the ancestry set using images that 
have already been analyzed by the system and returns a list of image digests and their respective layers. The ancestry computation is done by matching 
exact layer digests between the requested image and other images in the system.

If an image X is a parent of image Y, then image X's layers will be the first N layers of image Y where N in the number of layers in image X.

A case where an image may have multiple results in its ancestry is as follows:

A base distro image, for example `debian:10`:
```
FROM scratch
...
```

An application container image from that debian image, for example a node.js image let's call `mynode:latest` :
```
FROM debian:10

# Install nodejs
```

The application image itself built from the framework container, let's call it `myapp:v1` :
```
FROM mynode:latest
COPY ./app /
...
```

In this case, the parent image of `myapp:v1` is `mynode:latest` and its base is `debian:10`. Anchore will return each of those images with their matching layers
in the API call. See the [API docs]({{< ref "/docs/using/api_usage" >}}) for more information on the specifics of the `GET /v1/enterprise/images/{digest}/ancestors` API call itself.

The returned images can be used for subsequent calls to the base image comparison APIs for vulnerabilities and policy evaluations allowing you to determine
which findings for an image are inherited from a specific parent or base image.

## Comparing an Image with its Base or Parent

Anchore Enterprise provides a mechanism to compare the policy checks and security vulnerabilities of an image with those of a base image. This allows clients to

- filter out results that are inherited from a base image and focus on the results relevant to the application image
- reverse the focus and examine the base image for policy check violations and vulnerabilities which could be a deciding factor in choosing the base image for the application    

To read more about the base comparison features, jump to 

- [Compare policy checks]({{< ref "/docs/overview/concepts/images/base_images/checks" >}})
- [Compare vulnerabilities]({{< ref "/docs/overview/concepts/images/base_images/vulnerabilities" >}})