
.. include:: ../common/common_definitions.rst

.. _pid_data_model.rst:

PID Data Model
++++++++++++++

The Person Identification Data (PID) is issued by the PID Provider following national laws and allows a natural person to be authenitcated and identified. 

The User attributes carried in the Italian PID are:

    - Current Family Name
    - Current First Name
    - Date of Birth
    - Place of Birth
    - Unique Identifier
    - Taxpayer identification number

The italian PID is extended according to the `OpenID Identity Assurance Profile [OIDC.IDA] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html>`_, that enables the binding of the PID to a national trust framework, giving all the evidence of the identity proofing procedures underlying the PID issuing in both remote and proximity flows.

The PID data format and the mechanism through which it is issued into the Wallet Instance and presented to a RP will be detailed in the next sections. 

        

SD-JWT
======

The PID is given as a Verifiable Credential with JSON payload based on the `Selective Disclosure JWT format <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-selective-disclosure-jwt-04>`_ as specified in `[draft-terbu-sd-jwt-vc-latest] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-oauth-sd-jwt-vc.html>`__.

An SD-JWT is a JWT that MUST be signed using the Issuer's private key. The SD-JWT payload of the MUST contain the **_sd_alg** claim described in `[SD-JWT]. Section 5.1.2. <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-selective-disclosure-jwt-04>`_ and other claims specified in this section, some of them may be selectively disclosable claims. 

The claim **_sd_alg** indicates the hash algorithm used by the Issuer to generate the digests over the salts and the claim values. The **_sd_alg** claim MUST be set to one of the specified algorithms in Section :ref:`Cryptographic Algorithms <supported_algs>`.

Selectively disclosable claims are omitted from the SD-JWT. Instead, the digests of the respective disclosures and decoy digests are contained as an array in a new JWT claim, **_sd**. 

Each digest value ensures the integrity of, and maps to, the respective Disclosure. Digest values are calculated using a hash function over the disclosures, each of which contains 

  - a random salt, 
  - the claim name (only when the claim is an object property), 
  - the claim value. 
  
The Disclosures are sent to the Holder together with the SD-JWT in the *Combined Format for Issuance* that MUST be an ordered series of base64url-encoded values, each separated from the next by a single tilde ('~') character as follows:

.. code-block::

  <SD-JWT>~<Disclosure 1>~<Disclosure 2>~...~<Disclosure N>

See `[draft-terbu-sd-jwt-vc-latest] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-oauth-sd-jwt-vc.html>`_ and `[SD-JWT] <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-selective-disclosure-jwt-04>`__ for more details. 

SD-JWT parameters
-----------------

The JOSE header contains the following mandatory parameters:

.. _pid_jose_header:

.. list-table:: 
  :widths: 20 60 20
  :header-rows: 1

  * - **Claim**
    - **Description**
    - **Reference**
  * - **typ**
    - MUST be set to ``vc+sd-jwt`` as defined in `[draft-terbu-sd-jwt-vc-latest] <https://www.ietf.org/archive/id/draft-terbu-sd-jwt-vc-02.html>`__. 
    - `[RFC7515, Section 4.1.9] <https://datatracker.ietf.org/doc/html/rfc7515#section-4.1.9>`_.
  * - **alg**
    - Signature Algorithm. 
    - `[RFC7515, Section 4.1.1] <https://datatracker.ietf.org/doc/html/rfc7515#section-4.1.1>`_.
  * - **kid**
    - Unique identifier of the public key. 
    - `[RFC7515, Section 4.1.8] <https://datatracker.ietf.org/doc/html/rfc7516.html#section-4.1.8>`_.
  * - **trust_chain**
    - JSON array containing the trust chain that proves the reliability of the issuer of the JWT. 
    - `[OIDC-FED, Section 3.2.1] <https://openid.net/specs/openid-connect-federation-1_0.html#name-trust-chain-header-paramete>`_.

The following claims MUST be in the JWT payload and MUST NOT be included in the disclosures,  i.e. cannot be selectively disclosed. 

