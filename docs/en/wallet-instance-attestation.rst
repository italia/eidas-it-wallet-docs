.. include:: ../common/common_definitions.rst

.. _wallet-instance-attestation.rst:

Wallet Instance Attestation
+++++++++++++++++++++++++++

The Wallet Instance Attestation is data containing details about the Wallet Provider, the Wallet Solution, the Wallet Instance, and the device's security level where the Wallet Instance is installed. It generally attests the **authenticity, integrity, security, privacy, and trust** of a specific Wallet Instance. The Wallet Instance Attestation MUST contain the Wallet Instance public key.

General Properties
------------------

The objectives include:

- Ensuring that the Wallet Instance maintains a level of **integrity** that is capable of preventing any manipulation or forgery attempts by unauthorized third parties.
- Having the Wallet Provider issue a certificate of conformity to assure that the previously mentioned security and trust objectives are fulfilled.

To meet these requirements, it is necessary for each Wallet Instance to issue an attestation of conformity, guaranteeing its security and compliance with the Trust Model.

This verifiable attestation, called **Wallet Instance Attestation**, must be electronically signed by its issuer.

.. hint::
  Given that the Wallet Instance does not represent an accredited entity and does not belong to an organization but resides on the User's device, the Trust Model, based on sustainability and scalability criteria, must delegate the task of issuing the **Wallet Instance Attestation** to the **Wallet Provider**.

Requirements
------------

The following requirements are assumed for the Wallet Instance Attestation:

1. **Efficiency**: The Wallet Instance Attestation should use an efficient format like JSON Web Token (JWT) for light and fast data management, and compliance with various formats used for eudiw solutions.
2. **Simplicity**: The Wallet Provider should be based on a REST architecture for issuing Wallet Instance Attestations.
3. **Public key holder binding**: The Wallet Instance Attestation must be securely linked to the Wallet Instance public key.
4. **Issued and signed by an accredited Wallet Provider**: The Wallet Instance Attestation must be issued and signed by an accredited and reliable Wallet Provider, thereby providing integrity and authenticity to the attestation.
5. **Authenticity/Genuineness of the Wallet Instance**: The Wallet Instance Attestation must ensure the integrity and authenticity of the Wallet Instance, verifying that it was accurately created and provided by the Wallet Provider. ⚠️
6. **Ability to request multiple claims for several public keys**: Each Wallet Instance should be able to request multiple attestations for different public keys associated with it. This requirement provides a privacy-preserving measure, as the public key could be used as a tracking tool in the credentials’ disclosure phase (also see point 10 below).
7. **Reusability**: The Wallet Instance Attestation should be usable multiple times during the validity period of the attestation, allowing for repeated authentication and authorization without the need to request new attestations with each interaction.
8. **Expiration**: The Wallet Instance Attestation should have a well-defined expiration date, after which it will no longer be considered valid, thereby ensuring the security and updating of attestations over time.
9. **Revocation in case of loss/deletion of the private key**: If the private key associated with the Wallet Instance is lost or deleted, the attestation automatically becomes invalid to prevent unauthorized use of the Wallet Instance. ⚠️
10. **Pseudonymisation**: The attestations are designed to be pseudonymised (i.e., they do not contain direct references to the person, making it impossible to identify them in the absence of additional information - see art. 4(5) GDPR for a comprehensive definition). Without such a measure, the use of the attestation on multiple RPs would pose a significant risk, as it would theoretically allow the RPs to merge databases and track Users. This requirement enhances the measures adopted according to

 art. 32 GDPR.

.. attention::
  ⚠️ Implementation of points no. 5 and 9 is still under discussion. This version assumes the authenticity and non-revocability of the Wallet Instance.

High-level Design
-----------------

Static Component View
~~~~~~~~~~~~~~~~~~~~~

.. figure:: ../../images/static_view_wallet_instance_attestation.svg
   :name: Wallet Solution Schema
   :alt: The image illustrates the containment of Wallet Provider and Wallet Instances within the Wallet Solution, managed by the Wallet Provider.
   :target: https://www.plantuml.com/plantuml/uml/XP4nJuSm44VtVehBdxbnPp2iRYx6qTHIjR7SaVQ0-EqzaICDgN4ZBxpqzTUXiCkyJCaupvJXzbH2le4hiCW7A7rsAGM6ETCQn-E7RMSloi0OJzDC691FeL1QE1BMWZBeraW2Mbv4wK8VQayPT5yX9TgCQPclpdy676lnGF0ZN93DyVs3xVsrhOU70hCi0_JshwHXFJp-Rg4dIuECo96moD7xeBQbUKBEbE0EPEwuEWx6N2zj_uXqU8wbhVMhD3tjbAX1BYIl_mq0

