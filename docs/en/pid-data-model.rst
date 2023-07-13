
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

This section contains all about the data model elements and the encoding formats as specified 
in the *SD-JWT* and the *ISO/IEC 18013-5* standards and extended by `EIDAS-ARF`_ .

SD-JWT
------

The PID is given as a digital credential with JSON payload based on the `Selective Disclosure JWT format <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-selective-disclosure-jwt-04>`_ as specified in `[draft-terbu-sd-jwt-vc-latest] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-oauth-sd-jwt-vc.html>`__.

An SD-JWT is a JWT that MUST be signed using the Issuer's private key. The SD-JWT payload of the MUST contain the **_sd_alg** claim described in `[SD-JWT]. Section 5.1.2. <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-selective-disclosure-jwt-04>`_ and other claims specified in this section, some of them may be selectively disclosable claims. 

The claim **_sd_alg** indicates the hash algorithm used by the Issuer to generate the digests over the salts and the claim values. The **_sd_alg** claim MUST be set to one of the specified algorithms in Section :ref:`Cryptographic Algorithms <supported_algs>`.

Selectively disclosable claims are omitted from the SD-JWT. Instead, the digests of the respective disclosures and decoy digests are contained as an array in a **_sd** JWT claim. 

Each digest value ensures the integrity of, and maps to, the respective Disclosure. Digest values are calculated using a hash function over the disclosures, each of which contains 

  - a random salt, 
  - the claim name (only when the claim is an object property), 
  - the claim value. 
  
The Disclosures are sent to the Holder together with the SD-JWT in the *Combined Format for Issuance* that MUST be an ordered series of base64url-encoded values, each separated from the next by a single tilde ('~') character as follows:

.. code-block::

  <SD-JWT>~<Disclosure 1>~<Disclosure 2>~...~<Disclosure N>

See `[draft-terbu-sd-jwt-vc-latest] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-oauth-sd-jwt-vc.html>`_ and `[SD-JWT] <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-selective-disclosure-jwt-04>`__ for more details. 

SD-JWT Data Elements
^^^^^^^^^^^^^^^^^^^^

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
      - Credential type as a string, MUST be set to ``eu.eudiw.pid.it``.
      - `[draft-terbu-sd-jwt-vc-latest. Section 4.2.2.2] <https://vcstuff.github.io/draft-terbu-sd-jwt-vc/draft-terbu-sd-jwt-vc.html#section-4.2.2.2>`__.
    * - **verified_claims**
      - JSON object containing the following sub-elements: 

            -   **verification**; 
            -   **claims**.
      - `[OIDC.IDA. Section 5] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html#section-5>`_.

Since the Italian PID is extended according to the `OpenID Identity Assurance Profile [OIDC.IDA] <https://openid.net/specs/openid-connect-4-identity-assurance-1_0-13.html>`_, 
that enables the binding of the PID to a national trust framework, giving all the evidence of the identity proofing procedures,
the ``verification`` claim contain the information as sub claims regarding the identity proofing evidence during the issuing phase of the PID. 
The ``claims`` parameter contains the user attributes claims. Some of these additional claims MAY be included in the Disclosures and MAY be 
selectively disclosed and they are given in the following tables that also specify whether a claim is selectively disclosable (SD) or not (NSD).

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

Finally, the ``claims`` parameter contains the following mandatory claims:

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



