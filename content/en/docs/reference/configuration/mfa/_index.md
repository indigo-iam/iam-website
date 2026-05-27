---
title: "Multi-factor authentication"
linkTitle: "Multi-factor authentication"
---

IAM supports Multi-Factor Authentication (MFA) via Time-based One-Time
Passwords (TOTP). When MFA is enabled, the TOTP secret associated with each
account is stored in the IAM database encrypted with a symmetric key derived
from a password that the administrator provides at IAM startup.

To enable MFA, activate the `mfa` Spring profile. The encryption password is
read from `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT` (which maps to the
`mfa.password-to-encrypt-and-decrypt` property).

> **Important.** The default value of
> `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT` is `define_me_please`. **Any
> production deployment must override it** with a strong, randomly generated
> password before enabling MFA. If MFA has already been enabled with the
> default value in place, follow the rotation procedure below to move to a
> proper secure key

Store the encryption password securely (for example in a secret manager): if
you lose it, the TOTP secrets already stored in the database cannot be
decrypted and the affected users will need to re-enroll their authenticator.

