# Data Service Consumer with Verifiable Credentials

This describes how to setup the environment of a data service consuming organisation within the i4Trust trusted data space with the example of the fictitious organisations

* Packet Delivery Company being the service provider (part of [Tutorial](../Data-Service-Provider))
* Happy Pets Inc. being the consuming organisation

using [Verifiable Credentials(VC)](https://www.w3.org/TR/vc-data-model/) for authentication and authorization.

Needed for issuing credentials that can be used against the authentication of the service provider's environment are the following
* [WaltId](https://github.com/walt-id/waltid-ssikit)
* [Keycloak](https://www.keycloak.org/) with the [VC Issuer plugin](https://github.com/wistefan/keycloak-vc-issuer)

Furthermore are needed, but not covered in this tutorial:
* VC compatible Wallet for executing the [OpenId Connect for Verifiable Credentials Issuing(OIDC4VCI)](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) flow
* issuer needs to be registered in a [EBSI-compatible trusted-issuers-registry](https://api-pilot.ebsi.eu/docs/apis/trusted-issuers-registry/latest#/) as a trust anchor(f.e. the iShare satellite)

![Components](./img/components.png "Components")

Note that no Shop System is provided as part of the example. In this use case example, it is assumed that there are already 
users registered at Happy Pets and which have placed orders. More precisely, this means that there are already 
existing delivery order entities within the Context Broker of 
the [Data Service Provider](../Data-Service-Provider).

## Installation

To configure the route for accessing the deployed components, adapt the values.yaml of the keycloak chart according to the [chart documentation](https://github.com/bitnami/charts/blob/main/bitnami/keycloak/values.yaml#L529)

To install the components execute the following commands

```
helm dependency build waltid/
helm install -ndemo waltid ./waltid/ -f ./waltid/values.yaml
helm dependency build keycloak/
helm install -ndemo keycloak ./keycloak/ -f ./keycloak/values.yaml
```

## Overview of the users/credentials

With the predefined value files, users and DIDs for the necessary parties are imported as follows:

|   |   |
|---|---|
|DID of consumer's authentication authority | did:key:z6MkigCEnopwujz8Ten2dzq91nvMjqbKQYcifuZhqBsEkH7g |
|DID of provider's authentication authority | did:key:z6MkigCEnopwujz8Ten2dzq91nvMjqbKQYcifuZhqBsEkH7h |
|Password for 'normal-user'   | normal-user  |
|Password for 'premium-user'   | premium-user  |

## Validation of components
### WaltId

WaltId is the component that is used for creating Verifiable Credentials that match a given template using a defined key. The template is provided on startup and the key is registered by the Keycloak VC Issuer Plugin.

To validate that the template was successfully imported, following command can be used in the k8s cluster:

```
curl -X 'GET' 'http://waltid-vcwaltid:7001/v1/templates/PacketDeliveryService' -H 'accept: application/json'
```
with an expected output of
```
{
    "type": [
        "VerifiableCredential",
        "PacketDeliveryService"
    ],
    "@context": [
        "https://www.w3.org/2018/credentials/v1"
    ],
    "id": "urn:uuid:3add94f4-28ec-42a1-8704-4e4aa51006b4",
    "issuer": "did:ebsi:2A9BZ9SUe6BatacSpvs1V5CdjHvLpQ7bEsi2Jb6LdHKnQxaN",
    "issuanceDate": "2021-08-31T00:00:00Z",
    "issued": "2021-08-31T00:00:00Z",
    "validFrom": "2021-08-31T00:00:00Z",
    "credentialSchema": {
        "id": "https://raw.githubusercontent.com/hesusruiz/dsbamvf/main/schemas/PacketDeliveryService/2022-10/schema.json",
        "type": "FullJsonSchemaValidator2021"
    },
    "credentialSubject": {
        "id": "did:ebsi:2AEMAqXWKYMu1JHPAgGcga4dxu7ThgfgN95VyJBJGZbSJUtp",
        "familyName": "User",
        "firstName": "Some",
        "roles": [
            {
                "names": [
                    "someRole"
                ],
                "target": "did:some:id"
            }
        ]
    }
}
```

To validate that the signing key was successfully registered by the Keycloak VC Issuer Plugin, the following command can be used

```
curl -X 'GET' 'http://waltid-vcwaltid:7002/keys/z6MkigCEnopwujz8Ten2dzq91nvMjqbKQYcifuZhqBsEkH7g'   -H 'accept: application/json'
```
with an expected output of
```
{
    "algorithm": "EdDSA_Ed25519",
    "cryptoProvider": "SUN",
    "keyId": {
        "id": "z6MkigCEnopwujz8Ten2dzq91nvMjqbKQYcifuZhqBsEkH7g"
    },
    "keyPair": {},
    "keysetHandle": null
}
```

### Keycloak

Keycloak IDM is used for managing the users in the consumer space and offering verifiable credentials for accessing the data provider.

A GUI for providing Verifiable Credentials to be imported into the wallet can be found under

https://<YOUR_CONFIGURED_DOMAIN>/realms/fiware-server/account/#/verifiable-credentials

and are provided via QR code like
![CredentialsQR](./img/credential_issuance.png "CredentialsQR")