SD-JWT Examples
^^^^^^^^^^^^^^^^^^^^^^

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

  eyJhbGciOiAiRVMyNTYifQ.eyJpc3MiOiAiaHR0cHM6Ly9waWRwcm92aWRlci5pdCIsI
  CJpYXQiOiAxNjgzMDAwMDAwLCAiZXhwIjogMTg4MzAwMDAwMCwgInZlcmlmaWVkX2NsY
  WltcyI6IHsidmVyaWZpY2F0aW9uIjogeyJfc2QiOiBbIk9HbTdyeVhndDVYemxldnAtS
  HUtVVRrMGEtVHhBYVBBb2JxdjFwSVdNZnciXSwgInRydXN0X2ZyYW1ld29yayI6ICJla
  WRhcyIsICJhc3N1cmFuY2VfbGV2ZWwiOiAiaGlnaCJ9LCAiY2xhaW1zIjogeyJfc2QiO
  iBbIjhKam96QmZvdk1OdlEzSGZsbVBXeTRPMTlHcHhzNjFGV0hqWmViVTU4OUUiLCAiQ
  m9NR2t0VzFyYmlrbnR3OEZ6eF9CZUw0WWJBbmRyNkFIc2RncGF0RkNpZyIsICJDRkxHe
  mVudEdOUkZuZ25MVlZRVmNvQUZpMDVyNlJKVVgtcmRiTGRFZmV3IiwgIkpVX3NUYUhDb
  mdTMzJYLTBhakhyZDEtSENMQ2twVDVZcWdjZlFtZTE2OHciLCAiVlFJLVMxbVQxS3hmc
  TJvOEo5aW83eE1NWDJNSXhhRzlNOVBlSlZxck1jQSIsICJ6VmRnaGNtQ2xNVldsVWdHc
  0dwU2tDUGtFSFo0dTlvV2oxU2xJQmxDYzFvIl19fSwgIl9zZF9hbGciOiAic2hhLTI1N
  iJ9.gsvYGCpWbnx8Dkd5ofKq-MtZplFFV49uY42Yf9S3rZe_SPTjg_AWdpm4bvSOhNbe
  P0aMzFGtftSk3-3sufXBdw~WyIyR0xDNDJzS1F2ZUNmR2ZyeU5STjl3IiwgImV2aWRlb
  mNlIiwgW3sidHlwZSI6ICJlbGVjdHJvbmljX3JlY29yZCIsICJyZWNvcmQiOiB7InR5c
  GUiOiAiZWlkYXMuaXQuY2llIiwgInNvdXJjZSI6IHsib3JnYW5pemF0aW9uX25hbWUiO
  iAiTWluaXN0ZXJvIGRlbGwnSW50ZXJubyIsICJvcmdhbml6YXRpb25faWQiOiAibV9pd
  CIsICJjb3VudHJ5X2NvZGUiOiAiSVQifX19XV0~WyJlbHVWNU9nM2dTTklJOEVZbnN4Q
  V9BIiwgInVuaXF1ZV9pZCIsICJ4eHh4eHh4eC14eHh4LXh4eHgteHh4eC14eHh4eHh4e
  Hh4eHgiXQ~WyI2SWo3dE0tYTVpVlBHYm9TNXRtdlZBIiwgImdpdmVuX25hbWUiLCAiTW
  FyaW8iXQ~WyJlSThaV205UW5LUHBOUGVOZW5IZGhRIiwgImZhbWlseV9uYW1lIiwgIlJ
  vc3NpIl0~WyJRZ19PNjR6cUF4ZTQxMmExMDhpcm9BIiwgImJpcnRoZGF0ZSIsICIxOTg
  wLTAxLTEwIl0~WyJBSngtMDk1VlBycFR0TjRRTU9xUk9BIiwgInBsYWNlX29mX2JpcnR
  oIiwgeyJjb3VudHJ5IjogIklUIiwgImxvY2FsaXR5IjogIlJvbWUifV0~WyJQYzMzSk0
  yTGNoY1VfbEhnZ3ZfdWZRIiwgInRheF9pZF9jb2RlIiwgIlRJTklULVhYWFhYWFhYWFh
  YWFhYWFgiXQ



MDOC-CBOR
---------

The PID MDOC-CBOR data model is defined in 
ISO/IEC 18013-5, the standard born for the the mobile driving license (mDL) use case. 

The MDOC data elements for the EUDI Wallet PID 
are those defined in the `EIDAS-ARF`_, these must be encoded as defined in `RFC 8949 - Concise Binary Object Representation (CBOR) <RFC 8949 - Concise Binary Object Representation (CBOR)>`_.

The PID MDOC-CBOR document type uses the namespace `eu.europa.ec.eudiw.pid.1`, 
according to the reverse domain approach defined in the 
`EIDAS-ARF`_ and in fully harmonization with the ISO/IEC 18013-5 standard.

