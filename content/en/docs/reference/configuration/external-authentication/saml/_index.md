---
title: "SAML authentication"
linkTitle: "SAML authentication"
weight: 2
---

IAM supports brokered SAML authentication. When SAML authentication is turned
on for an IAM instance, users can log into the service using the home Identity
Provider (IdP) credentials.

To enable SAML authentication for your IAM instance you have to:

- Choose an `entityId` for your IAM instance; see the [Shibboleth
  docs][shib-docs] for guidance on this
- Generate a self-signed certificate for your IAM instance that will be used
  for SAML cryptographic operations and include it in a Java keystore;
- Obtain SAML metadata from your IdP or identity federation; 
- If the metadata is signed, import the certificate that must be used to verify
  the signature in the Java keystore generated above;
- Enable the `saml` profile for the IAM and configure SAML support via the
  appropriate environment variables.

## Generate a Self-signed certificate 

To generate a self-signed certificate that lasts 5 years use the following
openssl command:

```console
openssl req -newkey rsa:2048 -nodes -x509 -days 1825  -keyout self-signed.key.pem \
  -out self-signed.cert.pem
```

The above command will generate a private key and the corresponding certificate
in PEM format. You will be asked to provide a subject for the certificate being
generated. 

To import this certificate in a Java keystore we need in in p12 format. Use the
following commands to convert the certificate in p12 format and import it in a
Java keystore:

```console
openssl pkcs12 -export -inkey self-signed.key.pem \
    -name self-signed \
    -in self-signed.cert.pem \
    -out self-signed.p12

keytool -importkeystore -destkeystore self-signed.jks \
    -srckeystore self-signed.p12 \
    -srcstoretype PKCS12
```

During the above process you will be asked to provide a p12 export password and
a keystore password. **Use the same password**, at least 6 characters long,
otherwise it will not work for the java keystore.

You can list the contents of the keystore with the following command:

```console
keytool -list -keystore self-signed.jks
```

## Obtain SAML metadata for your IdP or identity federation

You IdP or identity federation will have a SAML metadata document describing,
in SAML terms, which authentication flows are enabled and providing access to
cryptographic material and other information that make the federated
authentication schemes work reliably and securely. For more information on SAML
metadata, see [the Shibboleth metadata documentation][shib-docs-md].

{{% alert title="Warning" color="warning" %}}
In case you configure IAM such to support a SAML federation, the polling and
refreshing of the federation metadata may encounter an out of memory issue.
Then, we reccomend to increase the heap space to 3 GB through the JVM options:
`IAM_JAVA_OPTS=-Xms3000m -Xmx3000m`.
{{% /alert %}}

###  Signed metadata advice 

Typically identity federations sign the SAML metadata for security reasons, and
provide a certificate (and linked public key) that can be used to verify
signatures on the metadata at a well-known location (e.g., the federation
website).

To support metadata signature verification you will need to import this
certificate in the Java keystore created above. 

Certificates are typically published in PEM format, while to import a
certificate in a Java keystore you will need it in DER format.

The following commands show how to import a PEM certificate into a Java
keystore:

```console
openssl x509 -outform der -in idem_signer_2019.pem -out idem_signer_2019.der
keytool -import -alias idem_signer_2019 -keystore self-signed.jks -file idem_signer_2019.der
```

### Metadata configuration

The IdP or federation metadata to be trusted can be specified with the
`IAM_SAML_IDP_METADATA` environment variable, which contains an URL pointing to
the metadata.

If the metadata is signed, you can enforce signature validation with the
`IAM_SAML_METADATA_REQUIRE_VALID_SIGNATURE` variable.

Metadata needs to be periodically refreshed to learn about new entities in the
federation or updated key material. The metadata refresh interval can be
specified with the `IAM_SAML_METADATA_LOOKUP_SERVICE_REFRESH_PERIOD_SEC`
environment variable, which defines a refresh period in seconds (a reasonable
value is typically refresh metadata every hour).

An example metadata configuration could be:

```env
IAM_SAML_IDP_METADATA=http://www.garr.it/idem-metadata/idem-metadata-sha256.xml
IAM_SAML_METADATA_REQUIRE_VALID_SIGNATURE=true
IAM_SAML_METADATA_LOOKUP_SERVICE_REFRESH_PERIOD_SEC=3600
```

## IAM SAML attribute requirements 

IAM requires that the IdP used for authentication releases a persistent
identifier attribute that can be used to link the external identity to local
IAM accounts. If the IdP does not release persistent identifier attributes, it
**cannot** be integrated with the IAM.

The attribute used to link the external SAML account is configurable. In the
default configuration, IAM uses the following attributes (in the given order):

1. [EduPersonUniqueId][epuid]
2. [EduPersonTargetedId][eptid]
3. [EduPersonPrincipalName][eppn]

The order of these can be changed with the `IAM_SAML_ID_RESOLVERS` environment variable,
which accepts a list of aliases for the attributes to be searched.

The following tables maps attribute names to the aliases currently supported:

| SAML attribute         | Alias                  |
| ---                    | ---                    |
| EduPersonUniqueId      | eduPersonUniqueId      |
| EduPersonTargetedId    | eduPersonTargetedId    |
| EduPersonPrincipalName | eduPersonPrincipalName |
| EduPersonOrcid         | eduPersonOrcid         |
| spidCode               | spidCode               |

