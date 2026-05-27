---
title: "Rotate TOTP symmetric key"
linkTitle: "Rotate TOTP symmetric key"
weight: 2
---

IAM provides a built-in mechanism to rotate the encryption password used to
protect TOTP secrets.

The rotation is driven by two properties:

| Property                              | Environment variable                              | Description                                                                  |
| ------------------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------------- |
| `mfa.password-to-encrypt-and-decrypt` | `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT`    | The **new** password you want to use for encrypting and decrypting TOTP secrets.               |
| `mfa.old-password-to-decrypt`         | `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT`            | The **previous** password, used only to decrypt existing secrets during rotation. Leave blank when no rotation is in progress. |

At startup, IAM compares the current password persisted in the database with
the value of `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT`. If they differ
and `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT` is set, IAM will:

1. Decrypt every existing TOTP secret using `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT`.
2. Re-encrypt each secret using `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT`.
3. Persist the new password as the current encryption key.

Once the migration has completed successfully,
`IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT` is no longer needed and **MUST BE
REMOVED** from the configuration on the next restart.

### Step-by-step rotation procedure


1. **Stop the IAM service.**

    The following procedure assumes IAM is currently running with an encryption
    password `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT`. Please set that to `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT`.


2. **Back up the IAM database.** Rotation re-writes the encrypted TOTP secret
   on every MFA-enabled account. A backup lets you roll back if anything goes
   wrong.

3. **Choose a NEW password.** 

4. **Update the configuration** so that the old password is exposed via
   `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT` and the new one via
   `IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT`:

   ```env
   IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT=<strong_random_password>
   IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT=<define_me_please>
   ```

5. **Start IAM.** Re-encryption runs automatically during startup. Inspect
   the application logs to confirm that the rotation completed without
   errors. A failure here typically means
   `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT` does not match the password
   originally used to encrypt the secrets.

6. **Verify MFA login** with at least one test account that already had MFA
   enabled before the rotation.

7. **Remove `IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT` from the configuration**
   and restart IAM. Leaving the old password in the environment after
   rotation is unnecessary.
