---
title: "IAM Notifications"
linkTitle: "IAM Notifications"
weight: 7
---

The e-mail notifications from the IAM service may be customized with a YAML file. Here are reported the default values of the notification properties:

```yaml
notification:
  # When set to true, the notifications are not sent by the mail server,
  # but appears in the log
  disable: false
  # Alias for the address of the email sender
  mailFrom: indigo@localhost
  # Time interval in milliseconds between two consecutive runs of IAM notification task
  taskDelay: 5000
  # Retention of delivered messages, in days
  cleanupAge: 30
  # Address of the recipient of IAM notifications
  adminAddress: indigo-alerts@localhost
  # Notification policy for account requests. Possible values are:
  # - notify-address: the notifications are sent to the email address (default)
  # - notify-admins: the notification arrives to  all IAM admins
  # - notify-address-and-admins: combine the two behaviors
  admin-notification-policy: notify-address
  # When set to true, email notifications will be made, when a certificate is linked 
  # or unlinked from a profile. The email strategy for this notification follows 
  # the admin-notification-policy
  certificateUpdate: false
  # Notification policy for group requests. Default value notifies both
  # admins and group managers. Set to notify-gms if you want to notify only group managers
  group-manager-notification-policy: notify-gms-and-admins
  # The IAM email subject prefix
  subject-prefix: "[${iam.organisation.name} IAM]"
  # The IAM email subject (after prefix) for
  subject:
    # new registration request
    adminHandleRequest: New ${iam.organisation.name} account request
    # a confirmation of the registration request
    confirmation: Confirm your ${iam.organisation.name} registration request
    # account activation
    activated: Your ${iam.organisation.name} account is now active
    # reject a registration request
    rejected: Your request for a ${iam.organisation.name} account has been rejected
    # reset user's password
    resetPassword: Reset your ${iam.organisation.name} account password
```

In order to indicate the SMTP mail server used to send IAM notifications the following properties have to be modified:

```yaml
spring:
  mail:
    # SMTP server hostname 
    host: localhost
    # SMTP server port
    port: 25
    # SMTP server username
    username:
    # SMTP server password
    password:
```