The the data elements contained in the document, use the same namespace 
for the required PID attributes, while the national PID attributes use the domestic
namespace `eu.europa.ec.eudiw.pid.it.1`, defined in this implementation profile.

.. note::

    This specification publishes the Italian PID MDOC-CBOR namespace, 
    including all  data element identifiers, their definition 
    and encoding format. 


MDOC-CBOR Data Schema
^^^^^^^^^^^^^^^^^^^^^^^^

A PID issued in the MDOC-CBOR format must have the following data element:

.. list-table:: 
    :widths: 20 20 60
    :header-rows: 1

    * - **Element**
      - **Encoding**
      - **Description**
    * - **docType**
      - tstr (text string)
      - The document type. 
    * - **issuerSigned**
      - bstr (byte string)
      - The issuerSigned object containing the `nameSpaces` and the 
        `issuerAuth` objects.

.. note::
    The `issuerSigned` contains the `nameSpaces` object and the 
    `Mobile Security Object`, however sole signed object is
    the `Mobile Security Object`, while the `nameSpaces` object is not
    signed.


A non-normative example of the PID document is represented below.

.. code-block::

  {
    "docType": "eu.europa.ec.eudiw.pid.1",
    "issuerSigned": {
        "nameSpaces": {
            "eu.europa.ec.eudiw.pid.1": [
                {
                    "digestID": 0,
                    "random": $binary-random-value,
                    "elementIdentifier": "family_name",
                    "elementValue": "Rossi"
                },
                {
                    "digestID": 1,
                    "random": $binary-random-value,
                    "elementIdentifier": "given_name",
                    "elementValue": "Mario"
                },
                {
                    "digestID": 2,
                    "random": $binary-random-value,
                    "elementIdentifier": "birth_date",
                    "elementValue": {
                        "CBORTag:1004": "2007-10-20"
                    }
                },
                {
                    "digestID": 3,
                    "random": $binary-random-value,
                    "elementIdentifier": "birth_place",
                    "elementValue": "Rome"
                },
                {
                    "digestID": 4,
                    "random": $binary-random-value,
                    "elementIdentifier": "birth_country",
                    "elementValue": "IT"
                }
            ],
            "eu.europa.ec.eudiw.pid.it.1": [
                {
                    "digestID": 0,
                    "random": $binary-random-value,
                    "elementIdentifier": "tax_id_code",
                    "elementValue": "TINIT-XXXXXXXXXXXXXXXX"
                },
            ],
            ]
        },
        "issuerAuth": $MobileSecurityObject,
    },
  }

nameSpaces
^^^^^^^^^^^^^

The `nameSpaces` object contains the key value representing the namespaces 
where each value is an array of `IssuerSignedItems` encoded in CBORTag(24), representing the disclosure information for each digest within the `Mobile Security Object`.

Each digest disclosure object contains the following parameters:

.. list-table:: 
    :widths: 20 20 60
    :header-rows: 1

    * - **Name**
      - **Encoding**
      - **Description**
    * - **digestID**
      - integer
      - Digest unique identifier within its namespace.
    * - **random**
      - bstr (byte string)
      - Random byte value used as salt for the hash function. This value shall be different for each IssuerSignedItem and shall have a minimum length of 16 bytes.
    * - **elementIdentifier**
      - tstr (text string)
      - User attribute name.
    * - **elementValue**
      - depends by the value, see the next table.
      - User attribute value.

The supported `elementValue` in this specification are the followings:

.. list-table:: 
    :widths: 20 60
    :header-rows: 1

    * - **Element**
      - **Encoding**
    * - **given_name**
      - tstr (text string)
    * - **family_name**
      - tstr (text string)
    * - **birth_date**
      - full-date (CBORTag 1004)
    * - **birth_place**
      - tstr (text string)
    * - **birth_country**
      - tstr (text string)
    * - **unique_id**
      - tstr (text string)
    * - **tax_id_code**
      - tstr (text string)

