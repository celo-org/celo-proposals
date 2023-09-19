# CIP-40 Sequential Delay Domain

## Simple Summary

CIP-40 domain supporting signature-authenticated rate limits defined as a series of time-delayed stages.

## Background

This is an extension to the new ODIS interface put forward in [CIP-40](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0040.md) which specifies a domain supporting signature-authenticated rate limits defined as a series of time-delayed stages.
The motivating use case is allowing wallets to define how often users can attempt to recover their account via the scheme outlined in [Cloud backup with PIN encryption](https://www.notion.so/Cloud-backup-with-PIN-encryption-cea30f57bdaf4945a50eba2a42e1b85c), but the design attempts to accommodate a number of similar rate limiting goals based solely on time.

## Specification

As a starting point, we define a rate limiting structure to describe an arbitrary sequence of time intervals, where interval `i` is the `delay`, relative to attempt `i-1`, before attempt `i` can be made against the domain instance.

#### Sequential delay rate limit

| Attempt      | 1                                   | 2                                                  | 3                                             | 4                                             | 5                                              | 6                                              | 7                                                       |
| ------------ | ----------------------------------- | -------------------------------------------------- | --------------------------------------------- | --------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------- |
| delay (days) | 0                                   | 0                                                  | 1                                             | 1                                             | 2                                              | 4                                              | 0                                                       |
| explanation  | User can start querying immediately | User can make 2 initial attempts without any delay | User must wait 1 day between attempts 2 and 3 | User must wait 1 day between attempts 3 and 4 | User must wait 2 days between attempts 4 and 5 | User must wait 4 days between attempts 5 and 6 | User can make attempt 7 immediately following attempt 6 |

Notice here we determine when an attempt can be performed by adding the `delay` to the timestamp at which the preceding attempt was performed. Alternatively, we could add the delay to the _earliest timestamp at which the preceding attempt could have been performed._ These two behaviors have the following effects on quota.

1. If `delay` signifies the amount of time that must pass between attempts $i-1$ and $i$, then quota does not accumulate.
   - ex. The user waits 3 days after attempt 1, at which point their quota is 1. Regardless of when attempt 2 is made, attempt 3 must be made at least a day later.
2. If `delay` signifies the amount of time that must pass between _the earliest time attempt $i-1$ could have been performed_ and attempt $i$, then quota is accumulated as `delay` intervals pass.
   - ex. The user waits 4 days after attempt 1, at which point their quota is 4 because they have accumulated attempts 2, 3, 4 and 5.

Combining these two behaviors in account recovery allows developers to avoid situations where users make one attempt, set the task aside for a while and then burn through their entire accumulated quota the next time they try to recover their account. It's likely the last few attempts before a hard cap should be separated by strict delay intervals so the user is forced to take breaks during which they may remember their password.

We can expose both of these behaviors via a `resetTimer` boolean that is configurable per attempt.

- We keep track of a `timer` timestamp for each sequential delay domain instance that is used to determine when the next attempt will be accepted.

  ```typescript
  if (now < timer + delay) {
    return error;
  }
  ```

- The `timer` for a given sequential delay domain instance starts at 0 (Unix epoch) and is updated to the current timestamp whenever an attempt is made for which the rate limit is satisfied and `resetTimer` is true.
  When `resetTimer` is false, the `timer` is incremented by `delay` to record the earliest timestamp at which the attempt would have satisfied the rate limit.

  ```typescript
  if (now >= timer + delay) {
    timer = resetTimer ? now : timer + delay;
  }
  ```

#### Adding reset timer option

| Attempt      | 1                                                           | 2                                                                                                                      | 3                                                                                             | 4                                                                                                                       | 5                                                                                                                                                                           | 6                                                                                              | 7                                                                                                        |
| ------------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| delay (days) | 0                                                           | 0                                                                                                                      | 1                                                                                             | 1                                                                                                                       | 2                                                                                                                                                                           | 4                                                                                              | 0                                                                                                        |
| resetTimer   | yes                                                         | yes                                                                                                                    | no                                                                                            | yes                                                                                                                     | no                                                                                                                                                                          | yes                                                                                            | yes                                                                                                      |
| explanation  | The timer will be set to the current timestamp at attempt 1 | The timer will be set to the current timestamp at attempt 2. This means that 1 day must pass between attempts 2 and 3. | The user can perform attempt 4 two days after attempt 2 regardless of when attempt 3 is made. | The timer will be set to the current timestamp at attempt 4. This means that 2 days must pass between attempts 4 and 5. | The timer will be set to the timestamp at attempt 4 plus 2 days at attempt 5. This means if 6 days pass between attempt 4 and attempt 5 the user can also perform attempt 6 | The timer will be set to the current timestamp at attempt 6, which has no impact in this case. | Even if this were not the last interval in the RateLimit, the user could not have a quota greater than 2 |

Notice the first `delay` is relative to the Unix Epoch (i.e. 00:00 UTC Jan 1st 1970). This has 2 notable consequences

1. The first `delay`can be used to set an arbitrary time in the future before which the domain instance cannot be queried.
2. If the first attempt does not reset the `timer`, then quota will begin accumulating after the first `delay` and will continue to accumulate with each subsequent `delay` until `resetTimer` is true.

These behaviors may enable some interesting use cases such as lotteries and other random-selection based protocols where queries are unavailable until a certain absolute time. However, for most use cases (including account recovery) the first attempt should set `resetTimer` to true and `delay` to 0.

### Batching

Applications may require rate limits where users can perform a batch of queries after a given `delay`. In our example, notice that attempts {0, 1} and {6, 7} always become available as `batches` of 2. To help us express this, we can define rules in terms of `stages` rather than attempts.

#### Adding batching

| Attempt      | 1                                                                                                                                                             | 2   | 3   | 4   | 5                                                                                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| delay (days) | 0                                                                                                                                                             | 1   | 1   | 2   | 4                                                                                                                                                             |
| resetTimer   | yes                                                                                                                                                           | no  | yes | no  | yes                                                                                                                                                           |
| batchSize    | 2                                                                                                                                                             | 1   | 1   | 1   | 2                                                                                                                                                             |
| explanation  | Attempts 0 and 1 can be combined into a single stage with a batchSize of 2 because there is no delay between them and they have the same value for resetTimer |     |     |     | Attempts 6 and 7 can be combined into a single stage with a batchSize of 2 because there is no delay between them and they have the same value for resetTimer |

Notice that a stage with a `batchSize` of `n` is equivalent to inserting `n - 1` stages after stage `i` with `delay` 0 and `resetTimer` matching stage `i`. The `batchSize` option serves to make the rate limit structure more concise.

### Repetitions

Similarly, imagine we wish to append 6 stages to our rate limit that are duplicates of stage 5. That is, we want users to be able to use 2 attempts every 4 days at the end of our rate limit for 7 repetitions. We could then combine these into a single stage as follows.

#### Adding repetitions

| Attempt      | 1   | 2   | 3   | 4   | 5                                                             |
| ------------ | --- | --- | --- | --- | ------------------------------------------------------------- |
| delay (days) | 0   | 1   | 1   | 2   | 4                                                             |
| resetTimer   | yes | no  | yes | no  | yes                                                           |
| batchSize    | 2   | 1   | 1   | 1   | 2                                                             |
| repetitions  | 1   | 1   | 1   | 1   | 7                                                             |
| explanation  |     |     |     |     | stage 5 is repeated 7 times before the RateLimit is exhausted |

Repetitions will be useful for rate limits that follow simple repeating patterns. For example, a developer may wish to give a user 1 attempt per day for 3 weeks.

Because the default value for `repetition` and `batchSize` will be 1, developers who prefer a simpler interface or require a modest number of stages can simply ignore them.

### Structures

```typescript
interface SequentialDelayStage {
  // How many seconds each batch of attempts in this stage is delayed with
  // respect to the timer.
  delay: number;
  // Whether the timer should be reset between attempts during this stage.
  // Defaults to true.
  resetTimer: Optional<boolean>;
  // The number of continuous attempts a user gets before the next delay
  // in each repetition of this stage. Defaults to 1.
  batchSize: Optional<number>;
  // The number of times this stage repeats before continuing to the next stage
  // in the RateLimit array. Defaults to 1.
  repetitions: Optional<number>;
}

type SequentialDelayDomain = {
  name: "Sequential Delay Domain";
  version: "1";
  stages: SequentialDelayStage[];
  // Optional Celo address against which signed requests must be authenticated.
  // In the case of Cloud Backup, this will be derived from a one-time key stored with the ciphertext.
  address: Optional<string>;
  // Optional string to distinguish the output of this domain instance from
  // other SequentialDelayDomain instances
  salt: Optional<string>;
};

type SequentialDelayDomainOptions = {
  // EIP-712 signature over the entire request by the address specified in the domain.
  // Required if `address` is defined in the domain instance. If `address` is
  // not defined in the domain instance, then a signature must not be provided.
  signature: Optional<string>;
  // Used to prevent replay attacks. Required when sending a DomainRestrictedSignatureRequest
  // if a signature is provided.
  nonce: Optional<number>;
};
```

### Querying Domain Status

In response to a domain quota status request, the following status structure will be returned.
Note that this includes the `counter` field, which is used in setting and checking the query nonce. (See [Replay Handling](#replay-handling) below)

```typescript
interface SequentialDelayDomainStatusResponse {
  // How many attempts the user has already made against the domain that have
  // satisfied the rate limit
  counter: number
  // The timestamp, in seconds, to which the next delay is added to determine
  // when the next quota increase will occur
  timer: number
  // Whether the domain instance has been permanently disabled
  disabled: boolean
}
```

## Signer DB Schema Changes

To support this new domain, a new table will be added to ODIS Signers that maps the hash of full domain instances to `timer` and `counter`.
To implement the `/disableDomain` endpoint specified in [CIP-40](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0040.md) this table will also have a boolean `disabled` column.

## Replay Handling

Replaying of a `DomainRestrictedSignatureRequest` should lead to the user's quota being consumed. In order to prevent this, a `nonce` is included in the options field.
The `nonce` must be greater than or equal to the `counter` stored for a given `SequentialDelayDomain`.
When the request is processed, the `counter` value will be set to the provided `nonce` value.

If the client does not have their `counter` / `nonce` , it can be queried via quota status endpoint (See [CIP-40](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0040.md)).

Note that requests to the disable domain and quota status endpoints do not require a nonce to be specified.
This is not required because disable domain is permanent and idempotent, and quota status has no side effects.
As a result, replays against those APIs do not result in any change of state.

## Example Implementation

```typescript
interface IndexedSequentialDelayStage extends SequentialDelayStage {
  // The attempt number at which the Stage begins
  start: number;
}

interface SequentialDelayResult {
  accepted: boolean;
  state?: State;
}

interface SequentialDelayState {
  // Timestamp used for deciding when the next request will be accepted.
  timer: number;
  // Number of queries that have been accepted for the SequentialDelayDomain instance.
  counter: number;
}

const getIndexedStage = (
  domain: SequentialDelayDomain,
  counter: number
): IndexedSequentialDelayStage | undefined => {
  let attemptsInStage = 0;
  let stage = 0;
  let i = 0;
  while (i <= counter) {
    if (stage >= domain.stages.length) {
      return undefined;
    }
    const repetitions = domain.stages[stage].repetitions ?? 1;
    const batchSize = domain.stages[stage].batchSize ?? 1;
    attemptsInStage = repetitions * batchSize;
    i += attemptsInStage;
    stage++;
  }

  i -= attemptsInStage;
  stage--;

  return { ...domain.stages[stage], start: i };
};

const getDelay = (
  stage: IndexedSequentialDelayStage,
  counter: number
): number => {
  const batchSize = stage.batchSize ?? 1;
  if ((counter - stage.start) % batchSize === 0) {
    return stage.delay;
  }
  return 0;
};

const checkSequentialDelay = (
  domain: SequentialDelayDomain,
  attemptTime: number,
  state?: SequentialDelayState
): SequentialDelayResult => {
  // If no state is available (i.e. this is the first request against the domain) use the initial state.
  const counter = state?.counter ?? 0;
  const timer = state?.timer ?? 0;
  const stage = getIndexedStage(domain, counter);

  // If the counter is past the last stage (i.e. the domain is permanently out of quota) return early.
  if (!stage) {
    return { accepted: false, state };
  }

  const resetTimer = stage.resetTimer ?? true;
  const delay = getDelay(stage, counter);
  const notBefore = timer + delay;

  if (attemptTime < notBefore) {
    return { accepted: false, state };
  }

  // Request is accepted. Update the state.
  return {
    accepted: true,
    state: {
      counter: counter + 1,
      timer: resetTimer ? attemptTime : notBefore,
    },
  };
};

const t = 1631650286;

const domain: SequentialDelayDomain = {
  name: "Sequential Delay Domain",
  version: 1,
  stages: [
    { delay: t, resetTimer: true, batchSize: 2, repetitions: 1 },
    { delay: 1, resetTimer: false, batchSize: 1, repetitions: 1 },
    { delay: 1, resetTimer: true, batchSize: 1, repetitions: 1 },
    { delay: 2, resetTimer: false, batchSize: 1, repetitions: 1 },
    { delay: 4, resetTimer: true, batchSize: 2, repetitions: 2 },
  ],
};

let state: State | null = null;

// { accepted: false, state: null }
state = checkRateLimit(domain, state, t - 1).state;

// { accepted: true, state: { timer: t, counter: 1 } }
state = checkRateLimit(domain, state, t).state;

// { accepted: true, state: { timer: t+1, counter: 2 } }
state = checkRateLimit(domain, state, t + 1).state;

// { accepted: true, state: { timer: t+2, counter: 3 } }
state = checkRateLimit(domain, state, t + 3).state;

// { accepted: true, state: { timer: t+3, counter: 4 } }
state = checkRateLimit(domain, state, t + 3).state;

// { accepted: true, state: { timer: t+5, counter: 5 } }
state = checkRateLimit(domain, state, t + 6).state;

// { accepted: false, state: { timer: t+5, counter: 5 } }
state = checkRateLimit(domain, state, t + 8).state;

// { accepted: true, state: { timer: t+9, counter: 6 } }
state = checkRateLimit(domain, state, t + 9).state;

// { accepted: true, state: { timer: t+10, counter: 7 } }
state = checkRateLimit(domain, state, t + 10).state;

// { accepted: true, state: { timer: t+14, counter: 8 } }
state = checkRateLimit(domain, state, t + 14).state;

// { accepted: true, state: { timer: t+15, counter: 9 } }
state = checkRateLimit(domain, state, t + 15).state;
```

## Future Improvements

This feature will not be part of the initial implementation but could be added to later versions of rate limited domains.

### Mathematical Expressions

If we want to support long or infinite sequences of intervals we will need more concise syntax.
Future versions of the `SequentialDelayDomain` could support mathematical expressions to easily define arbitrary sequences of intervals.

#### Adding mathematical expressions

| Attempt      | 1   | 2   | 3   | 4   | 5                                                                                                                                                   |
| ------------ | --- | --- | --- | --- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| delay (days) | 0   | 1   | 1   | 2   | $min(d_{i-1} + i, 365)$                                                                                                                             |
| resetTimer   | yes | no  | yes | no  | yes                                                                                                                                                 |
| batchSize    | 2   | 1   | 1   | 1   | 1                                                                                                                                                   |
| repetitions  | 1   | 1   | 1   | 1   | ...                                                                                                                                                 |
| explanation  |     |     |     |     | Rather than impose a hard cap on attempts, the user will continue accumulating quota forever at increasingly infrequent intervals up to a year long |