Dynamic Component View
~~~~~~~~~~~~~~~~~~~~~~

This section describes the Wallet Instance Attestation format and how the Wallet Provider issues it.

.. figure:: ../../images/dynamic_view_sequence_wallet_instance_attestation.svg
   :name: Sequence Diagram for Wallet Instance Attestation Request
   :alt: The figure illustrates the sequence diagram for issuing a Wallet Instance Attestation, with the steps explained below.
   :target: https://www.plantuml.com/plantuml/uml/XPB1RzKm3CRl-IlCJY3nn7s7QOZ3118IGi0kkxYDLLcqJd2SLMz_FLvV6r7AnDN-_Fi-ExajXcfr6iEhh3XC24Rf2Kmh1QoMf4uTQGZPLTnpHZ6u-bv8hm0Br7tz7iUH33wAGwMdHJBpFpLVD3roN35p5qA5qusBhtsQZN7a9uBvekMLzo19GUbNfMBlib8X1_PAaUHveeIPJpTpTmrtPDjiNdrW8iE8Xc7kJgvoeyzh1VeaXYmimnyqi7EcyXP-qddnPAN9EruXYJcnsEhdf1yUrqbqC3MjnM3aOgxT5hmZ8NNrWix8MhQcH_zwMGyaIK-U5KwNgRNGB3yeFIF-kZYyBuNKE4a3VRh_5h0tVbpoTRiROLE__Y_eZOTP9W_RyZOpa5GM4YhbA2uy25fLQgrXkmDANDe7OClN7ktbXO-FyJ8jqluYpguDtVJSFc9y42MCPx04gJDa0Q5vz_LkIMATnjy0

- **Message 1**: The User starts the Wallet Instance mobile app, a new Wallet Instance Attestation is automatically obtained if the previous one results expired.
- **Message 2-3**: The Wallet Instance retrieves metadata about its Wallet Provider, including the list of supported algorithms, public keys, and endpoints.
- **Message 4**: The Wallet Instance verifies the Wallet Provider's trustworthiness by resolving the provider's trust chain to the Trust Anchor.
- **Message 5-7**: The Wallet Instance generates a new key pair and requests a ``nonce`` from the Wallet Provider to guard against replay attacks.
- **Message 8**: The Wallet Instance creates a Wallet Instance Attestation Request in JWS format, signed with the private key associated with the public key for which it seeks attestation.
- **Message 9-13**: The Wallet Instance sends the Wallet Instance Attestation Request to the Wallet Provider, which validates it and issues a signed attestation in return.
- **Message 13-14**: The Wallet Instance receives the Wallet Instance Attestation signed by the Wallet Provider and performs formal verification.
- **Message 15**: The Wallet Instance Attestation is now ready for use.

Detailed Design
---------------

The detailed design is explained below.



Format of the Wallet Instance Attestation Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To obtain a Wallet Instance Attestation from the Wallet
Provider it is necessary to send a Wallet Instance Attestation
Request from the Wallet Instance containing the associated public key
and a ``nonce`` previously requested to avoid replay attacks.

Header
^^^^^^
+-----------------------------------+-----------------------------------+
| **key**                           | **value**                         |
+-----------------------------------+-----------------------------------+
| alg                               | Algorithm to verify the token     |
|                                   | signature (es. ES256)             |
+-----------------------------------+-----------------------------------+
| kid                               | Key id of the Wallet Instance     |
+-----------------------------------+-----------------------------------+
| typ                               | Media type, in this case we use   |
|                                   | the value var+jwt (Verifiable     |
|                                   | Assertion Request JWT)            |
+-----------------------------------+-----------------------------------+