.. list-table:: 
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **iss**
      - The PID Provider identifier as URL string (the issuer of this JWT)
      - `[RFC7519, Section 4.1.1] <https://www.iana.org/go/rfc7519>`_.
    * - **sub**
      - Thumbprint of the JWK in the ``cnf`` parameter
      - `[RFC7519, Section 4.1.2] <https://www.iana.org/go/rfc7519>`_.
    * - **jti**
      - Unique Token ID identifier of this JWT. It SHOULD be a String in *uuid4* format.
      - `[RFC7519, Section 4.1.7] <https://www.iana.org/go/rfc7519>`_.
    * - **iat**
      - UNIX Timestamp with the time of JWT issuance, coded as NumericDate as indicated in :rfc:`7519`. 
      - `[RFC7519, Section 4.1.6] <https://www.iana.org/go/rfc7519>`_.
    * - **exp**
      - UNIX Timestamp with the expiry time of the JWT, coded as NumericDate as indicated in :rfc:`7519`.
      - `[RFC7519, Section 4.1.4] <https://www.iana.org/go/rfc7519>`_.
    * - **status**
      - HTTPS URL where the credential validity status is available
      - `[SD-JWT-VC. Section 4.2.2.2] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-sd-jwt-vc.html#section-4.2.2.2>`_.
    * - **cnf**
      - JSON object containing the proof-of-possession key materials. By including a **cnf** (confirmation) claim in a JWT, the issuer of the JWT declares that the presenter is in control of the private key related to the public one defined in the **cnf** parameter. The recipient MUST cryptographically verify that the presenter is in control of that key. 
      - `[RFC7800, Section 3.1] <https://www.iana.org/go/rfc7800>`_.
    * - **type**
      - Credential type as a string, MUST be set to ``PersonIdentificationData``.
      - `[draft-terbu-sd-jwt-vc-latest. Section 4.2.2.2] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-sd-jwt-vc.html#section-4.2.2.2>`__.
    * - **verified_claims**
      - JSON object containing the following sub-elements: 

            -   **verification**; 
            -   **claims**.
      - `[OIDC.IDA. Section 5] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5>`_.


Verification field 
------------------

The ``verification`` claim contain the information as sub claims regarding the identity proofing evidence during the issuing phase of the PID.  Some of these additional claims MAY be included in the Disclosures and MAY be selectively disclosed and they are given in the following tables that also specify whether a claim is selectively disclosable (SD) or not (NSD).

The ``verification`` claim is a JSON structure with all the following mandatory sub-claims.

.. list-table:: 
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **trust_framework**
      - [NSD]. MUST be set to eidas
      - `[OID.IDA. Section 5.1] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5.1>`_
    * - **assurance_level**
      - [NSD]. MUST be set to high
      - `[OID.IDA. Section 5.1] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5.1>`_
    * - **evidence**
      - [SD]. JSON Array. Each element is the electronic evidence of the user identification during the PID issuing phase. It MUST contain at least the following claims:

            - **type**: MUST be set to ``electronic_record``
            - **record**: JSON object (see the table below)
      - `[OID.IDA. Section 5.1] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5.1>`_


The ``record`` MUST have at least the following sub parameters:

.. list-table:: 
  :widths: 20 60 20
  :header-rows: 1

  * - **Claim**
    - **Description**
    - **Reference**
  * - **type** 
    - identification of the national eID framework used by the User. For example ``eidas.it.cie`` means that the CIE id identification scheme is used by the User. 
    - `[OID.IDA. Section 5.1.1.2] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5.1.1.2>`_
  * - **source**
    - JSON Object cointaining the follwoing mandatory claims:

      - **organization_name**: Name of the Organization handling the eID used by the User
      - **organization_id**: Identification code for the Organization. It MUST be set to the *IPA Code* of the Organization
      - **country_code**: String representing country in `[ISO3166-1] Alpha-2 (e.g., IT) or [ISO3166-3] syntax <https://www.iso.org/iso-3166-country-codes.html>`_.
    - `[OID.IDA. Section 5.1.1.2] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5.1.1.2>`_

.. warning::
  Note that the sub-claims of the **evidence** parameter are not selectively disclosable separately, thus, for example, the User cannot give only the *record type* without disclosure the *record source* (organization name, identifier and country that hendles the User identity proofing). 

Claims field 
------------

The ``claims`` parameter contains the user attributes claims with the following mandatory fields:


