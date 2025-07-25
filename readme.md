# Random Tools

A collection of utilities for working with AMD SEV-SNP (Secure Encrypted Virtualization - Secure Nested Paging) confidential computing environments.

## Overview

This repository contains two complementary tools:
- **`get-snp-report`**: Generates SEV-SNP attestation reports from confidential hardware
- **`hex2report`**: Parses and formats SEV-SNP report data into human-readable output

## Prerequisites

- SEV-SNP enabled hardware (tested on Azure Confidential Compute Instances)
- Linux environment with bash shell
- Recommended: Azure Confidential Compute Instance using image `mcr.microsoft.com/acc/samples/aci/helloworld:2.8`

---

## get-snp-report

A tool for generating SEV-SNP attestation reports directly from confidential hardware. This utility allows you to create cryptographic proof that code is running in a genuine confidential computing environment.

**Source**: Compiled from [Microsoft's confidential-sidecar-containers](https://github.com/microsoft/confidential-sidecar-containers/tree/main/tools/get-snp-report)

### Download and Setup

```bash
# Download the binary
curl -LO https://github.com/ivarprudnikov/random-tools/raw/refs/heads/main/get-snp-report

# Make it executable
chmod a+x get-snp-report
```

### Usage

#### Generate a basic SEV-SNP report

Creates an attestation report that proves code execution in a confidential environment:

```bash
./get-snp-report
```

#### Generate a report with custom data

Include your own data in the report's `report_data` field. This is useful for:
- Passing nonces for replay protection
- Including application-specific identifiers
- Enabling downstream verifiers to validate specific data

```bash
# Example with hex data
MY_HEX_DATA="ababababab"
./get-snp-report $MY_HEX_DATA
```

The custom data will be embedded in the attestation report, allowing verifiers to confirm that the report was generated with knowledge of this specific data.

---

## hex2report

A parser that converts raw SEV-SNP report data (in hexadecimal format) into structured, human-readable output. This tool helps you understand and verify the contents of attestation reports.

**Source**: Compiled from [Microsoft's confidential-sidecar-containers](https://github.com/microsoft/confidential-sidecar-containers/tree/main/tools/get-snp-report)

### Download and Setup

```bash
# Download the binary
curl -LO https://github.com/ivarprudnikov/random-tools/raw/refs/heads/main/hex2report

# Make it executable
chmod a+x hex2report
```

### Usage

#### Parse live SEV-SNP report data

Combine both tools to generate and immediately parse a report:

```bash
./get-snp-report | ./hex2report
```

This command will:
1. Generate a fresh SEV-SNP report using `get-snp-report`
2. Parse the binary output into readable fields using `hex2report`

#### Parse existing report data

You can also parse previously collected report data:

```bash
# From base64 encoded data
echo -n "$BASE64_REPORT" | base64 -d | xxd -p | tr -d '\n' | ./hex2report

# From hex file
cat report.hex | ./hex2report
```

### Sample Output

The tool parses SEV-SNP reports into structured fields including:

```bash
# Example: Parse a base64-encoded sample report
SAMPLE_B64="AwAAAAIAAAAfAAMAAAAAAAEAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAEAAAAAAAY2yUAAAAAAAAAAAAAAAAAAABGDc4HsDoKTzpCz5PyAQWV56jaBnezoBob8IghpIvfrwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAX+7jDW1+Gin0A9cKQZgjfd+xMFGi1pdkOUh8YJOI7X+YGJiHkgqy+gCWkDoMI/yhc5c7eNcMxoNTQm3hiNtd/Fflt2bjmZNftzphEn6ibSAK15zrC2SLDmqQ2KqfbqJMM6lotmMghTUxReixmkdBotq5ujQuE75PwNIl6InMGlgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB7IvQp0syiOy7T/FvWD2tAY2VpIE5HCUiCDY//1LKv1///////////////////////////////////////////BAAAAAAAGNsZAQEAAAAAAAAAAAAAAAAAAAAAAAAAAACw/Rktl2nwt7LCFRlUTl6D/PIt5ZN7oVaBLurrFRTxd4Yf3RThp0ik4tZfZ6SEBtt2SiU8P5qXjMiBBCFwdIAeBAAAAAAAGNsdNwEAHTcBAAQAAAAAABjbAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAmr8/+gbIDhJivgdNj8r6klcaZR1utgszbq8r194pitT0+qkr+xl+FIO4hrkbf4oYAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAK50yU+gQ8H2Q9lNeUO7+6YcnCiLf8oSrcQNke7KDUayvXIbL2yIMFCCMGRPptBe2AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA="

echo -n "$SAMPLE_B64" | base64 -d | xxd -p | tr -d '\n' | ./hex2report
```

**Output format:**
```
  version:              00000003                    # Report format version
  guest_svn:            00000002                    # Guest security version number  
  policy:               000000000003001f            # Guest policy settings
  family_id:            00000000000000000000000000000001    # Processor family ID
  image_id:             00000000000000000000000000000002    # Image/firmware ID
  vmpl:                 00000000                    # Virtual Machine Privilege Level
  signature_algo:       00000001                    # Signature algorithm identifier
  current_tcb:          db18000000000004            # Current Trusted Computing Base
  platform_info:       0000000000000025            # Platform-specific information
  author_key_en:        00000000                    # Author key enablement flag
  reserved1:            00000000                    # Reserved field
  report_data:          460dce07b03a0a4f3a42cf93f2010595...  # Custom data (64 bytes)
  measurement:          5feee30d6d7e1a29f403d70a4198237d...  # Code/firmware measurement
  host_data:            73973b78d70cc68353426de188db5dfc...  # Host-provided data
  id_key_digest:        0ad79ceb0b648b0e6a90d8aa9f6ea24c...  # Identity key digest
  author_key_digest:    00000000000000000000000000000000...  # Author key digest  
  report_id:            7b22f429d2cca23b2ed3fc5bd60f6b40...  # Unique report identifier
  # ... additional fields including TCB versions, chip ID, and signature
```

## Understanding the Output

Key fields to focus on:
- **`report_data`**: Your custom data (if provided to `get-snp-report`)
- **`measurement`**: Cryptographic hash of the running code/firmware
- **`policy`**: Security policy configuration of the guest VM
- **`signature`**: Cryptographic signature validating the report authenticity

These fields enable remote attestation workflows where verifiers can:
1. Validate the report signature against AMD's signing keys
2. Check that measurements match expected values
3. Verify custom data in `report_data` field
4. Confirm the security policy meets requirements