The default value for IAM_SAML_ID_RESOLVERS is:

```env
IAM_SAML_ID_RESOLVERS=eduPersonUniqueId,eduPersonTargetedId,eduPersonPrincipalName
```

## Checking SAML assertion and authentication freshness

IAM provides the ability to set a maximum age for SAML assertion and
authentication statements received from IdPs.

The `IAM_SAML_MAX_ASSERTION_TIME` allows to define an upper bound (in seconds)
on the SAML assertion lifetime, so that assertions that exceed such limit are
considered invalid.

The `IAM_SAML_MAX_AUTHENTICATION_AGE` allows to define an upper bound (in
seconds) on the authentication age linked to SAML authentication statements, so
that users are forced to reauthenticate when this limit is exceeded.

## Limiting authentication only to selected IdPs

IAM provides the ability to define constraints on which IdPs will be allowed
for authentication. This is useful when you want to limit which IdPs in a large
federation (e.g., [EduGAIN][edugain]) will be trusted for authentication.

Currently three types of IdPs limiting are supported:

- entity id whitelist 
- SIRTFI compliance
- Research and Scholarship compliance

### Limiting by entity id

The `IAM_SAML_IDP_ENTITY_ID_WHITELIST` environment variable can be used to
define a comma-separated whitelist of IdPs entity IDs that will be trusted for
authentication.

The following example limits trusted IdPs to the INFN IdP:

```env
IAM_SAML_IDP_ENTITY_ID_WHITELIST="https://idp.infn.it/saml2/idp/metadata.php"
```

### Limiting by SIRTFI compliance

To limit authentication only to IdPs that are [SIRFTI][sirtfi]-compliant, set
the the `IAM_SAML_METADATA_REQUIRE_SIRTFI` environment variable to true.


## Joining a SAML federation

An Indigo IAM instance can join a SAML federation as a service provider (SP).
To join a feaderation you need to follow the following steps:

* Configure the `saml` profile in IAM
* Register the Indigo IAM instance into the federation, following the federation
instructions (generally involves connecting to a registration portal to 
declare the SP)
* Wait for the federation metadata to be updated at the federation IdPs. This
typically involves a 12h delay.
* Check if login in IAM works successfully through the federation: note that it
will not work before the federation IdPs have updated their metadata. If the
error persists after the delay, contact your federation or IdP support.

When registering the Indigo IAM SP, there are a few critical parameters to supply:

* Assertion Consumer Service (ACS), consisting of 3 parameters:
  * Binding: must be `HTTP Post`
  * URL: it is the instance URL + `/saml/SSO`, e.g. `https://iam.example.com/saml/SSO`
  * Index: should be set to 0
* X509 SAML certificate: the generated `self-signed` certificate

Other, generally optional, parameters include the requested attribute. It is a good
practice to declare as mandatory one of the attribute listed in 
`IAM_SAML_ID_RESOLVERS`.

## Customizing SAML login button text

You can customize the SAML login button text shown in the IAM login page with
the `IAM_SAML_LOGIN_BUTTON_TEXT` environment variable.

## Minimal SAML configuration example

```env
IAM_SAML_ENTITY_ID=urn:example:iam-saml-example
IAM_SAML_KEYSTORE=file:///self-signed.jks
IAM_SAML_KEYSTORE_PASSWORD=xxxxxx
IAM_SAML_KEY_ID=self-signed
IAM_SAML_KEY_PASSWORD=xxxxxx
IAM_SAML_IDP_METADATA=http://www.garr.it/idem-metadata/idem-metadata-sha256.xml
IAM_SAML_METADATA_REQUIRE_VALID_SIGNATURE=true
IAM_SAML_METADATA_LOOKUP_SERVICE_REFRESH_PERIOD_SEC=3600
IAM_SAML_LOGIN_BUTTON_TEXT="Sign in with IDEM"
```

## Custom SAML configuration

The configuration based on environment variables relies on a [configuration
template][application-saml] that fits most use cases embedded in the IAM war
file. 

To have full control on the SAML configuration, provide a custom
`application-saml.yml` file in the IAM configuration directory.

See the [configuration reference][conf-ref] for more SAML metadata options and
instructions on how to override the default IAM configuration.

[epuid]:http://software.internet2.edu/eduperson/internet2-mace-dir-eduperson-201602.html#eduPersonUniqueId
[eptid]:http://software.internet2.edu/eduperson/internet2-mace-dir-eduperson-201602.html#eduPersonTargetedID
[eppn]: http://software.internet2.edu/eduperson/internet2-mace-dir-eduperson-201602.html#eduPersonPrincipalName
[conf-ref]: {{< ref "/docs/reference/configuration" >}}
[shib-docs]: https://wiki.shibboleth.net/confluence/display/CONCEPT/EntityNaming
[shib-docs-md]: https://wiki.shibboleth.net/confluence/display/CONCEPT/Metadata
[sirtfi]: https://refeds.org/sirtfi
[edugain]: https://edugain.org/
[application-saml]: https://raw.githubusercontent.com/indigo-iam/iam/{{< param version >}}/iam-login-service/src/main/resources/application-saml.yml


## Registration form: filling information from IdP

See [registration configuration][external-idp-config].

[external-idp-config]: {{< ref "/docs/reference/configuration/registration/#registration-with-external-idp" >}}

