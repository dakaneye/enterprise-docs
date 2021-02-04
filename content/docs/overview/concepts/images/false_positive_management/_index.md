---
title: "False Positive Management"
linkTitle: "False Positive Management"
weight: 1
---

When Anchore analyzes an image, it reports a Software Bill of Materials (SBOM) to be stored and later scanned in order to match package metadata against known vulnerabilities.
One aspect of the SBOM is a best effort guess of the CPE (Common Platform Enumeration) for a given package. The Anchore analyzer builds a list of CPEs for each package
based on the metadata that is available (ex. for Java packages, the manifest, which contains multiple different version specifications among other metadata), but sometimes gets this wrong.

For example, Java Spring packages are generally reported as follows:
* Spring Core, version 5.1.4
    * `cpe:2.3:a:*:spring-core:5.1.4:*:*:*:*:*:*:*`
    
However, since Spring is a framework built by Pivotal Software, the CPE referenced in the NVD database looks more like:
* `cpe:2.3:a:pivotal_software:spring_security:5.1.4:*:*:*:*:*:*:*`

To facilitate this correction, Anchore provides the False Positive Management feature. Now, a user can provide a correction that will
update a given package's metadata so that attributes (including CPEs) can be corrected when Anchore does a vulnerability scan

Using the above example, a user can add a correction as follows, via HTTP POST to the `/corrections` endpoint:
```json
{
  "description": "Update Spring Core CPE",
  "match": {
    "type": "java",
    "field_matches": [
        {
            "field_name": "package",
            "field_value": "spring-core"
        },
        {
            "field_name": "implementation-version",
            "field_value": "5.1.4.RELEASE"
        }
    ]
  },
  "replace": [
      {
          "field_name": "cpes",
          "field_value": "cpe:2.3:a:pivotal_software:spring_security:5.1.4:*:*:*:*:*:*:*"
      }
  ],
  "type": "package"
}
```
JSON Reference:
* description: A description of the correction being added (for note taking purposes)
* type: The type of correction being added. Currently only "package" is supported
* match:
    * type: The type of package to match upon. Supported values are based on the type of content available to images being analyzed (ex. java, gem, python, npm, os, go, nuget)
    * field_matches: A list of name/value pairs based on which package metadata fields to match this correction upon
        * The schema of the fields to match can be found by outputting the direct JSON content for the given content type: 
            * Ex. Java Package Metadata JSON: 
            ```json 
            {
                "cpes": [
                    "cpe:2.3:a:*:spring-core:5.1.4.RELEASE:*:*:*:*:*:*:*",
                    "cpe:2.3:a:*:spring-core:5.1.4.RELEASE:*:*:*:*:java:*:*",
                    "cpe:2.3:a:*:spring-core:5.1.4.RELEASE:*:*:*:*:maven:*:*",
                    "cpe:2.3:a:spring-core:spring-core:5.1.4.RELEASE:*:*:*:*:*:*:*",
                    "cpe:2.3:a:spring-core:spring-core:5.1.4.RELEASE:*:*:*:*:java:*:*",
                    "cpe:2.3:a:spring-core:spring-core:5.1.4.RELEASE:*:*:*:*:maven:*:*"
                ],
                "implementation-version": "5.1.4.RELEASE",
                "location": "/app.jar:BOOT-INF/lib/spring-core-5.1.4.RELEASE.jar",
                "maven-version": "N/A",
                "origin": "N/A",
                "package": "spring-core",
                "specification-version": "N/A",
                "type": "JAVA-JAR"
            }
            ```
* replace: a list of field name/value pairs to replace. Note: if a new field is specified here, it will be added to the content output when the correction is matched. See below for additional functionality around CPEs
        
Corrections may be updated and deleted via the API as well. Creation of a Correction generates a UUID that may be used to reference that Correction later.
Refer to the Enterprise Swagger spec for more details.

## CPE Templating

CPE replacement can be templated based on the other fields of the package as well. In the above example, a replacement could have been provided as follows:
```json
{
  "field_name": "cpes",
  "field_value": "cpe:2.3:a:pivotal_software:spring_security:{implementationVersion}:*:*:*:*:*:*:*" 
}
```

For the "cpes" field only, Anchore Enterprise can recognize a templated field via curly braces "{}". Package JSON keys contained here will be replaced with their corresponding value.
