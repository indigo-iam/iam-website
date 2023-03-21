---
title: "JSON Web Keys (JWK)"
linkTitle: "JSON Web Keys (JWK)"
weight: 7
---

JSON Web Key (JWK) is a JSON data structure that represents a single public key or a set of
public keys. IAM uses private JWKs to sign tokens, and IAM clients use public JWKs to validate the tokens.
A [JWK Generator][jwk-generator] can be used to generate new keys.

### Keystore Location

JWKs are stored in a file that is called keystore. All the keys in this file will be
used for validating the tokens. Keystore location is set using the `IAM_KEY_STORE_LOCATION`
environment variable, e.g.:

```env
IAM_KEY_STORE_LOCATION=file:///secrets/jwk/keystore.jwks
```

### Default Key ID (kid)

If there are multiple JWKs in the keystore, `IAM_JWK_DEFAULT_KEY` environment variable
can be set to choose the key that signs the tokens, e.g.:

```env
IAM_JWK_DEFAULT_KEY=rsa2
```

If this variable is not set it will use the default value `rsa1`.

### Default JWE Keys

JSON Web Encryption (JWE) represents encrypted content using JSON-based data structures.
JWE allows to encrypt the JWT tokens so that only the intended recipient can decrypt
and see the content.

Default JWE decrypt key and encrypt key id can be set using the
`IAM_JWK_DEFAULT_JWE_DECRYPT_KEY_ID` and `IAM_JWK_DEFAULT_JWE_ENCRYPT_KEY_ID`
environment variables respectively, e.g.:

```env
IAM_JWK_DEFAULT_JWE_DECRYPT_KEY_ID=jwe-decrypt
IAM_JWK_DEFAULT_JWE_ENCRYPT_KEY_ID=jwe-encrypt
```

### Default Algorithms

Default alorithms used for signing and encrypting the key can be set using
`IAM_JWK_DEFAULT_JWS_ALGORITHM` and `IAM_JWT_DEFAULT_JWE_ALGORITHM` environment variables
respectively, default algorithms are like following:

```env
IAM_JWK_DEFAULT_JWS_ALGORITHM=RS256
IAM_JWT_DEFAULT_JWE_ALGORITHM=RSA_OAEP_256
```

## Rotating Keys

To change the JWKs without any interruption on the service, a rolling update can be done
using these configurations. Below is an example of a rolling update.

At the beginning there is one JWK in the keystore but it needs to be changed (e.g. regular
update or because of a security instance, etc.). For example:

```json
{
  "keys": [
    {
      "p": "7IsuNM1T2G83isHFmV8zNUri2x3xLBBT3gbA5905owUTqzqHsj7QJw8GqnadNse9G3FRFLB-ShNq64yvr7hXPO35E5HSrRsWKxYgzdPKPdl-M5xc6O-QQlKmkmje_r82M5KGbDpV_uBjoQHPGMxME6MlFaYtCFSd_keKvzpkev8",
      "kty": "RSA",
      "q": "6Q5OunfOfXqkvKdh-mUysuS5jJY4t18MAO9KLcH5wIH4iV2FmasEeqB0g0LSNrgitv9icGhQiRUVR8dqpATBgrhZiQlB1KneUdIe8W_QQQVhA_tqJgEuAb_kl8QDHvIyXAs8onYCSDtgSD6gw-qDivGpf7TzXZcosWrgSQDYi9k",
      "d": "DUSnJ_OYFqBG9gDzcj20RLPJj-CT6kqH0nh4zQIRV9mB2XkVsWk0mlo4tauqBXT8lrTN5ACyaIhFMItDqkApJeBtvyMf3E8G3tBEQnwEa8HABcCd54ZzC94t-U-pazXvoqpNxEfu6qCFFdV7u-0OgDmYbTT6_c4jDQ9jTZozuzxoh_qv6iOijBJ62yJLf4kvIOB-ghYwig6Cz1OLSgh_Y8EIO-6wk-aSZWXS7l7xnM7_8BBOzKxVG73YfPQi-9fPVnee8Cijd3RU5_uUw4yh5I87wYZzPzN3gK6Pazf6BZ-9vs1SboEyeVtd3TiOB2aBOyS4Ntohj6eljsfXEHYN4Q",
      "e": "AQAB",
      "kid": "rsa1",
      "qi": "oMPC-4NyoDZge2bOL5SHaSNKyrcnpC1XKctcxN64lRql5zETC30Nuq0zJWwFQQfVgkS8fiMRV1NEMy98EbTAj8k63PBfp5jywIx5atQFjZ101tYEsxHG4bfAuTxIwUFALS3KLIHyGAlTognRZ3UWNX0FHwH2AJUS0LJdbCuHqGA",
      "dp": "fn0Kj1fA3zrRw5RMuvAKbRQHbU58mah9gHuyCgrleEQNuYGJ8IngOxKw6NSWyyVR_WvCBMNANRJfGWhYxilYFQ_YIlvXAZQ7bXzeu9HmEt7LuoOBt1nrlVunESGdJTJQpJFU6mjjdXC-NenGuab9g365JPOAnzTIa6IMEqgB9Tc",
      "dq": "134JQdXBsriSVnYW3GZySIPBJcNJUhSqjrqzxjeT7XLAEpPNEtfU3mY7FZvVsA722mqt5O4veXFrKeG0M1DKdUpsT4-ltTQUg2dcgTY4Q8bre7ke32E73xZGEiaGFDkyY3VobXfciFmOn355PG49j14AsjroxGMel14RYK9Ni5E",
      "n": "11flaUmrserMifKmz8ONBbWWfCRAFjerniHvW1kIXP62hVI8xDWipojhLo7Ai-tV0jA4WSyodlfaA1NRXzZbbAC8pOOpPGkwLBHyOhcB4zpwTykvz5Ype3Mex20XIdBAc0DR4i8d9ZEpnIBqmPcmwP0gANmX82RZd3gZKxQ-umfDXZdNJqy2g1kawRjIP_CpznYzX8MY1OCfjZYXXsZvVeC2v0pRRre9tJEilCZEBY42jJyf6bY2cZhu2KTIpIx1_8Y2JB6AAAdnPsEgNRxMe0g5ShCmDvc93m9GsLZSy4Nm0pOWWVbx7-JK6-QWBdaiG_fAT_2OOndDL-hL1Ry3Jw"
    }
  ]
}
```

