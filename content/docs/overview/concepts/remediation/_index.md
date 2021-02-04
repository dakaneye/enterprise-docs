---
title: "Remediation"
linkTitle: "Remediation"
weight: 2
---

After Anchore analyzes images, discovers their contents, and matches vulnerabilities, it can suggest possible actions that can be taken.

These actions range from adding a Healthcheck to your Dockerfile, to upgrading a package version. 

Since the solutions for resolving vulnerabilities can vary, and may require several different forms of remediation and intervention, Anchore provides the capability to plan out your course of action

## Action Plans

Action plans group up the resolutions that may be taken to address the vulnerabilities or issues found in a particular image, and provide a way for you to take "action".

Currently, we support one type of Action Plan, which can be used to notify an existing endpoint configuration of those resolutions. This is a great way to facilitate communication across teams when vulnerabilities need to be addressed. 

Here's an example JSON that describes an Action Plan for notifications:
```json
{
    "type": "notification",
    "image_tag": "docker.io/alpine:latest",
    "image_digest": "sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e7841a5b5a",
    "bundle_id": "anchore_default_bundle",
    "resolutions": [
        {
            "trigger_ids": ["CVE-2020-11-09-fake"],
            "content": "This is a Resolution for the CVE",
        }
    ],
    "subject": "Actions required for image: alpine:latest",
    "message": "These are some issues Anchore found in alpine:latest, and how to resolve them",
    "endpoint": "smtp",
    "configuration_id": "cda118f9ec63ddefb4a173a2b2a03"
}
```

Parts:
* type: The type of action plan being submitted (currently, only notification supported)
* image_tag: The full image tag of the image requiring action
* image_tag: The image digest of the image requiring action
* bundle_id: the id of the policy bundle that discovered the vulnerabilities
* resolutions: A list composed of the remediations and corresponding trigger IDs
* subject: The subject line for the action plan notification
* message: The body of the message for the action plan notification
* endpoint: The type of notification endpoint the action plan will be sent to
* configuration_id: The uuid of the notification configuration for the above endpoint

