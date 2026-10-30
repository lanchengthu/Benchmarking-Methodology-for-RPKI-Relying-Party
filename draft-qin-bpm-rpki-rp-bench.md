---
title: Benchmarking Methodology for RPKI Relying Party
abbrev: BM for RPKI RP
docname: draft-qin-bpm-rpki-rp-bench-00
obsoletes:
updates:
date:
category: info
submissionType: IETF

ipr: trust200902
area: Operations and Management
workgroup: BPM
keyword: RPKI, Relying Party, Benchmarking

author:
 -
  ins: L. Qin
  name: Lancheng Qin
  organization: Zhongguancun Laboratory
  email: qinlc@mail.zgclab.edu.cn
  city: Beijing
  country: China
 -
  ins: Y. Su
  name: Yingying Su
  organization: Zhongguancun Laboratory
  email: suyy@mail.zgclab.edu.cn
  city: Beijing
  country: China
 -
  ins: C. Zhang
  name: Chenlu Zhang
  organization: Zhongguancun Laboratory
  email: zhangcl@mail.zgclab.edu.cn
  city: Beijing
  country: China
 -
  ins: D. Li
  name: Dan Li
  organization: Tsinghua University
  email: tolidan@tsinghua.edu.cn
  city: Beijing
  country: China

  
normative:

informative:
  RFC6480:
  RFC9286:
  RFC6487:
  RFC6488:
  RFC8182:
  RFC8360:
  RFC8897:
  RFC9582:
  RFC5280:
  RFC7935:
  RFC9589:
  I-D.ietf-sidrops-rpki-erik-protocol:
  I-D.ietf-sidrops-rpki-ccr:
  RSYNC: 
    title: The rsync web pages
    target: https://rsync.samba.org
  BaRRy: 
    title: BaRRy
    target: https://github.com/lacnic/barry
  Rapport:
    title: Rapport
    target: https://github.com/lacnic/rapport
  FORT:
    title: FORT-validator
    target: https://nicmx.github.io/FORT-validator
  Routinator:
    title: Routinator
    target: https://github.com/NLnetLabs/routinator
  rpki-client:
    title: rpki-client
    target: https://www.rpki-client.org
  rpki-prover:
    title: rpki-prover
    target: https://github.com/lolepezy/rpki-prover

...

--- abstract

This document defines a benchmarking methodology for evaluating RPKI Relying Party (RP) implementations in controlled laboratory environments. The methodology focuses on whether RP implementations correctly perform required validation steps and on the performance of these operations. RP implementations are treated as black boxes, enabling consistent and objective assessment based on externally observable behavior rather than internal design or implementation details.

--- middle

# Introduction {#sec-intro}

The Resource Public Key Infrastructure (RPKI) [RFC6480] provides a framework for cryptographically securing Internet routing by allowing Relying Parties (RPs) to validate Route Origin Authorizations (ROAs) and other RPKI objects. Currently, there is no standardized methodology to evaluate whether RP implementations correctly perform the required validation procedures. In addition, the processing performance of RPs, such as the time required to validate objects and generate validated ROA payloads (VRPs), has not been systematically measured.

This document defines a benchmarking methodology for Relying Parties that addresses both functional correctness and processing performance. Specifically, the methodology provides:

1. Functional correctness tests to evaluate compliance with the RFC requirements.
2. Performance tests to measure the processing time and resource consumption from object retrieval to VRP generation.

This document currently focuses on the validation output produced from ROAs, namely the generated VRP set. Validation outputs associated with ASPA objects or other current or future RPKI signed objects are outside the scope of this document, although they may be evaluated using a similar methodology with object-specific validation requirements and expected outputs.

The remainder of this document is structured as follows:

- Section 4 defines the functional correctness and performance tests.
- Section 5 defines the format for reporting test results.

Passing all the correctness tests does not mean conforming to all the RPKI RFC.
Performance tests do not have criteria or requirements.


## Requirements Language

{::boilerplate bcp14-tagged}


# Relying Party Overview {#sec-rp}

A Relying Party (RP) in the Resource Public Key Infrastructure (RPKI) is responsible for retrieving, validating, and making available RPKI objects to support secure route validation. [RFC8897] specifies the expected behavior of RPs, which includes:

- Object Retrieval: obtaining RPKI objects from Publication Points (PPs) and keeping them up-to-date.

- Object Syntax Validation: checking DER encoding, syntax, and structural correctness of RPKI objects, including certificates, CRLs, ROAs, and manifests.

- Certification Path Validation: constructing and validating certificate chains from the Trust Anchor to each leaf certificate.

- CMS Signed Object Signature Validation: verifying digital signatures of the RPKI CMS objects.

- Manifest Processing: ensuring completeness and integrity of published objects.


After validation, the RP produces a Validated Payload for use in routing systems. These functions ensure that only valid and trusted RPKI objects influence routing decisions.

# Test Setup {#sec-test-methodology}

This section defines the test setup. The System Under Test (SUT) (i.e., the RP software) is treated as a black box. No internal configuration or implementation behavior of the RP software is mandated. The test setup focuses on providing controlled inputs and observing RP outputs to enable reproducible and comparable measurements.

In this methodology, the input to the SUT is the controlled RPKI repositories configured by the Controller and published through the controlled Servers. The output of the SUT is the generated VRP set, or equivalent validation output that allows the Controller to determine the resulting VRPs. Verification in this methodology is based on the SUT's externally observable output, rather than on its internal cache layout, internal processing pipeline, or implementation-specific debug information. For the correctness metrics defined in this document, a one-shot validation run is sufficient. 

