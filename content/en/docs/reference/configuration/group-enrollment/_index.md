---
title: "Automatic group enrollment"
linkTitle: "Automatic group enrollment"
weight: 5
---

Starting from v1.10.0 version, IAM allows to define a list of group a newly created account will be member of, skipping the group request phase. This is useful for instance for IAM using also VOMS-AA, where every user must belong at least to the primary group equal to the VO name in order to request for a VOMS proxy certificate.

To include this feature, the `application.yml` file must contain the following properties

```yaml
iam:
  registration:
    default-groups:
      - name: <iam-group>
        enrollment: INSERT
```

As the `iam.registration.default-groups` is a list, more than one group may be configured, e.g.

```yaml
iam:
  registration:
    default-groups:
      - name: atlas
        enrollment: INSERT
      - name: privileged
        enrollment: INSERT
```

In the current IAM version, the only enrollment type supported is `INSERT`, meaning that when creating a new account, it automatically becomes part of the default group if that group exists.