Mobile Security Object
^^^^^^^^^^^^^^^^^^^^^^^^

The `Mobile Security Object` is an untagged value containing a `COSE Sign1 Document`, as defined in `RFC 9052 - CBOR Object Signing and Encryption (COSE): Structures and Process <https://www.rfc-editor.org/rfc/rfc9052.html>`_. It is composed by:

* protected header
* unprotected header
* payload
* signature.

The protected header must contain the following parameter encoded in CBOR format:

.. list-table:: 
    :widths: 20 20 60
    :header-rows: 1

    * - **Element**
      - **Description**
      - **Reference**
    * - **Signature algorithm**
      - `-7` means ES256, SHA-256. 
      - RFC8152

.. note::
    Only the Signature Algorithm must be present in the protected headers, other elements should not be present in the protected header.


The unprotected header must contain the following parameter:

.. list-table:: 
    :widths: 20 20 60
    :header-rows: 1

    * - **Element**
      - **Description**
      - **Reference**
    * - **x5chain**
      - Identified with the label 33
      - `RFC 9360 CBOR Object Signing and Encryption (COSE) - Header Parameters for Carrying and Referencing X.509 Certificates <RFC 9360 CBOR Object Signing and Encryption (COSE) - Header Parameters for Carrying and Referencing X.509 Certificates>`_.

.. note::
    The `x5chain` is included in the unprotected header with the aim to make the Holder able to update the X.509 certificate chain, related to the `Mobile Security Object` issuer, without breaking the signature.

The payload must contain the following parameter:

.. list-table:: 
    :widths: 20 20 60
    :header-rows: 1

    * - **Element**
      - **Encoding**
      - **Description**
    * - **MobileSecurityObjectBytes**
      - CBORTag(24) 
      - object, without any `content-type` definition, containing the following attributes:

        * version
        * digestAlgorithm
        * valueDigests
        * deviceKeyInfo

Therein a non-normative example of a decoded `MobileSecurityObjectBytes`, represented below in JSON format:

.. dcode-block::
    
    {
     'version': '1.0',
     'digestAlgorithm': 'SHA-256',
     'valueDigests': {
        'eu.europa.ec.eudiw.pid.1': {
            0: $digestbinaryvalue,
            1: $digestbinaryvalue,
            2: $digestbinaryvalue,
            3: $digestbinaryvalue,
            4: $digestbinaryvalue,
        },
        'eu.europa.ec.eudiw.pid.it.1': {
            0: $digestbinaryvalue
        }
       },
     'deviceKeyInfo': {
        'deviceKey': {
            1: 2,
            -1: 1,
            -2: b'\x961=lc\xe2N3rt+\xfd\xb1\xa3;\xa2\xc8\x97\xdc\xd6\x8a\xb8\xc7S\xe4\xfb\xd4\x8d\xcak\x7f\x9a',
            -3: b'\x1f\xb3&\x9e\xddA\x88W\xde\x1b9\xa4\xe4\xa4K\x92\xfaHL\xaar,"\x82\x88\xf0\x1d\x0c\x03\xa2\xc3\xd6'}
        },
        'docType': 'eu.europa.ec.eudiw.pid.1',
        'validityInfo': {
            'signed': Tag(0, '2020-10-01T13:30:02Z'),
            'validFrom': Tag(0, '2020-10-01T13:30:02Z'),
            'validUntil': Tag(0, '2021-10-01T13:30:02Z')
        }
    }

The `MobileSecurityObjectBytes` attributes are described below:

.. list-table:: 
    :widths: 20 20 60
    :header-rows: 1

    * - **Element**
      - **Encoding**
      - **Description**
    * - **version**
      - tstr
      - 
    * - **digestAlgorithm**
      - tstr
      - According to the algorithm defined in the protected header.
    * - **valueDigests**
      - bstr
      - Mapped digest by unique id, grouped by namespace.
    * - **deviceKeyInfo**
      - Information related to the Wallet Instance's cryptographic keys.
      - Containing:
        
        * **deviceKey**, public key (Holder Key Binding) of the Wallet Instace where this MSO was issued for.
        * **docType**, public key pair used for authentication. The ‘deviceKey’ element is encoded as an untagged COSE_Key element as specified in RFC 8152.
        * **validityInfo**, object containing issuance and expiration datetimes.

