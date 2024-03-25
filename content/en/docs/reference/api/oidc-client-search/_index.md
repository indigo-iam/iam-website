---
title: "OpenID Connect client search"
linkTitle: "OpenID Connect client search"
---

IAM supports searching for clients based on user-specified filters via a ``GET`` on the ``/iam/api/search/clients`` endpoint.

| Name | Values | Description |
|:------------:|-------------|-------------|
| `search`<br/>**string**| Non-blank string | What to search. |
| `SearchType`<br/>**string**| One of: ``name``, ``contacts``, ``scope``, ``grantType``, ``redirectUri`` | Where to search for the ``search`` string. |
| `drOnly`<br/>**boolean**| ``true`` or ``false`` | Dynamically registered only. |
| `sortProperties`<br/>**string**| One of: ``clientDescription``, ``reuseRefreshTokens``, ``dynamicallyRegistered``, ``allowIntrospection``, ``idTokenValiditySeconds``, ``clientId``, ``clientSecret``, ``accessTokenValiditySeconds``, ``refreshTokenValiditySeconds``, ``applicationType``, ``clientName``, ``tokenEndpointAuthMethod``, ``subjectType``, ``logoUri``, ``policyUri``, ``clientUri``, ``tosUri``, ``jwksUri``, ``jwks``, ``sectorIdentifierUri``, ``requestObjectSigningAlg``, ``userInfoSignedResponseAlg``, ``userInfoEncryptedResponseAlg``, ``userInfoEncryptedResponseEnc``, ``idTokenSignedResponseAlg``, ``idTokenEncryptedResponseAlg``, ``idTokenEncryptedResponseEnc``, ``tokenEndpointAuthSigningAlg``, ``defaultMaxAge``, ``requireAuthTime``, ``createdAt``, ``initiateLoginUri``, ``clearAccessTokensOnRefresh``, ``softwareStatement``, ``codeChallengeMethod``, ``softwareId``, ``softwareVersion``, ``deviceCodeValiditySeconds``, ``lastUsed`` | The property of the client used to sort the results. |
| `sortDirection`<br/>**UUID**| ``ASC`` or ``DESC`` | Ascending or descending. |
| `accountId`<br/>**UUID**| A valid account UUID | Filters by account UUID. |
| `startIndex`<br/>**integer**| [1-n] | The 1-based index of the first query result. |
| `count`<br/>**integer**| [0-n] | The number of results to collect. |
