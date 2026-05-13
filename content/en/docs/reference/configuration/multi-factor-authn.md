---
title: "Configuration to enforce MFA for all users"
linkTitle: "Enforce MFA for all users"
weight: 100
---

## Configuration to enforce MFA for all users

Administrators can configure IAM to enforce multi-factor authentication (MFA) for all users.
In the application-mfa.yaml file, the following property should be configured. By default, it is set to false:
```yaml
multi-factor-mandatory:  ${IAM_MULTI_FACTOR_MANDATORY:true}
```

When `multi-factor-mandatory` is set to `true`, users will be prompted to set up MFA during login.

<img width="1025" height="1042" alt="image" src="https://github.com/user-attachments/assets/d4060f37-70b5-4322-9ea2-3967dcf14f99" />

After successfully enabling MFA, the user will be redirected to the logout page. To access the application again, the user must log in using MFA.


