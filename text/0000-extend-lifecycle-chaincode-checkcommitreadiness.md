---
layout: default
title: RFC Template
nav_order: 3
---

- Feature Name: Extend to lifecycle chaincode checkcommitreadiness to provide details of the discrepancies
- Start Date: 2023-09-XX
- RFC PR: (leave this empty)
- Fabric Component: core, lifecycle
- Fabric Issue: (leave this empty)

# Summary
[summary]: #summary

This proposal aims to extend `peer lifecycle chaincode checkcommitreadiness` by adding an option that provides additional information to diagnose why a chaincode definition's parameters and each organization's approval status are mismatched. This enhancement will facilitate root cause analysis and streamline inter-organizational coordination during operations.

# Motivation
[motivation]: #motivation

The `checkcommitreadiness` command provides a way to check whether channel members have approved the same chaincode definition. Currently, this command only outputs a true/false result, indicating if the specified chaincode definition parameters match those approved by each organization.
Since the command output lacks detailed information about mismatches, Fabric administrators often find it time-consuming to identify the root cause of discrepancies.

Real-world issues, like the one discussed in https://github.com/hyperledger/fabric/issues/3330, show that Fabric administrators may face difficulties in troubleshooting discrepancies.
Therefore, providing additional information about which parameters are causing the mismatch will enhance the lifecycle usability and assist administrators in root cause analysis.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

This RFC proposes adding a new `--inspect` option to `checkcommitreadiness` subcommand under `peer lifecycle chaincode`. This option outputs additional information to identify the cause when the approval from each organization is `false`.

*The example of executing checkcommitreadiness with inspection and JSON output format:*
```
$ peer lifecycle chaincode checkcommitreadiness --channelID [channel_name] --name [chaincode_name] --version 2.0 --init-required --sequence 2 --output json --inspect
{
   "Approvals": {
      "Org1MSP": {
        "result": true,
      },
      "Org2MSP": {
        "result": false,
        "mismatch": ["EndorsementInfo (Check the Version, InitRequired, EndorsementPlugin)", "ValidationInfo (Check the ValidationParameter, ValidationPlugin)",
        "Collections"]
      },
      "Org3MSP": {
        "result": false,
        "mismatch": ["Metadata (Check the Sequence, ChaincodeName)"]
      },
   }
}
```

*The example of executing checkcommitreadiness with inspection and Plain output format:*
```
$ peer lifecycle chaincode checkcommitreadiness --channelID [channel_name] --name [chaincode_name] --version 2.0 --init-required --sequence 2 --inspect
Chaincode definition for chaincode '[chaincode_name]', version '2.0', sequence '2' on channel
'[channel_name]' approval status by org:
Org1MSP: true
Org2MSP: false (mismatch: EndorsementInfo, ValidationInfo, Collections)
Org3MSP: false (mismatch: Metadata)
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

The proposed feature is expected to enhance the existing implementation of `checkcommitreadiness`.

In the current implementation, the check logic compares the hash values of the Private Collection without actually retrieving each organization's approved definition information (Private/Org Scope Implicit Collection).

Reference Information:
- [Link 1](https://github.com/hyperledger/fabric/blob/8a33e5f53e2f7ba6dd202d3d69926903542cda0a/core/chaincode/lifecycle/lifecycle.go#L668)
- [Link 2](https://github.com/hyperledger/fabric/blob/8a33e5f53e2f7ba6dd202d3d69926903542cda0a/core/chaincode/lifecycle/serializer.go#L222)

Since `checkcommitreadiness` cannot fully retrieve the approved definition information, it will support root cause analysis by providing mismatch information for the following field definitions:
- Metadata (includes ChaincodeName, Sequence)
- EndorsementInfo (includes Version, EndorsementPlugin, InitRequired)
- ValidationInfo (includes ValidationPlugin, ValidationParameter)
- Collections

While this won't fully pinpoint the cause, knowing which of these four types is mismatched will provide clues for parameters review and modification, thus being helpful for troubleshooting.

# Drawbacks
[drawbacks]: #drawbacks

None.

# Rationale and alternatives
[alternatives]: #alternatives

- Simply outputting this additional information as logs would be beneficial.
  - However, considering the usability for Fabric administrators, it's more desirable to check this information directly via command rather than referring to logs.
  - Implementing both a new command option and additional log output could also be an alternative approach.

# Prior art
[prior-art]: #prior-art

None.

# Testing
[testing]: #testing

- Enhance existing unit and integration tests on 'peer lifecycle chaincode checkcommitreadiness' command.

# Dependencies
[dependencies]: #dependencies

- fabric-protos
	- This feature needs to extend lifecycle protos.

# Unresolved questions
[unresolved]: #unresolved-questions

None.