.. list-table:: 
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **given_name**
      - [SD]. Current First Name
      - `[OpenID Connect Core 1.0, Section 5.1] <http://openid.net/specs/openid-connect-core-1_0.html>`_
    * - **family_name**
      - [SD]. Current Family Name
      - `[OpenID Connect Core 1.0, Section 5.1] <http://openid.net/specs/openid-connect-core-1_0.html>`_
    * - **birthdate**
      - [SD]. Date of Birth
      - `[OpenID Connect Core 1.0, Section 5.1] <http://openid.net/specs/openid-connect-core-1_0.html>`_
    * - **place_of_birth**
      - [SD]. Place of Birth. JSON Object with the following subclaims:

        - **country**
        - **locality**
      - `[OpenID Connect for Identity Assurance 1.0, Section 4] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-4>`_
    * - **unique_id**
      - [SD]. Unique citizen identifier (ID ANPR) given by the National Register of the Resident Population (ANPR). It MUST be set according to `ANPR rules <https://www.anagrafenazionale.interno.it/anpr/notizie/identificativo-unico-nazionale-idanpr/>`_
      - This specification
    * - **tax_id_code**
      - [SD]. National tax identification code of natural person as a String format. It MUST be set according to ETSI EN 319 412-1. For example ``TINIT-<ItalianTaxIdentificationNumber>``
      - This specification



Non-normative examples
----------------------

In the following, we provide a non-normative example of PID VC in JSON.

.. code-block:: JSON

  {
  "verified_claims": {
      "verification": {
        "trust_framework": "eidas",
        "assurance_level": "high",
        "evidence": [
          {
            "type": "electronic_record",
            "record": {
              "type": "eidas.it.cie",
              "source": {
                "organization_name": "Ministero dell'Interno",
                "organization_id": "m_it",
                "country_code": "IT"
              }
            }
          }
        ]
      },
      "claims": {
        "unique_id":
          "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "given_name": "Mario",
        "family_name": "Rossi",
        "birthdate": "1980-01-10",
        "place_of_birth": {
          "country": "IT",
          "locality": "Rome"
        },
        "tax_id_code": "TINIT-XXXXXXXXXXXXXXXX"
      }
    }
  }

The corresponding SD-JWT verson for PID is given by

.. code-block:: JSON

  {
     "typ":"vc+sd-jwt",
     "alg":"RS512",
     "kid":"dB67gL7ck3TFiIAf7N6_7SHvqk0MDYMEQcoGGlkUAAw",
     "trust_chain" : [
      "NEhRdERpYnlHY3M5WldWTWZ2aUhm ...",
      "eyJhbGciOiJSUzI1NiIsImtpZCI6 ...",
      "IkJYdmZybG5oQU11SFIwN2FqVW1B ..."
     ]
  }

.. code-block:: JSON

  {
    "iss": "https://pidprovider.example.org",
    "sub": "NzbLsXh8uDCcd7noWXFZAfHkxZsRGC9Xs...",
    "jti": "urn:uuid:6c5c0a49-b589-431d-bae7-219122a9ec2c",
    "iat": 1541493724,
    "exp": 1541493724,
    "status": "https://pidprovider.example.org/status",
    "cnf": {
      "jwk": {
        "kty": "RSA",
        "use": "sig",
        "n": "1Ta-sE …",
        "e": "AQAB",
        "kid": "YhNFS3YnC9tjiCaivhWLVUJ3AxwGGz_98uRFaqMEEs"
      }
    },
    "type": "PersonIdentificationData",
    "verified_claims": {
      "verification": {
        "_sd": [
          "OGm7ryXgt5Xzlevp-Hu-UTk0a-TxAaPAobqv1pIWMfw"
        ],
        "trust_framework": "eidas",
        "assurance_level": "high"
      },
      "claims": {
        "_sd": [
          "8JjozBfovMNvQ3HflmPWy4O19Gpxs61FWHjZebU589E",
          "BoMGktW1rbikntw8Fzx_BeL4YbAndr6AHsdgpatFCig",
          "CFLGzentGNRFngnLVVQVcoAFi05r6RJUX-rdbLdEfew",
          "JU_sTaHCngS32X-0ajHrd1-HCLCkpT5YqgcfQme168w",
          "VQI-S1mT1Kxfq2o8J9io7xMMX2MIxaG9M9PeJVqrMcA",
          "zVdghcmClMVWlUgGsGpSkCPkEHZ4u9oWj1SlIBlCc1o"
        ]
      }
    },
    "_sd_alg": "sha-256"
  }

In the following the disclosure list is given

Claim **evidence**:

