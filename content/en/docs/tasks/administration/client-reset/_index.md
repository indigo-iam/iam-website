---
title: Security Reset a Client or Revoke Tokens
weight: 2
---
Existing clients can have their tokens revoked or be get a security reset by an Administrator from dashboard.

## Revoking a client's tokens using the dashboard

Log into the service using admin credentials and click on the _Clients_ link on the left
navigation bar:

![dashboard](../images/client-status-change-1.png)

From the _Clients_ link, select _Any client_ you want to disable, for example _Test Client_:

![client list](../images/client-status-change-2.png)

To revoke the client's tokens click on the _Revoke tokens_ button on the bottom of the page:

![revoke tokens button](../images/client-revoke-1.png)

To specify which tokens you want to remove, select the corresponding checkbox. To confirm your choice click on the _Revoke Tokens_ button on the modal window:

![revoke tokens modal](../images/client-revoke-2.png)

On success you will get a confirmation message and you will be redirected to the clients page:

![revoke tokens confirmation](../images/client-revoke-3.png)

## Doing a Security-reset

A security reset will revoke all the tokens issued to that client and rotate the client secret. 
The intention of this is that given one knows the client has been compromised, then it is easily and quickly possible to reset all access for that given client. 

Log into the service using admin credentials and click on the _Clients_ link on the left
navigation bar:

![dashboard](../images/client-status-change-1.png)

From the _Clients_ link, select _Any client_ you want to disable, for example _Test Client_:

![client list](../images/client-status-change-2.png)

To do a security reset of the client click on _Security Reset_ button on the bottom of the page:

![reset client button](../images/client-reset-1.png)

To confirm your choice click on the _Reset Client_ button on the modal window:

![reset client modal](../images/client-reset-2.png)

On success you will get a confirmation message which displays the new client secret of the client that was reset:

![reset client confirmation](../images/client-reset-3.png)