.. note::
    The private key related to the public key stored in the `deviceKey` object, is used to authenticate the PID during the presentation phase, where the `DeviceSignedItems` structure is involved (see the presentation phase with MDOC-CBOR).


MDOC-CBOR Examples
^^^^^^^^^^^^^^^^^^^

An MDOC-CBOR is represented below in AF Binary format:

.. code-block:: text
    
    a36776657273696f6e63312e3069646f63756d656e747381a367646f6354797065781865752e6575726f70612e65632e65756469772e7069642e316c6973737565725369676e6564a26a6e616d65537061636573a2781865752e6575726f70612e65632e65756469772e7069642e3185d818a100a4686469676573744944006672616e646f6d5820d54af67810e436b912842e188eed143c86dcc3401406361fd741975a5216d9ce71656c656d656e744964656e7469666965726a676976656e5f6e616d656c656c656d656e7456616c7565684d61736365747469d818a101a4686469676573744944016672616e646f6d5820f6f645066abe33ee80a33b19991642a948e05e1a999a0405d72b38371a8e8c8b71656c656d656e744964656e7469666965726b62697274685f706c6163656c656c656d656e7456616c756564526f6d65d818a102a4686469676573744944026672616e646f6d582074472a7395521a9abebf2f54055838cad6680ffc1362ad617e620e9edee2d34b71656c656d656e744964656e7469666965726b66616d696c795f6e616d656c656c656d656e7456616c7565695261666661656c6c6fd818a103a4686469676573744944036672616e646f6d582019162e5db6ca631e64a8c6f2544b8f631584e5e5524911601dc01b59615bc32271656c656d656e744964656e7469666965726a62697274685f646174656c656c656d656e7456616c7565d903ec6a313932322d30332d3133d818a104a4686469676573744944046672616e646f6d5820cdf438ae3d911f56d1001c180c04cd2510eeef70723126b798453efcd58e3bb871656c656d656e744964656e7469666965726d62697274685f636f756e7472796c656c656d656e7456616c7565624954781b65752e6575726f70612e65632e65756469772e7069642e69742e3181d818a105a4686469676573744944056672616e646f6d58207deab1156908843cf021477ee64be6ef89855e177fef6c1a34e44b38dddb102971656c656d656e744964656e7469666965726b7461785f69645f636f64656c656c656d656e7456616c75657554494e49542d5858585858585858585858585858586a69737375657241757468590664d28459021fa30126044864656d6f2d6b6964182159020d30820209308201afa0030201020214495201a8cf727f8e088b05f3a4aba52420daf1e7300a06082a8648ce3d0403023064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d301e170d3233303730353232313231355a170d3233303731353232313231355a3064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d3059301306072a8648ce3d020106082a8648ce3d03010703420004e73c9949473967e4628276af45264132271cdb7fd124711610e8f1ed3657910e08f099237fef088ff7dad20b921d22687e18349d4f86a9d4354d05742400a07fa33f303d303b0603551d1104343032863068747470733a2f2f63726564656e7469616c2d6973737565722e6f6964632d66656465726174696f6e2e6f6e6c696e65300a06082a8648ce3d04030203480030450221008943f01bae512ce2b09a20cd7cf021c35a37a57e469f2e71164f55a7093390990220717a23d8606da5d6593af78571924febced8d3fc01a71010afd363338e88f7afa1182159020d30820209308201afa00302010202143581691b51fc4ad4d83bfff13db6cfa5eced9533300a06082a8648ce3d0403023064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d301e170d3233303730353232313231355a170d3233303731353232313231355a3064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d3059301306072a8648ce3d020106082a8648ce3d03010703420004e73c9949473967e4628276af45264132271cdb7fd124711610e8f1ed3657910e08f099237fef088ff7dad20b921d22687e18349d4f86a9d4354d05742400a07fa33f303d303b0603551d1104343032863068747470733a2f2f63726564656e7469616c2d6973737565722e6f6964632d66656465726174696f6e2e6f6e6c696e65300a06082a8648ce3d0403020348003045022014792657a1277a8c5f95674655f16a50de27284652b812945ef2f39e6f295da8022100f0d088aa1ffa4620096b361a84b9cf70232d7c18362b2e5822997456a8a458355901e8a66776657273696f6e63312e306f646967657374416c676f726974686d667368613235366c76616c756544696765737473a2781865752e6575726f70612e65632e65756469772e7069642e31a5005820ee1511fc134fd8e77b14f5072d9c4c37a713406a326ae4725a86d355cf5f9720015820ea72665ccb72a2626d6c1a2d5ca674deec139373b683337b3b416c33dfc20f2602582004f76b3e64394ff208d769b6ca4e075ecbc72c17b9961a3c1254fd33a33356620358207ee833a6e574c4f586486f5a3fc86e5256d0682bc62353cf39ebc0dd3598fcc7045820bd9009db48ea5511cafda7f8a033aaef02028acbad0760377c3a8128cc361154781b65752e6575726f70612e65632e65756469772e7069642e69742e31a1055820b4df6e90658d355a99819ab230dfe918927c0165db01124741df32fa0526e0836d6465766963654b6579496e666fa1696465766963654b6579f667646f6354797065781865752e6575726f70612e65632e65756469772e7069642e316c76616c6964697479496e666fa3667369676e656456c074323032332d30372d30355432323a31353a30315a6976616c696446726f6d56c074323032332d30372d30355432323a31353a30315a6a76616c6964556e74696c56c074323032382d30372d30335432323a31353a30315a5840813ae368cbcb186be3e7b29aa9dd261b17f65aece799007408381221f19d16dddd795b3dc2b703bf0d96fa6e8266e4953509fdbcf38984096ca6ec8dcb02071c6c6465766963655369676e6564a06673746174757300


