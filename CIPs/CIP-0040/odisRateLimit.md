# ODIS RateLimit Structures

## Background

This is a proposed extension to the new ODIS interface put forward in [CIP-40](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0040.md) that aims to specify how rate limits are defined within Domains. The motivating use case is allowing wallets to define how often users can attempt to recover their account per the scheme outlined in [Cloud backup with PIN encryption](https://www.notion.so/Cloud-backup-with-PIN-encryption-cea30f57bdaf4945a50eba2a42e1b85c).

## Overview

The use of arrays in this document is for illustrative purposes only. It may be useful to look at the [RateLimit Structures]() section below to see where this proposal is headed before reading on.

We'd like to define a `RateLimit` structure that nests within a `Domain` and specifies an arbitrary sequence of time intervals, where the $*i*$th interval is the `delay` before the $*i*$th query attempt against the `Domain`.

Using days for simplicity, the sequence

$d_i = [ 0, 1, 1, 2, 2, 4, 0]$

Would specify that the user can start querying immediately, but must wait

- 1 day before the 2nd query
- 1 day before the 3rd query
- 2 days before the 4th query
- 2 days before the 5th query
- 4 days before the 6th and 7th queries (which can happen back to back)

From here, two different behaviors should be exposed to developers.

1. The user accumulates quota as time intervals pass.

ex. The user waits 4 days before performing the 2nd query, at which point they can query up to 3 times.

2. The user must wait at least the specified amount of time between attempts, regardless of whether they have waited for longer than necessary between previous attempts.

This option can be controlled by a boolean on a per-interval basis. That is,

$d_i = [ 0, 1, 1, 2, 2, 4, 0]$

$c_i = [0, 1, 1, 1, 0, 0, 0]$

Would dictate that the user can accumulate a quota of up to 3 by waiting 4 days before making a 2nd attempt, but regardless of whether they wait longer they will not get more quota until 2 days after their 4th attempt.

Allowing developers to combine these two behaviors is useful for avoiding situations where users make one attempt, set the task aside for a while and then burn through their entire accumulated quota the next time they try to recover their account. It's likely the last few attempts before a hard cap should be separated by strict delay intervals so the user is forced to take breaks during which they may remember their password.

## Enhancements

Many applications will want to define RateLimits where the user can perform batches of queries without delay. For instance, the user may perform 3 queries immediately before being told to wait until the next day. To make defining these sequences easier, we can use a new array of values where the $*i*$th element corresponds to how many attempts are in the $*i*$th `batch`.

For example, we can rewrite the above `RateLimit` as

$d_i = [ 0, 1, 1, 2, 2, 4]$

$c_i = [0, 1, 1, 1, 0, 0]$

$b_i = [ 1, 1, 1, 1, 1, 2]$

Finally, notice that contiguous values are repeated when $i ∈ (1,2)$. Let's call this a `stage` of the rate limit with 2 repetitions. We can make these arrays more concise by introducing a new (optional) array where the $*i*$th element denotes how many `repetitions` of each `stage` there should be.

$d_i = [ 0, 1, 2, 2, 4]$

$c_i = [0, 1, 1, 0, 0]$

$b_i = [ 1, 1, 1, 1, 2]$

$r_i = [ 1, 2, 1, 1, 1]$

## Potential Future Extensions

Simply providing these arrays may be enough for most use cases, but if we want to support longer sequences we need more concise syntax. Future versions of `Domains` that use the `RateLimit` structure could support simple mathematical expressions to easily define arbitrary sequences of intervals.

For example,

$d_i = [ 0, 1, 2, 2, 4, d_{i-1} + i]$

$c_i = [0, 1, 1, 0, 0, 1]$

$b_i = [ 1, 1, 1, 1, 2, 1]$

$r_i = [ 1, 2, 1, 1, 1, ...]$

Specifies the same `RateLimit` we've constructed above, but replaces a hard cap on attempts with a backoff function that accumulates over time.

In this way, the developer could easily define any rate limit for their `Domain` as an arbitrary sequence of time intervals.

## RateLimit Structures

```tsx
interface RateLimit {
  stages: Stage[];
}

interface Stage {
  // How long the user must wait between batches
  // of attempts in this stage. (ex. "1 day")
  delay: string;
  // Whether quota accumulates across time intervals during this stage.
  cumulative: boolean;
  // The number of continuous attempts a user gets before the next delay
  // in each repetition of this stage. Defaults to 1.
  batch?: number;
  // The number of times this stage repeats before continuing to the next stage
  // in the RateLimit array. Defaults to 1.
  repetitions?: number;
}
```

## Usage in Cloud Backup Domain

```tsx
type CloudBackupDomain = {
  name: "ODIS Cloud Backup Domain";
  version: "1";
  rateLimit: RateLimit;
  // Public key of a key-pair derived from a salt stored alongside the
  // cyphertext that is backed up in the cloud.
  publicKey: string;
};

type CloudBackupDomainOptions = {
  // EIP-712 signature over the entire request by the private key of the salt
  // derived key-pair.
  signature: string;
};
```

## Signer DB Schema Changes

To support `RateLimits`, a new table will be added to ODIS Signers that maps full `Domain` instances (or their hash) to a timestamp and nonce. The nonce and timestamp will be updated each time the signer receives a request for that `Domain` that does not violate its `RateLimit`.
