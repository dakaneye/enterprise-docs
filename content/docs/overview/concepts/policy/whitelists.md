---
title: "Allowlists"
linkTitle: "Allowlists"
weight: 5
---

Allowlists provide a mechanism within a policy bundle to explicitly override a policy-rule match. An allowlist is a named set of exclusion rules that match trigger outputs.

Example allowlist:

```JSON
{
  "id": "allowlist1",
  "name": "My First Allowlist",
  "comment": "A allowlist for my first try",
  "version": "1_0",
  "items": [
    {
      "gate": "vulnerabilities",
      "trigger_id": "CVE-2018-0737+*",
      "id": "rule1",
      "expires_on": "2019-12-30T12:00:00Z"
    }
  ]
}
```

The components:

- Gate: The gate to allowlist matches from (ensures trigger_ids are not matched in the wrong context)
- Trigger Id: The specific trigger result to match and allowlist. This id is gate/trigger specific as each trigger may have its own trigger_id format. We'll use the most common for this example: the CVE trigger ids produced by the vulnerability->package gate-trigger. The trigger_id specified may include wildcards for partial matches.
- id: an identifier for the rule, must only be unique within the allowlist object itself
- Expires On: (optional) specifies when a particular allowlist item expires. This is an RFC3339 date-time string. If the rule matches, but is expired, the policy engine will NOT allowlist according to that match. 

The allowlist is processed if it is specified in the mapping rule that was matched during bundle evaluation and is applied to the results of the policy evaluation defined in that same mapping rule. If a allowlist item matches a specific policy trigger output, then the action for that output is set to go and the policy evaluation result notes that the trigger output was matched for a allowlist item by associating it with the allowlist id and item id of the match.

An example of a allowlisted match from a snippet of a policy evaluation result (See Policies for more information on the format of the policy evaluation result). This a single row entry from the result:

```JSON
[                                                
  "0e2811757f931e2259e09784938f0b0990e7889a37d15efbbe63912fa39ff8b0", 
  "docker.io/node:latest", 
  "CVE-2018-0737+openssl", 
  "vulnerabilities", 
  "package", 
  "MEDIUM Vulnerability found in os package type (dpkg) - openssl (fixed in: 1.0.1t-1+deb8u9) - (CVE-2018-0737 - https://security-tracker.debian.org/tracker/CVE-2018-0737)", 
  "go", 
  {
    "matched_rule_id": "rule1", 
    "whitelist_id": "allowlist1", 
    "whitelist_name": "My First Allowlist"
  }, 
  "myfirstpolicy"
]
```

The items in order are:

- Image ID
- Tag used for evaluation
- Trigger ID of the policy rule match
- Gate name
- Trigger name
- Trigger Check Output message
- **Allowlist result object** - This shows that the match was allowlisted by our example allowlist policy and its rule.
Policy Id

**Note:** Allowlist are evaluated only as far as necessary. Once a policy rule match has been allowlisted by one allowlist item, it will not be checked again for allowlist matches. But, allowlist items may be evaluated out-of-order for performance optimization, so if multiple allowlist items match the same policy rule match any one of them may be the item that is actually matched against a given trigger_id.

### Next Steps

Read more about the [Anchore Policy Checks]({{< ref "/docs/overview/concepts/policy/policy_checks" >}}) for a complete list of gates and triggers.