An MDOC-CBOR is represented below in `Diagnostic Notation`:

.. code-block:: text
    
    {
      "version": "1.0",
      "documents": [
        {
            "docType": "eu.europa.ec.eudiw.pid.1",
            "issuerSigned": {
                "nameSpaces": {
                    "eu.europa.ec.eudiw.pid.1": [
                        24_0({
                            0: {
                                "digestID": 0,
                                "random": h'876418ea921bf6c61f4934b168f5a05785a7232d12043189a0e7ed44b1a3d29a',
                                "elementIdentifier": "birth_country",
                                "elementValue": "IT",
                            },
                        }),
                        24_0({
                            1: {
                                "digestID": 1,
                                "random": h'187e7a359f09d99a7f3afe5b70fe7a15bdeb5379b17ae5d48fdc1762c8e564e5',
                                "elementIdentifier": "birth_place",
                                "elementValue": "Rome",
                            },
                        }),
                        24_0({
                            2: {
                                "digestID": 2,
                                "random": h'e3381db62cec1843c053f6f181c2e6ee3a9dddd79b3ee4e875cb29eedea40020',
                                "elementIdentifier": "birth_date",
                                "elementValue": 1004_1("1922-03-13"),
                            },
                        }),
                        24_0({
                            3: {
                                "digestID": 3,
                                "random": h'1bcf61a99cafba5da64bbda5fc2f3b0410070ae4c4ceb2c7ca2bab8f2fa9f095',
                                "elementIdentifier": "given_name",
                                "elementValue": "Mascetti",
                            },
                        }),
                        24_0({
                            4: {
                                "digestID": 4,
                                "random": h'89882a903d5c39e08686b0f8a0b9fca1ba96ed1aecd17c8844ea55ea4ba39b9a',
                                "elementIdentifier": "family_name",
                                "elementValue": "Raffaello",
                            },
                        }),
                    ],
                    "eu.europa.ec.eudiw.pid.it.1": [
                        24_0({
                            5: {
                                "digestID": 5,
                                "random": h'11aa7273a2d2daa973f5951f0c34c2fbaea0119f71279090b9619865e552f5d7',
                                "elementIdentifier": "tax_id_code",
                                "elementValue": "TINIT-XXXXXXXXXXXXXXX",
                            },
                        }),
                    ],
                },
                "issuerAuth": h'd284590220a30126044864656d6f2d6b6964182159020e3082020a308201afa003020102021411fbe1f69d2d1c2aac38555fe397267bed40db1e300a06082a8648ce3d0403023064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d301e170d3233303730353232323132335a170d3233303731353232323132335a3064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d3059301306072a8648ce3d020106082a8648ce3d030107034200048736acb9e284ae54996c595402803fdcef2e99ef9d31f3ea06513dcc679dbf14e48f4dd0e2ce3295cf9c4158f9e14072da25a0b09e077822cf1f604702005e97a33f303d303b0603551d1104343032863068747470733a2f2f63726564656e7469616c2d6973737565722e6f6964632d66656465726174696f6e2e6f6e6c696e65300a06082a8648ce3d0403020349003046022100c5d671bec6868e1463dc96dcaf491ee724a5fcbfc4aff2ddc6aec3ac8ec2c695022100e3e957c6d5b0d142f6ffe3829e30becb210c9ae9f20d72f6685a58ed9c1a294ba1182159020d30820209308201afa00302010202140d32e28fc513cf151f2a6d4823cfa46e16c930e1300a06082a8648ce3d0403023064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d301e170d3233303730353232323132335a170d3233303731353232323132335a3064310b30090603550406130255533113301106035504080c0a43616c69666f726e69613116301406035504070c0d53616e204672616e636973636f31133011060355040a0c0a4d7920436f6d70616e793113301106035504030c0a6d79736974652e636f6d3059301306072a8648ce3d020106082a8648ce3d030107034200048736acb9e284ae54996c595402803fdcef2e99ef9d31f3ea06513dcc679dbf14e48f4dd0e2ce3295cf9c4158f9e14072da25a0b09e077822cf1f604702005e97a33f303d303b0603551d1104343032863068747470733a2f2f63726564656e7469616c2d6973737565722e6f6964632d66656465726174696f6e2e6f6e6c696e65300a06082a8648ce3d0403020348003045022042cbba619cea7957733abe8cc3f4d1808810f078bef44da42d9d6d8ce525039d022100bea9d284f595b057ba6653ca227a2bf4456ecf1387651c2c8111de82d593020b5901e8a66776657273696f6e63312e306f646967657374416c676f726974686d667368613235366c76616c756544696765737473a2781865752e6575726f70612e65632e65756469772e7069642e31a500582062f85d809532321b3456e80c9f7262d0d8e5eff10afcaf5e1b7660e5a4bcd9d6015820ec21324ccac9a11fa17c16de509a3bbbd2b3b6151d0a66d40c79a2133a1d836a025820cb68596fbb3b6c58fb7dc1e1355a91dec8da16557820ce52547aeb0cc3b8182603582033e06223fee9dce67d7254de77efc6d511ecdc71de93576e12fde5b640b0d2f40458209bc489583f528d615cf9bb9e3a6d8985ebc2dd841e9363be9909feb876823401781b65752e6575726f70612e65632e65756469772e7069642e69742e31a1055820d9cdd9efbf5583ceced590bdaad6f9540c0e0f257e37d68cdadf7ee4b9212df26d6465766963654b6579496e666fa1696465766963654b6579f667646f6354797065781865752e6575726f70612e65632e65756469772e7069642e316c76616c6964697479496e666fa3667369676e656456c074323032332d30372d30355432323a32313a32335a6976616c696446726f6d56c074323032332d30372d30355432323a32313a32335a6a76616c6964556e74696c56c074323032382d30372d30335432323a32313a32335a584008b6dab030657451dbb2167d561fb87a8e1db7cb2896a5a2a89eca4798a69d74e373a1bb7b231346c090a96eee670bf528a7e699503bd0d194ff00cfab85a706',
            },
        },
    ],
    "status": 0,
  }