- Firstly, new JWK should be added to the keystore using the file in the `IAM_KEY_STORE_LOCATION`.
  The `IAM_JWK_DEFAULT_KEY` should still be left as the key id of the old key. This way all the
  clients can cache the new public key. Before the next step, waiting as long as the agreed policy for
  key caching for the infrastructure would be enough to make sure every client cached the new
  key.

  So new keystore in the example would be:

  ```json
  {
    "keys": [
      {
        "p": "7IsuNM1T2G83isHFmV8zNUri2x3xLBBT3gbA5905owUTqzqHsj7QJw8GqnadNse9G3FRFLB-ShNq64yvr7hXPO35E5HSrRsWKxYgzdPKPdl-M5xc6O-QQlKmkmje_r82M5KGbDpV_uBjoQHPGMxME6MlFaYtCFSd_keKvzpkev8",
        "kty": "RSA",
        "q": "6Q5OunfOfXqkvKdh-mUysuS5jJY4t18MAO9KLcH5wIH4iV2FmasEeqB0g0LSNrgitv9icGhQiRUVR8dqpATBgrhZiQlB1KneUdIe8W_QQQVhA_tqJgEuAb_kl8QDHvIyXAs8onYCSDtgSD6gw-qDivGpf7TzXZcosWrgSQDYi9k",
        "d": "DUSnJ_OYFqBG9gDzcj20RLPJj-CT6kqH0nh4zQIRV9mB2XkVsWk0mlo4tauqBXT8lrTN5ACyaIhFMItDqkApJeBtvyMf3E8G3tBEQnwEa8HABcCd54ZzC94t-U-pazXvoqpNxEfu6qCFFdV7u-0OgDmYbTT6_c4jDQ9jTZozuzxoh_qv6iOijBJ62yJLf4kvIOB-ghYwig6Cz1OLSgh_Y8EIO-6wk-aSZWXS7l7xnM7_8BBOzKxVG73YfPQi-9fPVnee8Cijd3RU5_uUw4yh5I87wYZzPzN3gK6Pazf6BZ-9vs1SboEyeVtd3TiOB2aBOyS4Ntohj6eljsfXEHYN4Q",
        "e": "AQAB",
        "kid": "rsa1",
        "qi": "oMPC-4NyoDZge2bOL5SHaSNKyrcnpC1XKctcxN64lRql5zETC30Nuq0zJWwFQQfVgkS8fiMRV1NEMy98EbTAj8k63PBfp5jywIx5atQFjZ101tYEsxHG4bfAuTxIwUFALS3KLIHyGAlTognRZ3UWNX0FHwH2AJUS0LJdbCuHqGA",
        "dp": "fn0Kj1fA3zrRw5RMuvAKbRQHbU58mah9gHuyCgrleEQNuYGJ8IngOxKw6NSWyyVR_WvCBMNANRJfGWhYxilYFQ_YIlvXAZQ7bXzeu9HmEt7LuoOBt1nrlVunESGdJTJQpJFU6mjjdXC-NenGuab9g365JPOAnzTIa6IMEqgB9Tc",
        "dq": "134JQdXBsriSVnYW3GZySIPBJcNJUhSqjrqzxjeT7XLAEpPNEtfU3mY7FZvVsA722mqt5O4veXFrKeG0M1DKdUpsT4-ltTQUg2dcgTY4Q8bre7ke32E73xZGEiaGFDkyY3VobXfciFmOn355PG49j14AsjroxGMel14RYK9Ni5E",
        "n": "11flaUmrserMifKmz8ONBbWWfCRAFjerniHvW1kIXP62hVI8xDWipojhLo7Ai-tV0jA4WSyodlfaA1NRXzZbbAC8pOOpPGkwLBHyOhcB4zpwTykvz5Ype3Mex20XIdBAc0DR4i8d9ZEpnIBqmPcmwP0gANmX82RZd3gZKxQ-umfDXZdNJqy2g1kawRjIP_CpznYzX8MY1OCfjZYXXsZvVeC2v0pRRre9tJEilCZEBY42jJyf6bY2cZhu2KTIpIx1_8Y2JB6AAAdnPsEgNRxMe0g5ShCmDvc93m9GsLZSy4Nm0pOWWVbx7-JK6-QWBdaiG_fAT_2OOndDL-hL1Ry3Jw"
      },
      {
        "p": "04huYoKFVGrsAaJ2djo8YUnmdbz4_Dt5a3A1r32dZhcY6gJYLtQmLzzSt1m7Z3p8UsZ8eerDBYFQYWlMIDhmsNCeKw5kE6LuSi1JGE3boveKcIpBrI5oxIduZUTr8QBltcKuklzR_Nra-JIrR_Nkgd6nA7C7VPC3MDSmQpbAyx8",
        "kty": "RSA",
        "q": "0wOm2SerufATV0ZWKOZmAgWlQ9qPATVn-FuZsEKMw4LOjlYba63iuHPWhcvgEJEgqP4DKrCn49HI95eDbYpoCpfYDCBNN8K7Q_k3P8T7a_FN-xNc9OtG80lNiFb93CSwJndGhsXoaLjGFcTpIC9RnPzi_tC3OSvdHaXgymkwEyU",
        "d": "F02mbTRlQ9J5DKkCfCgkkE_KLOmAr5VTkh2fhOZwsZ78_CNlK1NBr1a0tNpdZbn0Rt31oKLHKhfw6DcV2koso3IZVJynHHvlX3DPI8f6csXLJZGKhG7Zh6X-WnibD65h9nZo8QUd_uRdEfx6aqxPNzVKxeAX6lh7pkvkSRB7zoX_H5Vf7pn9fWXzLDB1BQacFRtdqnEpiLur-Z1CcjC68DcRGrl-EqA37ZohnVPD7T7Lw14V_nfz5rrQZGqLImCOy5Daz4ZuRBXaiR47fkWfyy7PSLnwIAHIe24328zDygNSYohw84iY0440xlAutU8wSWhrMpn4PWB_fWKiFnbiCQ",
        "e": "AQAB",
        "kid": "rsa2",
        "qi": "CnkIYAjTvJ7FS0uQgzTDtdMhi87s5r2wbGjg82HUWyRDsb7Xadf85K0aqgc5UKiM-ZHA-uc9PMDyxxnxAz4MrxVr12Gx2SS_dPFoPYVnr2jE9nQLWDUqitES1NMozhTsngmfVfgux9Rr4-rJFHP10jU9hjh9ELyv4BGoopFaFYU",
        "dp": "PKRMIOmSF5Vfe0h79YDUdn-dFfvK1WSndmmAuoKkcPQGLG_nn-Up3Tjun6mV1CCFK6XDYaAvsJBV3tc0_7ermNz1uxoxDoATGCjLX7BSWvHkOkcodLOOzWiV-Wo8yaZ3ghgBu-voNzj2tXQR_FqGYcTW1HQCAGPR4Hv3ffeB5bk",
        "dq": "T0fM9UPJane_3IeJsZ4270WNvTuthQe6w34u9SFb380SrDm_3aocR68GB4j861yLh9K__9cVjSvxtMyRTaO1yV0ZiBzOxkd8smI9FtF8oSatZMr2nvKS2q9M2Q_OG1sRb7JapiJPWqEpk15DNJtyr2f4JULnxCwtzIjZN1NkA50",
        "n": "rlx3cmO8jN3q32ndXusuoWb7MrMXgEcsJVQiU_BhgNpYEJyyA5QBmH01y04_itbCvBzGHll8y768Zv76ohgcOOW_XtdlPyF1-GgBxVg-Ro-tj5zdP4znFXfNRn6ZGY60lvCFfh3XX-eQp7EZQbHI-wRask991d2-56PzorrJWHiYLza-C87MG-i2hxyQUs5vsdl92J1pqm8_4mjRZhARBQWGXlB-80TdsG3Tl5MliaHdWNJWQ_f43TfG0OfbiCtFITIoHMESL4IO9AxygnYznPXWIoWMsXpsEKUcOgzNqhngQWe9rvJKG5Cy0O0Y0-IJexTEuF68CZpwGra75cCoew"
      }
    ]
  }
  ```

  In this example `IAM_JWK_DEFAULT_KEY=rsa1` is kept unchanged.