## Test Environment

In this methodology, the Tester consists of the Servers and the Controller. Together, they generate the test conditions, trigger events, and support observation of the SUT for functional and performance evaluation. The SUT itself is evaluated for correctness and efficiency in processing RPKI objects.

~~~
+----------------------------------+
|             Controller           |
+----------------------------------+
       |                  |
       v                  v
+-------------+      +-------------+
|     SUT     | ---> |   Servers   |
+-------------+      +-------------+
~~~
{: #fig-1 title="Test environment"}

To ensure meaningful testing, the environment should include:

- At least one Trust Anchor (TA)
- One or more subordinate certificate authority (CA) certificates
- RPKI objects signed by CAs

The test environment should support multiple RPKI transport protocols for object retrieval, including Rsync [RSYNC], RRDP [RFC8182], or Erik [I-D.ietf-sidrops-rpki-erik-protocol]. 

## System Under Test and Servers

The SUT is an RPKI Relying Party implementation that retrieves RPKI objects from the controlled repositories, validates them, and generates the VRP set or equivalent validation output.

The Servers should include PPs which host RPKI objects signed by CAs. Each PP at least contains ROAs, a manifest, and a CRL necessary for the tests. RPKI objects can be added, modified, or removed by the Controller.

## Controller

The Controller orchestrates the entire testing process. In {{fig-1}}, the arrows from the Controller to both the SUT and Servers represent the flow of control information. 

For the SUT, the Controller controls the start and end of tests. The Controller can obtain the SUT's VRP set or equivalent validation output through an externally observable output interface configured for the test, such as an output file, RTR, or a CCR profile [I-D.ietf-sidrops-rpki-ccr]. The Controller then processes test results and compares them with the expected output derived from the controlled repository contents.

For the Servers, the Controller handles content changes to ensure that the SUT sees one set of RPKI objects during a validation run and a different set during the next run. Existing tools (e.g., [BaRRy], [Rapport]) can be used to implement and manage the test environment.

# RPKI Relying Party Benchmarking Tests

**General assumptions:**
The functional correctness and performance tests in this section assume that the controlled servers are correctly configured and reachable by the SUT.
Object retrieval is assumed to be complete successfully.

The tests in this section are based on the published RPKI specifications referenced by each test. The test cases are intended to exercise representative correctness properties, rather than to provide an exhaustive conformance test suite for RP software. Validation behavior introduced by Internet-Drafts or other work-in-progress updates may be relevant to implementations, but is outside the scope of this document.

The detailed functional test points covered in this document are catalogued in {{app-test-points}}.

## Baseline Valid Repository Processing

**Objective:** To establish a baseline for evaluating whether the SUT can process a controlled repository containing valid RPKI objects and generate the expected VRP set. This test verifies the SUT's output for a repository in which all relevant objects are valid and all validation conditions are satisfied. This baseline confirms that valid repository contents produce the expected VRP set before error cases are introduced.

The baseline repository in this test should contain all objects required for validating the intended RPKI objects, including the relevant resource certificates, manifests, CRLs, and ROAs. Each ROA under test MUST produce a distinct VRP payload. This ensures that the absence of a VRP from the generated VRP set can be attributed to the rejection of the corresponding ROA.

**Example Baseline Repository:**
{{fig-2}} illustrates an example baseline repository, denoted as BR, where the indentation shows the certification hierarchy and corresponding publication points conceptually.

~~~
ta.cer
	ca1.cer
		roa1-1.roa ----> Expected VRP: VRP1-1
		roa1-2.roa ----> Expected VRP: VRP1-2
		ca1.mft
		ca1.crl
	ca2.cer
		roa2-1.roa ----> Expected VRP: VRP2-1
		roa2-2.roa ----> Expected VRP: VRP2-2
		ca2.mft
		ca2.crl
	ta.mft
	ta.crl
~~~
{: #fig-2 title="An example baseline repository"}

In BR, ta.cer is the Trust Anchor certificate. It certifies ca1.cer and ca2.cer.
CA1 issues two ROAs, roa1-1.roa and roa1-2.roa. CA2 issues two ROAs, roa2-1.roa and roa2-2.roa. Each ROA is valid and produces one distinct VRP. The expected VRP set for BR is therefore: {VRP1-1, VRP1-2, VRP2-1, VRP2-2}.

The **procedure** for this test is as follows: 

1. Configure the controlled servers to host BR.

2. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

3. Start the SUT and allow it to perform one validation run over the controlled repository BR.

4. Compare the generated VRP set with the expected VRP set for BR.

**Expected Results:**
The generated VRP set matches the expected VRP set derived from the baseline repository contents, i.e., {VRP1-1, VRP1-2, VRP2-1, VRP2-2}. This indicates that, for the baseline repository, the SUT can retrieve, parse, validate, and process the relevant RPKI objects sufficiently to generate the expected VRPs.

Each functional correctness test in Section 4.2 through 4.6 starts from the baseline repository BR defined in this section. One target object is modified to violate a specific requirement.
The test setup MUST introduce only the violation under test.
Any dependent objects MUST be updated as needed to avoid unrelated violations, unless leaving them unchanged is necessary for the test. 
The SUT behavior is evaluated by checking whether the generated VRP set matches the expected result derived from the single-object violation. In general, VRPs whose production depends on the malformed object are expected to be absent from the generated VRP set, while unaffected baseline VRPs are expected to remain present.


## Object Syntax Validation Correctness

This section evaluates whether the SUT correctly performs syntax validation for different RPKI objects. The SUT is expected to perform syntax checks according to the relevant specifications and detect objects that do not conform to the defined syntax requirements.

The syntax requirements for different RPKI objects are defined in the following specifications:

- Certificate and CRL: [RFC5280], [RFC6487] and [RFC7935]
- Manifest: [RFC6488], [RFC9589] and [RFC9286]
- ROA: [RFC6488], [RFC9589] and [RFC9582]

### DER Decoding

**Objective:** To evaluate whether the SUT correctly rejects an RPKI object with invalid Distinguished Encoding Rules (DER) encoding. Examples include, but are not limited to, a malformed ASN.1 tag field, an inconsistent length field, and a BER-specific encoding form that is not permitted by DER. 

The **procedure** for this test is as follows: 

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace roa1-1.roa with a malformed version that cannot be decoded as valid DER, e.g., a truncated version produced by removing the last few octets of the file. Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ca1.mft is updated so that it lists the correct hash of the malformed version of roa1-1.roa, and the updated ca1.mft remains valid. 

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set. 

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRP derived from the malformed ROA removed, i.e., {VRP1-2, VRP2-1, VRP2-2}.
The absence of VRP1-1 is attributed to the DER decoding failure of the modified roa1-1.roa. VRPs derived from other unaffected ROAs remain present.

### Certificate Syntax Validation

**Objective:** To evaluate whether the SUT correctly rejects a certificate that violates the syntax requirements described in [RFC5280], [RFC6487] and [RFC7935]. Examples include, but are not limited to, an unsupported certificate version, an invalid certificate serial number, an invalid SIA or AIA extension, and an invalid IP or AS resource extension.
The complete set of test points is listed in {{app-cert-syntax}}. Each distinct test point is evaluated in a separate run. The procedure and expected results below illustrate one such run.

**Procedure:**

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.cer with a modified version in which the Basic Constraints extension is absent. Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ta.mft is updated so that it lists the correct hash of the malformed version of ca1.cer, and the updated ta.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set. 

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from ROAs under ca1.cer removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the certificate syntax validation failure of the modified ca1.cer.
VRP2-1 and VRP2-2 remain present because the ca2.cer branch is not modified and thus unaffected.

For each remaining test point in {{app-cert-syntax}}, repeat the procedure with the corresponding condition substituted in Step 2 and compare the observed results with the expected results for that test point. 

### CRL Syntax Validation

**Objective:** To evaluate whether the SUT correctly rejects a CRL that violates the syntax requirements described in [RFC5280] and [RFC6487]. Examples include, but are not limited to, an unsupported CRL version, an invalid CRL Number extension, a disallowed CRL extension, and a prohibited revoked-certificate entry.
The complete set of test points is listed in {{app-crl-syntax}}. Each distinct test point is evaluated in a separate run. The procedure and expected results below illustrate one such run.

**Procedure:**

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.crl with a malformed version containing a CRL-level extension other than Authority Key Identifier and CRL Number.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent.  In particular, ca1.mft is updated so that it lists the correct hash of the malformed version of ca1.crl, and the updated ca1.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set. 

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from ROAs under the publication point using the malformed ca1.crl removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the syntax validation failure of the modified ca1.crl.
VRP2-1 and VRP2-2 remain present because the ca2.cer branch is not modified and thus unaffected.

For each remaining test point in {{app-crl-syntax}}, repeat the procedure with the corresponding condition substituted in Step 2 and compare the observed results with the expected results for that test point.

### CMS Signed Object Syntax Validation

**Objective:** To evaluate whether the SUT correctly rejects a malformed signed object that violates the syntax requirements
common to all CMS-based RPKI signed objects defined in [RFC6488] and [RFC9589]. Examples include, but are not limited to, an illegal digest algorithm, certificate missing, and multiple signer infos.
The complete set of test points is listed in {{app-cms-syntax}}. Each distinct test point is evaluated in a separate run. The procedure and expected results below illustrate one such run.

**Procedure:**

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace roa1-1.roa with a modified version whose CMS signer identifier contains a subject key identifier different from the SKI in its embedded EE certificate.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ca1.mft is updated so that it lists the correct hash of the malformed version of roa1-1.roa, and the updated ca1.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRP derived from the malformed ROA removed, i.e., {VRP1-2, VRP2-1, VRP2-2}.
The absence of VRP1-1 is attributed to the syntax validation failure of the modified roa1-1.roa. VRPs derived from other unaffected ROAs remain present.

For each remaining test point in {{app-cms-syntax}}, repeat the procedure with the corresponding condition substituted in Step 2 and compare the observed results with the expected results for that test point.


### Manifest-Specific Syntax Validation

**Objective:** To evaluate whether the SUT correctly rejects a malformed manifest that violates the manifest-specific syntax requirements defined in [RFC9286]. Examples include, but are not limited to, an invalid manifest version, an invalid validity interval, and an unsupported hash algorithm OID. The complete set of test points is listed in {{app-mft-syntax}}. Each distinct test point is evaluated in a separate run. The procedure and expected results below illustrate one such run.

**Procedure:**

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.mft with a modified version in which nextUpdate is earlier than thisUpdate. Keep all other objects unchanged.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from ROAs under the publication point using the malformed ca1.mft removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the syntax validation failure of the modified ca1.mft. 
VRP2-1 and VRP2-2 remain present because the ca2.cer branch is not modified and thus unaffected.

For each remaining test point in {{app-mft-syntax}}, repeat the procedure with the corresponding condition substituted in Step 2 and compare the observed results with the expected results for that test point. 

### ROA-Specific Syntax Validation

**Objective:** To evaluate whether the SUT correctly rejects a ROA that violates the ROA-specific syntax requirements defined in [RFC9582]. 
Examples include, but are not limited to, an incorrect content type OID, an unsupported ROA version, an out-of-range origin AS number, and an unsupported IP address family.
The complete set of test points is listed in {{app-roa-syntax}}. Each distinct test point is evaluated in a separate run. The procedure and expected results below illustrate one such run.

**Procedure:**

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace roa1-1.roa with a malformed version containing an Address Family Identifier other than IPv4 or IPv6.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ca1.mft is updated so that it lists the correct hash of the malformed version of roa1-1.roa, and the updated ca1.mft remains valid. 

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRP derived from the malformed ROA removed, i.e., {VRP1-2, VRP2-1, VRP2-2}.
The absence of VRP1-1 is attributed to the syntax validation failure of the modified roa1-1.roa. VRPs derived from other unaffected ROAs remain present.

For each remaining test point in {{app-roa-syntax}}, repeat the procedure with the corresponding condition substituted in Step 2 and compare the observed results with the expected results for that test point. 

## Certification Path Validation Correctness

This section evaluates whether the SUT correctly handles failures during RPKI certificate path validation. The SUT is expected to perform the certificate path validation process for all RPKI resource certificates according to the requirements specified in Section 7.2 of [RFC6487]. This includes, for example, checking certificate signatures, issuer-subject relationships, certificate validity periods, and certificate revocation status. 

The scenarios in this section are representative and are not intended to enumerate all certificate path validation requirements. Each test introduces one specific certification path violation to evaluate the SUT behavior. 
A validation failure in a certificate path is expected to prevent the affected certificate and the RPKI objects depending on it from contributing payloads to the generated output.

### Invalid Certificate Signature

**Objective:** To evaluate whether the SUT correctly rejects a certificate whose signature cannot be verified using the public key of its issuer.

The **procedure** for this test is as follows:

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.cer with a modified version whose certificate signature cannot be verified using the public key of the trust anchor.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent.  In particular, ta.mft is updated so that it lists the correct hash of the modified version of ca1.cer, and the updated ta.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from ROAs under the modified ca1.cer removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the certificate signature validation failure of the modified ca1.cer.
VRP2-1 and VRP2-2 remain present because the ca2.cer branch is not modified and thus unaffected.


### Issuer-Subject Mismatch

**Objective:** To evaluate whether the SUT correctly rejects a certificate path in which the issuer information of a certificate does not match the subject information of the certificate expected to issue it.

The **procedure** for this test is as follows:

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.cer with a modified version whose issuer information does not match the subject information of the trust anchor.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent.  In particular, ta.mft is updated so that it lists the correct hash of the modified version of ca1.cer, and the updated ta.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from ROAs under the modified ca1.cer removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the issuer-subject mismatch in the certification path containing the modified ca1.cer.
VRP2-1 and VRP2-2 remain present because the ca2.cer branch is not modified and thus unaffected.


### Certificate Validity Check

**Objective:** To evaluate whether the SUT correctly rejects a certificate that is not valid at the time of validation. For example, the certificate may have a notBefore value later than the validation time or a notAfter value earlier than the validation time. 

The **procedure** for this test is as follows:

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.cer with a modified version whose validity period does not cover the validation time. Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ta.mft is updated so that it lists the correct hash of the modified version of ca1.cer, and the updated ta.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from ROAs under the modified ca1.cer removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the validity period failure of the modified ca1.cer. VRP2-1 and VRP2-2 remain present because the ca2.cer branch is not modified and thus unaffected.


### Certificate Revocation Check

This section evaluates whether the SUT correctly determines the revocation status of certificates during certification path validation and correctly propagates the revocation status to the validation results of relevant RPKI objects. The tests below cover the revocation of a CA certificate, a ROA's EE certificate, and a manifest's EE certificate.

#### Revoked CA Certificate

**Objective:** To evaluate whether the SUT correctly rejects a CA certificate that has been revoked by its issuer and consequently rejects RPKI objects that depend on the revoked CA certificate.

The **procedure** for this test is as follows:

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ta.crl with a modified CRL that lists the serial number of ca1.cer as revoked. Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ta.mft is updated so that it lists the correct hash of the modified version of ta.crl, and the updated ta.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from the CA1 publication point removed, i.e., {VRP2-1, VRP2-2}.
The absence of VRP1-1 and VRP1-2 is attributed to the revocation of ca1.cer.
Unaffected baseline VRPs from CA2 publication point remain present.


#### Revoked ROA's EE Certificate
**Objective:** To evaluate whether the SUT correctly rejects a ROA whose associated EE certificate has been revoked by its issuing CA.

The **procedure** for this test is as follows:

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.crl with a modified CRL that lists the serial number of the EE certificate contained in roa1-1.roa as revoked.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ca1.mft is updated so that it lists the correct hash of the modified version of ca1.crl, and the updated ca1.mft remains valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRP derived from roa1-1.roa removed, i.e., {VRP1-2, VRP2-1, VRP2-2}. The absence of VRP1-1 is attributed to the revoked EE certificate contained in roa1-1.roa. 
VRPs derived from other unaffected ROAs remain present.


#### Revoked Manifest's EE Certificate

**Objective:** To evaluate whether the SUT correctly treats a fetch as failed when the EE certificate for the current manifest is revoked by the current CRL. 

The **procedure** for this test is as follows: 

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace ca1.crl with a modified CRL that lists the serial number of the EE certificate contained in ca1.mft as revoked. Keep all other objects unchanged, except for the updates required to keep the repository internally consistent.  
In particular, ca1.mft is updated so that it lists the correct hash of the modified version of ca1.crl, and the updated ca1.mft remains otherwise valid.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from the CA1 publication point removed, i.e., {VRP2-1, VRP2-2}. 
Unaffected baseline VRPs from CA2 publication point remain present.
This reflects that, in a cold-start validation run, revoking the manifest's EE certificate causes the fetch for the corresponding publication point to fail.


## CMS Signed Object Signature Validation Correctness

**Objective:** To evaluate whether the SUT correctly rejects a CMS signed object whose digital signature cannot be validated using the EE certificate contained in the object as described in [RFC6488]. 
Representative test points include a message-digest attribute inconsistent with the encapsulated content and a signature value not verifiable by the public key in the associated EE certificate.
The complete set of test points is listed in {{app-cms-signature}}. Each distinct test point is evaluated in a separate run. The procedure and expected results below illustrate one such run.

**Procedure:**

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace roa1-1.roa with a modified version whose CMS signature cannot be verified using the public key in the EE certificate contained in roa1-1.roa. 
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ca1.mft is updated so that it lists the correct hash of the modified version of roa1-1.roa, and the updated ca1.mft remains valid. 

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRP derived from the modified ROA removed, i.e., {VRP1-2, VRP2-1, VRP2-2}.
The absence of VRP1-1 is attributed to the signature validation failure of the modified roa1-1.roa.
VRPs derived from other unaffected ROAs remain present.

For each remaining test point in {{app-cms-signature}}, repeat the procedure with the corresponding condition substituted in Step 2 and compare the observed results with the expected results for that test point. 


## ROA Resource Coverage Validation Correctness

**Objective:** To evaluate whether the SUT correctly rejects a ROA with an invalid resource coverage relationship. To be specific, each IP address prefix in the ROA is expected to be covered by the Verified Resource Set-IP (VRS-IP) resulting from validation of the ROA's EE certificate, according to the ROA validation rules in [RFC9582] and the resource validation model in [RFC8360].   

The **procedure** for this test is as follows:

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace roa1-1.roa with a modified version containing an IP address prefix that is not covered by the VRS-IP set of its EE certificate.
Keep all other objects unchanged, except for the updates required to keep the repository internally consistent. In particular, ca1.mft is updated so that it lists the correct hash of the malformed version of roa1-1.roa, and the updated ca1.mft remains valid. 

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRP derived from the modified ROA removed, i.e., {VRP1-2, VRP2-1, VRP2-2}. The absence of VRP1-1 is attributed to the ROA IP prefix not being covered by the VRS-IP set of the ROA's EE certificate.
VRPs derived from other unaffected ROAs remain present.


## Manifest Processing Correctness

This section evaluates whether the SUT correctly uses manifests to verify the integrity and completeness of RPKI repository objects.

Manifests are expected to be used to:

- Verify that the content of each RPKI object matches the hash listed in the manifest.
- Verify that objects accepted from a publication point are consistent with the object list declared in the corresponding manifest.

### Manifest Hash-Mismatch Check

**Objective:** To evaluate whether the SUT correctly handles a retrieved RPKI object whose content does not match the hash declared in its associated manifest.

The **procedure** for this test is as follows: 

1. Start from the baseline repository BR defined in Section 4.1.

2. Replace roa1-1.roa with a modified version whose computed hash no longer matches the hash value listed for roa1-1.roa in ca1.mft. Keep all other objects unchanged.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from the CA1 publication point removed, i.e., {VRP2-1, VRP2-2}.
Unaffected baseline VRPs from CA2 publication point remain present. 
This reflects that, in a cold-start validation run, a manifest file hash mismatch causes the fetch for the corresponding publication point to fail.


### Manifest Object-Mismatch Check

**Objective:** To evaluate whether the SUT correctly handles mismatches between the object list in a manifest and the actual objects available at the corresponding publication point. 
The following scenarios distinguish between a missing listed object scenario and an extra unlisted object scenario.

**Missing-object scenario:**

The **procedure** for this scenario is as follows: 

1. Start from the baseline repository BR defined in Section 4.1.

2. Remove roa1-1.roa from the CA1 publication point. Keep all other objects unchanged. In particular, keep ca1.mft unchanged so that it still lists roa1-1.roa and its original hash value.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set with the VRPs derived from CA1 publication point removed, i.e., {VRP2-1, VRP2-2}.
Unaffected baseline VRPs from CA2 publication point remain present. This reflects that, in a cold-start validation run, a missing listed object in the manifest causes the fetch for the corresponding publication point to fail.

**Extra-object scenario:**

The **procedure** for this scenario is as follows: 

1. Start from the baseline repository BR defined in Section 4.1.

2. Add an otherwise valid ROA object, roa1-extra.roa, to the CA1 publication point without listing it in the corresponding manifest ca1.mft, and record the VRP that would be produced from roa1-extra.roa if it were used by the SUT. Keep all other objects unchanged.

3. Configure the controlled servers to host the modified BR repository.

4. Initialize the SUT for a cold-start validation run (e.g., with no previously cached repository state).

5. Start the SUT and allow it to retrieve and validate the modified repository contents.

6. Collect the generated VRP set and compare it with the expected VRP set.

**Expected Results:**
The generated VRP set matches the baseline expected VRP set, i.e., {VRP1-1, VRP1-2, VRP2-1, VRP2-2}. No additional VRP derived from roa1-extra.roa appears in the generated VRP set.


## Performance Tests

This section evaluates the performance characteristics of the SUT during object retrieval and validation process under controlled repository conditions. For all tests in this section, object retrieval MUST adopt the same transport protocol within the same test set. This ensures that the measured results are comparable and reflect SUT behavior rather than differences introduced by transport protocols.

### Persistent Storage Space

**Objective:**
To measure the persistent storage space occupied by the SUT after completing a cold-start validation run. Persistent storage space is measured as the total additional disk space consumed by the SUT relative to its initial clean state, rather than the size of any specific cache directory or other implementation-specific storage location from each SUT.

**Procedure:**

1. Prepare a test repository containing a predefined set of RPKI objects, and record its size and composition, including the numbers of CAs, ROAs, manifests, CRLs, and generated VRPs.

2. Start the SUT for a cold-start validation run and allow it to retrieve and validate the test repository. 

3. Stop the SUT after the initial synchronization and validation are completed.

4. Measure the persistent storage space occupied by the SUT after processing the test repository.

5. Repeat the test using test repositories of different sizes to evaluate performance under varying repository scales.

### Processing Time

**Objective:** To measure the processing time of the SUT over a predefined sequence of validation runs, including an initial cold-start validation run and several subsequent validation runs. For each run, processing time is measured as the elapsed wall-clock time between the moment the Controller starts the validation run and the moment the SUT makes the generated VRP set or equivalent validation output available.

**Procedure:**

1. Prepare a predefined test repository sequence, consisting of a set of test repository states. 
For each repository state, record its size and composition, including the numbers of CAs, ROAs, manifests, CRLs, and generated VRPs.

2. Start the SUT with an empty or isolated cache for the initial cold-start validation run, and record the start timestamp.

3. Allow the SUT to complete the validation run and produce the generated VRP set or equivalent validation output.

4. Record the end timestamp when the generated VRP set or equivalent validation output becomes available.

5. Calculate the processing time for the run as the difference between the start and end timestamps.

6. For each subsequent repository state, trigger the next validation run without restarting the SUT, and record the processing time for that run using the same method.

7. Repeat the test using test repositories of different sizes to evaluate performance under varying repository scales.

### CPU Utilization

**Objective:** To measure the CPU utilization of the SUT over a predefined sequence of validation runs, including an initial cold-start validation run and several subsequent validation runs. 
CPU utilization is measured by monitoring the SUT process at a fixed sampling interval during object retrieval and validation. The test reports the average and peak CPU utilization for each run, as well as the overall average and peak CPU utilization over the complete sequence of validation runs.

**Procedure:**

1. Prepare a predefined test repository sequence, consisting of a set of test repository states. 
For each repository state, record its size and composition, including the numbers of CAs, ROAs, manifests, CRLs, and generated VRPs.

2. Start the SUT with an empty or isolated cache for the initial cold-start validation run using the first repository state.

3. Monitor the CPU utilization of the SUT process at the fixed sampling interval from the moment the Controller starts the validation run.

4. Allow the SUT to complete the validation run and produce the generated VRP set or equivalent validation output.

5. Calculate the average and peak CPU utilization for the initial cold-start validation run based on the recorded samples between the start of the run and the availability of the generated VRP set or equivalent validation output.

6. For each subsequent repository state, trigger the next validation run without clearing the SUT's cache, and measure the average and peak CPU utilization for that run using the same method.

7. Calculate the overall average CPU utilization and overall peak CPU utilization over the complete sequence of validation runs. The overall average CPU utilization is calculated over all samples collected during the measured validation-run intervals, rather than as an unweighted average of per-run averages.

8. Repeat the test using repository sequences of different sizes to evaluate CPU utilization under varying repository scales.


### Memory Usage

**Objective:** To measure the memory usage of the SUT over a predefined sequence of validation runs, including an initial cold-start validation run and several subsequent validation runs. Memory usage is measured by monitoring the SUT process at a fixed sampling interval during object retrieval and validation. The test reports the average and peak memory usage for each run, as well as the overall average and peak memory usage over the complete sequence of validation runs.

**Procedure:**

1. Prepare a predefined test repository sequence, consisting of a set of test repository states. 
For each repository state, record its size and composition, including the numbers of CAs, ROAs, manifests, CRLs, and generated VRPs.

2. Start the SUT with an empty or isolated cache for the initial cold-start validation run using the first repository state.

3. Monitor the memory usage of the SUT process at the fixed sampling interval from the moment the Controller starts the validation run.

4. Allow the SUT to complete the validation run and produce the generated VRP set or equivalent validation output.

5. Calculate the average and peak memory usage for the initial cold-start validation run based on the recorded samples between the start of the run and the availability of the generated VRP set or equivalent validation output.

6. For each subsequent repository state, trigger the next validation run without clearing the SUT's cache, and measure the average and peak memory usage for that run using the same method.

7. Calculate the overall average memory usage and overall peak memory usage over the complete sequence of validation runs. The overall average memory usage is calculated over all samples collected during the measured validation-run intervals, rather than as an unweighted average of per-run averages.

8. Repeat the test using repository sequences of different sizes to evaluate memory usage under varying repository scales.


# Report Format

This section defines the format for reporting the results of RPKI Relying Party benchmarking tests. The format is concise and suitable for documenting both functional correctness and performance results.

System Under Test (SUT) Information:

- RP implementation name and version
- Relevant SUT configuration used for the test

Test Repository Information:

- Number of objects and object types used in the baseline repository
- Number of expected VRPs in the baseline repository
- Test cases executed
- Test-specific repository modification, if any

Functional Correctness Test Results:

For each functional correctness test category, record whether each executed test point passed or failed, and report the total number of passed and failed test points in that category as below. 

- Baseline Valid Repository Processing: pass count, fail count
- DER Decoding Correctness:
     - Invalid DER Handling: pass count, fail count
- Object Syntax Validation Correctness:
     - Certificate: pass count, fail count
     - CRL: pass count, fail count
     - Manifest: pass count, fail count
     - ROA: pass count, fail count
- Certification Path Validation Correctness: 
     - Invalid Certificate Signature: pass count, fail count
     - Issuer-Subject Mismatch: pass count, fail count
     - Certificate Validity Check: pass count, fail count
     - Certificate Revocation Status Check: 
          - Revoked CA certificate: pass count, fail count
          - Revoked ROA's EE certificate: pass count, fail count
          - Revoked Manifest's EE certificate: pass count, fail count
- CMS Signed Object Signature Validation Correctness:
     - Invalid Signature Handling: pass count, fail count
- ROA Resource Coverage Validation Correctness: pass count, fail count
- Manifest Processing Correctness:
     - Hash-Mismatch Check: pass count, fail count
     - Object-Mismatch Check:
         - Missing Object: pass count, fail count
         - Extra Object: pass count, fail count

Performance Test Results:

For each performance test category, report the corresponding measurement results along with the composition of the test repository.

- Persistent Storage Space Test:
     - Repository composition
     - Total persistent storage space consumed by the SUT
- Processing Time Test:
     - Repository composition for each predefined repository state
     - Total elapsed time for each validation run     
- CPU Utilization Test:
     - Repository composition for each predefined repository state
     - Average CPU utilization for each validation run
     - Peak CPU utilization for each validation run
     - Overall average CPU utilization over the complete sequence of validation runs
     - Overall peak CPU utilization over the complete sequence of validation runs
- Memory Usage Test:
     - Repository composition for each predefined repository state
     - Average memory usage for each validation run
     - Peak memory usage for each validation run
     - Overall average memory usage over the complete sequence of validation runs
     - Overall peak memory usage over the complete sequence of validation runs


# Implementation Status

This section records known implementations and supporting test tools for RP Benchmarking tests.
Readers should note that this information is not an exhaustive catalogue, and other implementations may exist.

[BaRRy] is a configurable RPKI repository generator that can produce valid and intentionally invalid repositories, supporting certificates, CRLs, manifests, ROAs, and ASPA objects. 

[Rapport] is a script-based RP testing framework that uses [BaRRy] to generate controlled repositories and evaluate RP behavior. Its test suites cover certificate and CRL validation, manifest processing, ROA and ASPA validation, RRDP synchronization and fallback, and RPKI-Router protocol handling.

The authors have developed additional test cases and supporting scripts based on [BaRRy], covering all functional correctness tests defined in this document. The implementation has been used to evaluate [FORT], [Routinator], [rpki-client], and [rpki-prover]. The source code and test reports are available at https://github.com/chenluzh621/rpki-rp-benchmark.

# Security Considerations {#sec-security}

This document defines benchmarking methodologies for RPKI RP implementations in controlled laboratory environments using dedicated address space and constrained resources. No additional security considerations are identified within the scope of this document.

# IANA Considerations {#sec-iana}

This document has no IANA requests.

# Acknowledgements
{:numbered="false"}

The authors would like to thank Jorge Cano and the FORT team for their detailed reviews and constructive feedback, which helped clarify and improve the scope and methodology of this work. The authors also thank Mikhail Puzanov for his valuable feedback on the RP/SUT model and the externally observable validation output used in this methodology.

--- back


# Catalogue of Functional Test Points {#app-test-points}

## Certificate Syntax Validation {#app-cert-syntax}

| **Field** | **Test points** |
| --- | --- |
|           | Prohibited fields present |
| `version` | Non v3 version |
| `serialNumber` | Non-positive serial number; duplicate serial numbers under same issuer |
| `signature`/`signatureAlgorithm` | Inner and outer algorithm identifiers mismatch; non-conforming signature algorithm identifiers |
| `issuer` | Missing `commonName`; multiple `commonName` attributes; wrong string type |
| `subject` | Missing `commonName`; multiple `commonName` attributes; wrong string type; duplicate subject names under same issuer |
| `validity` | `notAfter` earlier than `notBefore` |
| `subjectPublicKeyInfo` | Incorrect public key algorithm identifier; malformed RSA public key encoding |
| `basicConstraints` | Missing in a CA certificate; present in an EE certificate; wrong criticality marking; disallowed `pathLenConstraint` field present |
| `subjectKeyIdentifier` | Missing extension; non-conforming key identifier; wrong criticality marking |
| `authorityKeyIdentifier` | Missing extension; non-conforming key identifier; disallowed `authorityCertIssuer`/`authorityCertSerialNumber` fields present; wrong criticality marking |
| `keyUsage` | Missing extension; non-conforming bits setting; wrong criticality marking |
| `extendedKeyUsage` | Incorrectly present in CA and EE certificates used to verify RPKI objects |
| `cRLDistributionPoints` | Missing extension; disallowed `reasons`/`cRLIssuer` fields present; relative distribution point name; non-URI `generalName`; multiple distribution points; wrong criticality marking |
| AIA | Missing in a subordinate CA certificate; incorrectly present in a self-signed certificate; incorrect access method value; URI referencing a different issuer certificate; wrong criticality marking |
| SIA for CA  | Missing extension; missing `caRepository` access description; non-rsync repository URI; missing `rpkiManifest` access description; wrong criticality marking |
| SIA for EE | Missing `signedObject` access description; disallowed `caRepository` access description present; wrong criticality marking |
| `certificatePolicies` | Missing extension; multiple policy entries; wrong criticality marking |
| IP resource | Missing extension; disallowed subsequent AFI present; empty IP resource set; wrong criticality marking |
| AS resource | Missing extension; disallowed RDI value present; empty AS resource set; wrong criticality marking |
{: title="Certificate Syntax Validation Test Points" #tab-cer-tests}

## CRL Syntax Validation {#app-crl-syntax}

| **Field** | **Test points** |
| --- | --- |
| `version` | Non-v2 CRL version |
| `signature`/`signatureAlgorithm` | Invalid signature algorithm OID; inner and outer signature algorithm mismatch |
| `issuer` | Missing `commonName`; multiple `commonName` attributes; wrong string type |
| `thisUpdate`/`nextUpdate` | Stale CRL; invalid time format |
| `revokedCertificates` | Missing certificate serial number in an entry; missing revocation date in an entry; disallowed CRL entry extensions present |
| `crlExtensions` | Disallowed CRL extension present |
| `crlExtensions.authorityKeyIdentifier` | Missing extension; wrong criticality marking; key identifier inconsistent with issuer's SKI; disallowed field `authorityCertIssuer`/`authorityCertSerialNumber` present |
| `crlExtensions.cRLNumber` | Missing extension; wrong criticality marking; negative value; value exceeding `2^159−1` |
{: title="CRL Syntax Validation Test Points" #tab-crl-syntax}

## CMS Signed Object Syntax Validation {#app-cms-syntax}

| **Field** | **Test points** |
| --- | --- |
| `contentType` | Content type other than `id-signedData` |
| `signerInfos` | Multiple `SignerInfo` objects |
| `version` | Version other than 3 |
| `digestAlgorithms` | Multiple digest algorithm identifiers; unsupported digest algorithm OID |
| `certificates` | Missing certificates field; multiple EE certificates |
| `crls` | `crls` field present |
| `SignerInfo.version` | Version other than 3 |
| `SignerInfo.sid` | Signer identifier mismatched with the embedded EE certificate’s SKI |
| `SignerInfo.digestAlgorithm` | Unsupported digest algorithm OID |
| `SignerInfo.signedAttrs` | Missing `signedAttrs` field; missing `content-type` attribute; missing `message-digest` attribute; missing `signing-time` attribute; disallowed signed attribute; duplicate signed attribute; multiple values in one attribute |
| `SignerInfo.signatureAlgorithm` | Unsupported signature algorithm OID |
| `SignerInfo.unsignedAttrs` | `unsignedAttrs` field present |
{: title="CMS Signed Object Syntax Validation Test Points" #tab-cms-syntax}

## Manifest-Specific Syntax Validation {#app-mft-syntax}

| **Field** | **Test points** |
| --- | --- |
| `version`               |  Nonzero manifest version |
| `manifestNumber`        |  Negative manifest number; manifest number exceeding 20 octets |
| `thisUpdate`/`nextUpdate` |  Stale manifest; non-GeneralizedTime encoding; `nextUpdate` earlier than `thisUpdate` |
| `fileHashAlg`           | Unsupported file hashing algorithm |
| `fileList`              | Empty file list; filename containing prohibited character  |
| `eContentType`/`content-type`  | `content-type` OID inconsistent with manifest `eContentType` |
| `certificates`  | Explicit IP resources instead of inherit |
{: title="Manifest-Specific Syntax Validation Test Points" #tab-mft-tests}

## ROA-Specific Syntax Validation {#app-roa-syntax}

| **Field** | **Test points** |
| --- | --- |
| `version` | Nonzero ROA version |
| `asID` | AS identifier outside the permitted range |
| `addressFamily` | Unsupported AFI; duplicate AFI; more than two address-family entries |
| `maxLength` | `maxLength` shorter than the prefix length; `maxLength` exceeding the address-family limit |
| `eContentType`/`content-type` | `content-type` OID inconsistent with ROA `eContentType` |
| `certificates` | IP resources specified using `inherit` |
| `certificates` | Disallowed AS resource extension present |
{: title="ROA-Specific Syntax Validation Test Points" #tab-roa-tests}

## CMS Signed Object Signature Validation {#app-cms-signature}

| **Field** | **Test points** |
| --- | --- |
| `eContent` | Encapsulated content modified after signing |
| `SignerInfo.signature` | Signature value not verifiable with the public key in the associated EE certificate |
| `signedAttrs.message-digest` | Message digest inconsistent with the encapsulated content |
{: title="CMS Signed Object Signature Validation Test Points" #tab-sig-tests}