-  SHA-256 Hash: ``OGm7ryXgt5Xzlevp-Hu-UTk0a-TxAaPAobqv1pIWMfw``
-  Disclosure:
   ``WyIyR0xDNDJzS1F2ZUNmR2ZyeU5STjl3IiwgImV2aWRlbmNlIiwgW3sidHlw``
   ``ZSI6ICJlbGVjdHJvbmljX3JlY29yZCIsICJyZWNvcmQiOiB7InR5cGUiOiAi``
   ``ZWlkYXMuaXQuY2llIiwgInNvdXJjZSI6IHsib3JnYW5pemF0aW9uX25hbWUi``
   ``OiAiTWluaXN0ZXJvIGRlbGwnSW50ZXJubyIsICJvcmdhbml6YXRpb25faWQi``
   ``OiAibV9pdCIsICJjb3VudHJ5X2NvZGUiOiAiSVQifX19XV0``
-  Contents: ``["2GLC42sKQveCfGfryNRN9w", "evidence", [{"type":``
   ``"electronic_record", "record": {"type": "eidas.it.cie",``
   ``"source": {"organization_name": "Ministero dell'Interno",``
   ``"organization_id": "m_it", "country_code": "IT"}}}]]``

Claim **unique_id**:

-  SHA-256 Hash: ``BoMGktW1rbikntw8Fzx_BeL4YbAndr6AHsdgpatFCig``
-  Disclosure:
   ``WyJlbHVWNU9nM2dTTklJOEVZbnN4QV9BIiwgInVuaXF1ZV9pZCIsICJ4eHh4``
   ``eHh4eC14eHh4LXh4eHgteHh4eC14eHh4eHh4eHh4eHgiXQ``
-  Contents: ``["eluV5Og3gSNII8EYnsxA_A", "unique_id",``
   ``"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"]``

Claim **given_name**:

-  SHA-256 Hash: ``zVdghcmClMVWlUgGsGpSkCPkEHZ4u9oWj1SlIBlCc1o``
-  Disclosure:
   ``WyI2SWo3dE0tYTVpVlBHYm9TNXRtdlZBIiwgImdpdmVuX25hbWUiLCAiTWFy``
   ``aW8iXQ``
-  Contents: ``["6Ij7tM-a5iVPGboS5tmvVA", "given_name", "Mario"]``

Claim **family_name**:

-  SHA-256 Hash: ``VQI-S1mT1Kxfq2o8J9io7xMMX2MIxaG9M9PeJVqrMcA``
-  Disclosure:
   ``WyJlSThaV205UW5LUHBOUGVOZW5IZGhRIiwgImZhbWlseV9uYW1lIiwgIlJv``
   ``c3NpIl0``
-  Contents: ``["eI8ZWm9QnKPpNPeNenHdhQ", "family_name", "Rossi"]``

Claim **birthdate**:

-  SHA-256 Hash: ``CFLGzentGNRFngnLVVQVcoAFi05r6RJUX-rdbLdEfew``
-  Disclosure:
   ``WyJRZ19PNjR6cUF4ZTQxMmExMDhpcm9BIiwgImJpcnRoZGF0ZSIsICIxOTgw``
   ``LTAxLTEwIl0``
-  Contents: ``["Qg_O64zqAxe412a108iroA", "birthdate", "1980-01-10"]``

Claim **place_of_birth**:

-  SHA-256 Hash: ``JU_sTaHCngS32X-0ajHrd1-HCLCkpT5YqgcfQme168w``
-  Disclosure:
   ``WyJBSngtMDk1VlBycFR0TjRRTU9xUk9BIiwgInBsYWNlX29mX2JpcnRoIiwg``
   ``eyJjb3VudHJ5IjogIklUIiwgImxvY2FsaXR5IjogIlJvbWUifV0``
-  Contents:
   ``["AJx-095VPrpTtN4QMOqROA", "place_of_birth", {"country":``
   ``"IT", "locality": "Rome"}]``

Claim **tax_id_code**:

-  SHA-256 Hash: ``8JjozBfovMNvQ3HflmPWy4O19Gpxs61FWHjZebU589E``
-  Disclosure:
   ``WyJQYzMzSk0yTGNoY1VfbEhnZ3ZfdWZRIiwgInRheF9pZF9jb2RlIiwgIlRJ``
   ``TklULVhYWFhYWFhYWFhYWFhYWFgiXQ``
-  Contents: ``["Pc33JM2LchcU_lHggv_ufQ", "tax_id_code",``
   ``"TINIT-XXXXXXXXXXXXXXXX"]``

The combined format for the PID issuance is given by