Payload
^^^^^^^
+--------+--------------------------------------+
| **key**| **value**                            |
+--------+--------------------------------------+
|| iss   || The thumbprint                      |
||       || of the JWK of the Wallet Instance   |
||       || for which the attestation is        |
||       || being requested.                    |
+--------+--------------------------------------+
|| aud   || The public url of the Wallet        |
||       || Provider                            |
+--------+--------------------------------------+
|| jti   || Unique identifier of the request.   |
||       || This parameter will be used to      |
||       || avoid replay attacks.               |
+--------+--------------------------------------+
|| type  || String. It must be set to           |
||       || ``WalletInstanceAttestationRequest``|
+--------+--------------------------------------+
|| nonce || The nonce obtained from the         |
||       || Wallet Porvider.                    |
+--------+--------------------------------------+
|| cnf   || This parameter will contain the     |
||       || configuration of the Wallet         |
||       || Instance in JSON format. Among      |
||       || the mandatory attributes there      |
||       || will be the jwk parameter           |
||       || containing the public key of the    |
||       || Wallet Instance. It will also       |
||       || contain all the information         |
||       || useful for the Wallet Provider      |
||       || to verify that the app is genuine.  |
+--------+--------------------------------------+

Below a non-normative example of the Wallet Instance Attestation
request where the decoded JWS headers and payload are separated by a comma:

.. code-block:: javascript

  {
    "alg": "ES256",
    "kid": "vbeXJksM45xphtANnCiG6mCyuU4jfGNzopGuKvogg9c",
    "typ": "var+jwt"
  }
  .
  {
    "iss": "vbeXJksM45xphtANnCiG6mCyuU4jfGNzopGuKvogg9c",
    "aud": "https://wallet-provider.example.org",
    "jti": "6ec69324-60a8-4e5b-a697-a766d85790ea",
    "type": "WalletInstanceAttestationRequest",
    "nonce" : "....."
    "cnf": {
      "jwk": {
        "crv": "P-256",
        "kty": "EC",
        "x": "4HNptI-xr2pjyRJKGMnz4WmdnQD_uJSq4R95Nj98b44",
        "y": "LIZnSB39vFJhYgS3k7jXE4r3-CoGFQwZtPBIRqpNlrg",
        "kid": "vbeXJksM45xphtANnCiG6mCyuU4jfGNzopGuKvogg9c"
      }
    },
    "iat": 1686645115,
    "exp": 1686652315
  }

Whose corresponding JWS is verifiable through the public key
of the Wallet Instance present.


Format of the Wallet Instance Attestation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A JWT was chosen as the format for the Wallet Instance Attestation.
Let's see below the various fields that compose it.

Header
^^^^^^

+-----------------------------------+-----------------------------------+
| **key**                           | **value**                         |
+-----------------------------------+-----------------------------------+
| alg                               | Algorithm to verify the token     |
|                                   | signature (es. ES256).            |
+-----------------------------------+-----------------------------------+
| kid                               | Key id used by the Wallet         |
|                                   | Provider to sign the attestation. |
+-----------------------------------+-----------------------------------+
| typ                               | Media type, in this case we use   |
|                                   | the value va+jwt (Verifiable      |
|                                   | Assertion JWT).                   |
|                                   | This parameter is currently       |
|                                   | non-standard as it is not yet     |
|                                   | registered as `IANA Media         |
|                                   | Types <https://www.iana.org/assig |
|                                   | nments/media-types/media-types.xh |
|                                   | tml>`__.                          |
+-----------------------------------+-----------------------------------+
| x5c                               | Array containing the X.509        |
|                                   | certificate (and the entire chain |
|                                   | of certificates) used to certify  |
|                                   | the public key of the issuer.     |
+-----------------------------------+-----------------------------------+
| trust_chain                       | Array containing the JWS of the   |
|                                   | trust chain relating to its       |
|                                   | issuer (Wallet Provider).         |
+-----------------------------------+-----------------------------------+

Payload
^^^^^^^

