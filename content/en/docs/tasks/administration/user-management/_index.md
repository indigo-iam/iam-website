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

### Administrator privileges
Administrator privileges can be assigned to a user from the user home page by
clickin on the "Assign administrator privileges" button:

![Assign admin privileges](../images/assign-admin-privileges.png)

### Monitoring privileges
Monitoring privileges can be assigned to a user from the user home page by
clickin on the "Assign monitoring privileges" button:

<img width="1022" height="943" alt="image" src="https://github.com/user-attachments/assets/72b923a6-089e-4fc7-8b7a-f3e803977a7e" />

After confimation the user will have monitoring privileges

<img width="842" height="357" alt="image" src="https://github.com/user-attachments/assets/19daa481-42ec-4178-a6ce-a47ae2448e53" />


This will allow the privileged user to see all account 
details (except secrets) without edit permission 

<img width="1273" height="901" alt="image" src="https://github.com/user-attachments/assets/7a9478c8-c153-487d-8b89-e59a601e658d" />

### Service account privileges
#### Set
A user account can be designated as a service account from the User Home page by clicking the “Set as Service Account” button:

<img width="768" height="972" alt="image" src="https://github.com/user-attachments/assets/c2067158-dd51-4fc6-abdf-5bd6b5b761a7" />

Once an account is configured as a service account, it is exempt from the AUP (Acceptable Use Policy) signature process. Email notification will be sent to the user.
Service accounts are typically used solely to own robot certificates or manage token clients required by grid services.

#### Revoke
To revoke service account status "Revoke service account" button can be used. Once successfull, email notification will be sent to the user.

<img width="733" height="947" alt="image" src="https://github.com/user-attachments/assets/ead289df-624e-47b3-92b3-c5653a4e6b8a" />


## Managing external user account identities

Administrators can manually add and remove external accounts and certificates
for a local user account from the user home page:

![Manage credentials](../images/manage-credentials.png)



