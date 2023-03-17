---
title: "JSON Web Keys configuration"
linkTitle: "JSON Web Keys configuration"
weight: 3
date: 2017-01-05
---

JSON web keys are used by the IAM to sign and verify token signatures.

To generate a JSON web key for your IAM deployment, you can use the
[json-web-key-generator][jwk-generator] tool provided by MitreID connect.

The following command clones the `json-web-key-generator` Git repository to a
local directory:

```shell
git clone https://github.com/mitreid-connect/json-web-key-generator
```

Build the code with:

```shell
mvn package
```

Maven 3.6.x or greater supporting Java 11 is required.

You can then generate a key with the following command:

```shell
java -jar target/json-web-key-generator-0.9-SNAPSHOT-jar-with-dependencies.jar \
  -t RSA -s 1024 -S -i rsa1


Full key:
{
  "keys": [
    {
      "p": "3oh7ex6zgdmJh5NBD0IplmBDDGC2ECu2A1vcp8e8DqE7OSSpAc1T9tTjJioCGqkNM51JK_MtgCJz1CiysVDOQQ",
      "kty": "RSA",
      "q": "nRmBm5tQ2wmOtd1XUYDRH2qWai6eElt-1cvO5tnTdWZkFaAeaHQ3_xf_PFOjyAv5Y5rNLgf_Xbu9UCo_mSrDMQ",
      "d": "BGHRhQP6ADqqSrM8_mI0YhjGStj1aW9rLi7wXQMJ122kegPxIT7dfP-5UScxykD_BrCCHQVPxdJl5wXy-giZnhaL9wtDkOXb8D8RCi1n02cs3Z1T23xONi_AG47QPBZjM5GcX-oOGCENByuEIdkU_Bn6vvqM3oyVlj5sio7tNAE",
      "e": "AQAB",
      "kid": "rsa1",
      "qi": "RarXtTFCE3hk5ZanLWEapDnn7SLSxAvDcBTmG5SpCI9Eix7cfTigaK6N7OQIN0uGO1GJ-KVWL2v8dyI1jMoU6g",
      "dp": "MtBtieavzMXUzr2ETKyp_GmMxeXLjRO-IzQ1xaYpPhn5AQprATtWofVozQ0on9fcaN3QmJWV3T2Av4BvlWfDQQ",
      "dq": "CWJ7rpsBooQYpV6al8DVPUY1xBQS10_l7MmnC31Zt3qtYelVx7GhoriBQ85PS2UDueKGfUh3BddwQLi1YeX_EQ",
      "n": "iI_fuJq4z_9VQY5EH41sQWOAYUsjtxAFjRnAc1P5-GPOx3Izg9V7yKNmudLUt-jIkv6D5h-AzrhEV6DOdBRoiN4el1mCZ95jiJkjU2kpVOmutDysZkrn667zPd43w7E6IqHnahmMrVUjUyx6pie1SqJHLUXghz8Gle-1yi08_XE"
    }
  ]
}
```

Save the output of the above command (minus the `Full key:` initial text) in a
file, e.g. `keystore.jwks`.

See [JSON Web Keys (JWK)][jwk] reference page for more details about configuration of JSON web keys.


[jwk]: {{< ref "/docs/reference/configuration/jwk" >}}
[jwk-generator]: https://github.com/mitreid-connect/json-web-key-generator