+---------------------------+------------------------------------------------+
| **key**                   | **value**                                      |
+---------------------------+------------------------------------------------+
|| iss                      || The public url of the Wallet                  |
||                          || Instance attestation issuer. See              |
||                          || the example below in this section.            |
+---------------------------+------------------------------------------------+
|| sub                      || Thumbprint value                              |
||                          || of the JWK of the Wallet Instance             |
||                          || for which the attestation is                  |
||                          || being issued.                                 |
+---------------------------+------------------------------------------------+
|| iat                      || Unix timestamp of attestation                 |
||                          || issuance time.                                |
+---------------------------+------------------------------------------------+
|| exp                      || Unix timestamp regarding the                  |
||                          || expiration date time.                         |
||                          || A good practice to avoid security             |
||                          || problems is to have a limited                 |
||                          || duration of the attestation.                  |
+---------------------------+------------------------------------------------+
|| type                     || String:                                       |
||                          || "WalletInstanceAttestation".                  |
+---------------------------+------------------------------------------------+
|| policy_uri               || URL to the privacy policy                     |
||                          || of the wallet.                                |
+---------------------------+------------------------------------------------+
|| tos_uri                  || URL to the terms                              |
||                          || of use of the Wallet Provider.                |
+---------------------------+------------------------------------------------+
|| logo_uri                 || URL of the Wallet Provider logo in SVG format |
+---------------------------+------------------------------------------------+
|| attested_security_context|| Attested security context:                    |
||                          || Represents a level of "trust" of              |
||                          || the service containing a Level Of             |
||                          || Agreement defined in the metadata             |
||                          || of the Wallet Provider.                       |
+---------------------------+------------------------------------------------+
|| cnf                      || This parameter contains the ``jwk``           |
||                          || parameter                                     |
||                          || with the public key of the Wallet             |
||                          || necessary for the holder binding.             |
+---------------------------+------------------------------------------------+
|| authorization_endpoint   || URL of the OP's OAuth 2.0                     |
||                          || Authorization Endpoint.                       |
+---------------------------+------------------------------------------------+
|| response_types_supported || JSON array containing a list of               |
||                          || the OAuth 2.0 response_type values            |
||                          || that this OP supports.                        |
+---------------------------+------------------------------------------------+
|| vp_formats_supported     || JSON object containing                        |
||                          || ``jwt_vp_json`` and ``jwt_vc_json``           |
||                          || supported algorithms array.                   |
+---------------------------+------------------------------------------------+
|| request_object_signing   || JSON array containing a list of the           |
|| _alg_values_supported    || JWS signing algorithms (alg values)           |
||                          || supported by the OP for Request Objects.      |
+---------------------------+------------------------------------------------+
|| presentation_definition  || Boolean value specifying whether the          |
|| _uri_supported           || Wallet Instance supports the transfer of      |
||                          || presentation_definition by                    |
||                          || reference, with true indicating support.      |
+---------------------------+------------------------------------------------+

.. note::
   The claim ``attested_security_context`` (Attested Security Context) is under discussion
   and must be intended as experimental.

.. note::

    The Wallet Instance Attestation JWS is signed using the private key of the Wallet Provider.

Below is an example of Wallet Instance Attestation:

.. code-block:: javascript

  {
    "alg": "ES256",
    "kid": "5t5YYpBhN-EgIEEI5iUzr6r0MR02LnVQ0OmekmNKcjY",
    "trust_chain": [
      "eyJhbGciOiJFUz...6S0A",
      "eyJhbGciOiJFUz...jJLA",
      "eyJhbGciOiJFUz...H9gw",
    ],
    "typ": "va+jwt",
    "x5c": ["MIIBjDCC ... XFehgKQA=="]
  }
  .
  {
    "iss": "https://wallet-provider.example.org",
    "sub": "vbeXJksM45xphtANnCiG6mCyuU4jfGNzopGuKvogg9c",
    "type": "WalletInstanceAttestation",
    "policy_uri": "https://wallet-provider.example.org/privacy_policy",
    "tos_uri": "https://wallet-provider.example.org/info_policy",
    "logo_uri": "https://wallet-provider.example.org/logo.svg",
    "attested_security_context": "https://wallet-provider.example.org/LoA/basic",
    "cnf":
    {
      "jwk":
      {
        "crv": "P-256",
        "kty": "EC",
        "x": "4HNptI-xr2pjyRJKGMnz4WmdnQD_uJSq4R95Nj98b44",
        "y": "LIZnSB39vFJhYgS3k7jXE4r3-CoGFQwZtPBIRqpNlrg",
        "kid": "vbeXJksM45xphtANnCiG6mCyuU4jfGNzopGuKvogg9c"
      }
    },
    "authorization_endpoint": "eudiw:",
    "response_types_supported": [
      "vp_token"
    ],
    "vp_formats_supported": {
      "jwt_vp_json": {
        "alg_values_supported": ["ES256"]
      },
      "jwt_vc_json": {
        "alg_values_supported": ["ES256"]
      }
    },
    "request_object_signing_alg_values_supported": [
      "ES256"
    ],
    "presentation_definition_uri_supported": false,
    "iat": 1687281195,
    "exp": 1687288395
  }
