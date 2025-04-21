---
title: User management
weight: 1
---

IAM provides tools to manage users in an organization.
All the actions described below require administrator privileges.

## Creating a user account

A new user account in the organization can be created from the "Users"
management panel by clicking on the "Add user" button at the bottom of the
page:

![Add user](../images/add-user.png)

A dialog is shown that allows to set basic user information:


![Add user 2](../images/add-user-2.png)


## Deleting a user account

User accounts can be deleted from the "Users management" section of the
dashboard by clicking on the delete button for the users that should be
deleted:

![Delete user](../images/delete-users.png)


## Disabling a user account

User accounts can be disabled from the user home page by clicking on the
"Disable user" button. Disabled users are not allowed to login in the IAM and,
as such, cannot authenticate at any services that rely on the IAM for
authentication and authorization.

![Disable user](../images/disable-user.png)

## Managing user account privileges

Administrator privileges can be assigned to a user from the user home page by
clickin on the "Assign administrator privileges" button:

![Assign admin privileges](../images/assign-admin-privileges.png)

## Managing external user account identities

Administrators can manually add and remove external accounts and certificates
for a local user account from the user home page:

![Manage credentials](../images/manage-credentials.png)

The users can also request to link a certificate to their account. The administrator 
will receive an email with the request and can use the "Requests" page to approve or reject
it. The user will then be notified via mail of the request outcome.

![Manage cert link requests](../images/cert-link-request-approval.png)