.. code-block::

  eyJ0eXAiOiJ2YytzZC1qd3QiLCJhbGciOiJSUzUxMiIsImtpZCI6ImQxMjZhNmE4NTZmNzcyNDU2MDQ4NGZhOWRjNTlkMTk1IiwidHJ1c3RfY2hhaW4iOlsiTkVoUmRFUnBZbmxIWTNNNVdsZFdUV1oyYVVobSAuLi4iLCJleUpoYkdjaU9pSlNVekkxTmlJc0ltdHBaQ0k2IC4uLiIsIklrSllkbVp5Ykc1b1FVMTFTRkl3TjJGcVZXMUIgLi4uIl19.eyJpc3MiOiJodHRwczovL3BpZHByb3ZpZGVyLmV4YW1wbGUub3JnIiwic3ViIjoiTnpiTHNYaDh1RENjZDdub1dYRlpBZkhreFpzUkdDOVhzLi4uIiwianRpIjoidXJuOnV1aWQ6NmM1YzBhNDktYjU4OS00MzFkLWJhZTctMjE5MTIyYTllYzJjIiwiaWF0IjoxNTQxNDkzNzI0LCJleHAiOjE1NDE0OTM3MjQsInN0YXR1cyI6Imh0dHBzOi8vcGlkcHJvdmlkZXIuZXhhbXBsZS5vcmcvc3RhdHVzIiwiY25mIjp7Imp3ayI6eyJrdHkiOiJSU0EiLCJ1c2UiOiJzaWciLCJuIjoiMVRhLXNFIOKApiIsImUiOiJBUUFCIiwia2lkIjoiWWhORlMzWW5DOXRqaUNhaXZoV0xWVUozQXh3R0d6Xzk4dVJGYXFNRUVzIn19LCJ0eXBlIjoiUGVyc29uSWRlbnRpZmljYXRpb25EYXRhIiwidmVyaWZpZWRfY2xhaW1zIjp7InZlcmlmaWNhdGlvbiI6eyJfc2QiOlsiT0dtN3J5WGd0NVh6bGV2cC1IdS1VVGswYS1UeEFhUEFvYnF2MXBJV01mdyJdLCJ0cnVzdF9mcmFtZXdvcmsiOiJlaWRhcyIsImFzc3VyYW5jZV9sZXZlbCI6ImhpZ2gifSwiY2xhaW1zIjp7Il9zZCI6WyI4SmpvekJmb3ZNTnZRM0hmbG1QV3k0TzE5R3B4czYxRldIalplYlU1ODlFIiwiQm9NR2t0VzFyYmlrbnR3OEZ6eF9CZUw0WWJBbmRyNkFIc2RncGF0RkNpZyIsIkNGTEd6ZW50R05SRm5nbkxWVlFWY29BRmkwNXI2UkpVWC1yZGJMZEVmZXciLCJKVV9zVGFIQ25nUzMyWC0wYWpIcmQxLUhDTENrcFQ1WXFnY2ZRbWUxNjh3IiwiVlFJLVMxbVQxS3hmcTJvOEo5aW83eE1NWDJNSXhhRzlNOVBlSlZxck1jQSIsInpWZGdoY21DbE1WV2xVZ0dzR3BTa0NQa0VIWjR1OW9XajFTbElCbENjMW8iXX19LCJfc2RfYWxnIjoic2hhLTI1NiJ9.WzEiFaOjnobQisjTQ92JtKEXRN-2Sgvjklpu4IdC_cT2T6Tm8Z6sqbVy6n94AAEv-HFSv5JoSt6YjPDnGzOxN_W_131rILU8YaiNt8w31nRGIvHjJIC0w-hHIcG1LmvJshSMcT3RHeApRCmsO7xkHWmUsjt37dOzEagEti5i47hnZAbu7vWXsvUlBNNN8v7tJBLspO2Q0vnWhEDX1hQ7IH1b8oKh-_aQrhwVm9Bcs9CG8o6N9iqubCSpFI6Gty4ZZgHEb95knETVhw8IL10Z9P_Hr9twXZQaCCC8xrNh4afwR9TiDQzTr92m7luyvDfmzVgHCponI7VBhqmRqZVYQyDhq6EJbtRtIsYenla5NSKBjV8Etdlec94vJAHZNzue9aNUQeXae55V5m5O9wLoWhgV2vl4xV5C-N5s5Uzs08GAxo-CUaNOD3BQE9vfrT47IBCm4hUCnvDise_aWNCeKOQABV1J9_tV9lWZsECVuUuWWwELHCUXgdyiA3QtUtXz



MDOC-CBOR
=========

[TODO]


