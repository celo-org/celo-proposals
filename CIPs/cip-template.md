---
cip: <to be assigned>
title: <CIP title>
author: 
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track, Meta or Informational>
category (*only required for Standards Track): <Ring 0, 1, 2, 3>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <CIP number(s)>
replaces (*optional): <CIP number(s)>
license: Apache 2.0
---

This is the suggested template for new CIPs.

Note that a CIP number will be assigned by an editor. When opening a pull request to submit your CIP, please
use an abbreviated title in the filename, `cip-draft_title_abbrev.md`.

## Simple Summary
If you can't explain it simply, you don't understand it well enough. Provide a simplified and layman-accessible explanation of the CIP.

## Abstract
A short (~200 word) description of the technical issue being addressed.

## Motivation
The motivation is critical for CIPs that want to change the Celo protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.

## Specification
The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow for an implementation on the Celo platform (go-celo).

## Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.

## Backwards Compatibility
All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
Test cases for an implementation are mandatory for CIPs that are affecting consensus changes. Other CIPs can choose to include links to test cases if applicable.

## Implementation
The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.

## Security Considerations
All CIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. CIP submissions missing the "Security Considerations" section will be rejected. A CIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

## License
This work is licensed under the Apache License, Version 2.0.
