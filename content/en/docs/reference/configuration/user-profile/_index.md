---
title: "User editable fields"
linkTitle: "User editable fields"
weight: 6
---

Starting with version 1.6.0, IAM allows to limit which fields of the user profile are editable by users.

The default, backward-compatible settings that allow users to edit all their
profile fields are defined as follows:

```yaml
iam:
  user-profile:
    editable-fields:
      - email
      - name
      - picture
      - surname
```

To prevent modifications to any of the fields remove the field name from
`editable-fields` list.

External configuration can be managed by placing directives as shown above in a
[custom configuration
file][custom-config-file].

[custom-config-file]: {{< ref "/docs/reference/configuration/#overriding-default-configuration-templates" >}}