- Secondly, after the new key is cached by all of the clients, the signing key should be changed as the
  new key via the `IAM_JWK_DEFAULT_KEY` variable. The old key should still be kept in
  the keystore, otherwise all the current tokens would stop working. Before removing it from the
  key store, waiting as long the longest allowed token lifetime would be enough for the old tokens to expire
  and the clients will be left with only tokens that are signed with new key.

  So in the example default key should be set as `IAM_JWK_DEFAULT_KEY=rsa2`.

- As the last step, old key can be removed from the keystore. So only the tokens that were signed with the new
  key will be valid after this.

  So the keystore would be like following:

  ```json
  {
    "keys": [
      {
        "p": "04huYoKFVGrsAaJ2djo8YUnmdbz4_Dt5a3A1r32dZhcY6gJYLtQmLzzSt1m7Z3p8UsZ8eerDBYFQYWlMIDhmsNCeKw5kE6LuSi1JGE3boveKcIpBrI5oxIduZUTr8QBltcKuklzR_Nra-JIrR_Nkgd6nA7C7VPC3MDSmQpbAyx8",
        "kty": "RSA",
        "q": "0wOm2SerufATV0ZWKOZmAgWlQ9qPATVn-FuZsEKMw4LOjlYba63iuHPWhcvgEJEgqP4DKrCn49HI95eDbYpoCpfYDCBNN8K7Q_k3P8T7a_FN-xNc9OtG80lNiFb93CSwJndGhsXoaLjGFcTpIC9RnPzi_tC3OSvdHaXgymkwEyU",
        "d": "F02mbTRlQ9J5DKkCfCgkkE_KLOmAr5VTkh2fhOZwsZ78_CNlK1NBr1a0tNpdZbn0Rt31oKLHKhfw6DcV2koso3IZVJynHHvlX3DPI8f6csXLJZGKhG7Zh6X-WnibD65h9nZo8QUd_uRdEfx6aqxPNzVKxeAX6lh7pkvkSRB7zoX_H5Vf7pn9fWXzLDB1BQacFRtdqnEpiLur-Z1CcjC68DcRGrl-EqA37ZohnVPD7T7Lw14V_nfz5rrQZGqLImCOy5Daz4ZuRBXaiR47fkWfyy7PSLnwIAHIe24328zDygNSYohw84iY0440xlAutU8wSWhrMpn4PWB_fWKiFnbiCQ",
        "e": "AQAB",
        "kid": "rsa2",
        "qi": "CnkIYAjTvJ7FS0uQgzTDtdMhi87s5r2wbGjg82HUWyRDsb7Xadf85K0aqgc5UKiM-ZHA-uc9PMDyxxnxAz4MrxVr12Gx2SS_dPFoPYVnr2jE9nQLWDUqitES1NMozhTsngmfVfgux9Rr4-rJFHP10jU9hjh9ELyv4BGoopFaFYU",
        "dp": "PKRMIOmSF5Vfe0h79YDUdn-dFfvK1WSndmmAuoKkcPQGLG_nn-Up3Tjun6mV1CCFK6XDYaAvsJBV3tc0_7ermNz1uxoxDoATGCjLX7BSWvHkOkcodLOOzWiV-Wo8yaZ3ghgBu-voNzj2tXQR_FqGYcTW1HQCAGPR4Hv3ffeB5bk",
        "dq": "T0fM9UPJane_3IeJsZ4270WNvTuthQe6w34u9SFb380SrDm_3aocR68GB4j861yLh9K__9cVjSvxtMyRTaO1yV0ZiBzOxkd8smI9FtF8oSatZMr2nvKS2q9M2Q_OG1sRb7JapiJPWqEpk15DNJtyr2f4JULnxCwtzIjZN1NkA50",
        "n": "rlx3cmO8jN3q32ndXusuoWb7MrMXgEcsJVQiU_BhgNpYEJyyA5QBmH01y04_itbCvBzGHll8y768Zv76ohgcOOW_XtdlPyF1-GgBxVg-Ro-tj5zdP4znFXfNRn6ZGY60lvCFfh3XX-eQp7EZQbHI-wRask991d2-56PzorrJWHiYLza-C87MG-i2hxyQUs5vsdl92J1pqm8_4mjRZhARBQWGXlB-80TdsG3Tl5MliaHdWNJWQ_f43TfG0OfbiCtFITIoHMESL4IO9AxygnYznPXWIoWMsXpsEKUcOgzNqhngQWe9rvJKG5Cy0O0Y0-IJexTEuF68CZpwGra75cCoew"
      }
    ]
  }
  ```

So with this update method, service wasn't interrupted for any user. And the key has been changed to the new key.

[jwk-generator]: {{< ref "/docs/getting-started/jwk" >}}
