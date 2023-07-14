.. include:: ../common/common_definitions.rst

=============================================
Italian EUDI Wallet Technical Specifications
=============================================

Introduction
------------

**What is eIDAS**

eIDAS stands for "electronic IDentification, Authentication and trust Services" and is the commonly used name for the EU Regulation, 910/2014, on electronic identification and trust services for electronic transactions in the internal market, repealing the old signature Directive 1999/93/EC.

The new eIDAS project establishes the European Digital Identity Wallet (EUDI Wallet) and proposes to overcome the dissimilarities, both in technological and user experience terms, that exist among the 21 digital identities (eIDs) currently active within as many as 16 Member States.

**What is IT Wallet**

The IT Wallet project was created to improve the national digital identity governance experience and respond to the input received from the European community in the areas of innovation, decentralization and digital awareness. 

To date, three identity systems coexist in Italy for access to public and private web services, namely: 

 - Electronic Identity Card (CIE);
 - Public Digital Identity System (SPID);
 - Health Card - National Service Card (TS-CNS). 

The result is a difficult, and thus costly, experience for citizens and service providers who must implement and maintain multiple authentication systems to ensure citizens' access to their digital services.

Therefore, the IT Wallet proposes to:

 - Streamline the digital identity ecosystem in Italy, optimizing the allocation of public resources;
 - Simplify the digital access experience for citizens, the Public Administration and businesses;
 - Integrate new functions related to certified attributes, strengthening the digital identity model and promoting the inclusion of public and private entities;
 - Consolidate best practices in digital identities in Italy by maximizing deployment, quality of use and infrastructure provision;
 - Adequately accommodate the European Digital Identity Wallet on the basis of the experience acquired by developing the national Wallet in full compliance with the European model.

In order to achieve these objectives and enhance the already active and eIDAS-notified digital identity schemes, the IT Wallet project proposes a technological and governance evolution that envisages, in a progressive way, the migration of the digital identification component of CIE and SPID to IT Wallet.

**Purpose of these technical rules**

The purpose of the following technical rules is to define the technical architecture and reference framework to be used as a guideline by all the parties involved in the development of the IT Wallet project.

In this documentation you can find the technical specification 
for implementing the following components:

 - Entities of the ecosystem according to `EIDAS-ARF`_.
 - Infrastructure of trust attesting realiability and eligibility of the participants.
 - PID and EAAs data schemes and attribute sets.
 - PID/EAA in MDL CBOR format.
 - PID/EAA in `SD-JWT`_ format.
 - Wallet Solution general architecture.
 - Wallet Instance Attestation data model in `JWS`_ format.
 - Issuance of PID/EAA according to `OpenID4VCI`_.
 - Presentation of PID/EAA according to `OpenID4VP`_.
 - Presentation of pseudonyms according to `SIOPv2`_.
 - PID/EAA backup and restore mechanisms.
 - PID/EAA revocation lists.


Index of content
----------------

.. toctree:: 
   :maxdepth: 2
   
   ssi-introduction.rst
   defined-terms.rst
   trust.rst
   pid-eaa-data-model.rst
   pid-eaa-issuance.rst
   wallet-solution.rst
   wallet-instance-attestation.rst
   relying-party-solution.rst
   pseudonyms.rst
   backup-restore.rst
   revocation-lists.rst
   algorithms.rst
   contribute.rst
   standards.rst

