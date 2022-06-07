# PR Submission Process

## 1. Code author creates pull request

- Reference Github issue (if there is one).
- Code author to complete the PR submission template (see section below).
- Completed template to be pasted into PR as the first comment.
- Include [foundry gas reports](https://book.getfoundry.sh/forge/gas-reports.html). If updating pre-existing code, please include both before and after reports.
- Include report from [SMTChecker](https://docs.soliditylang.org/en/latest/smtchecker.html) or other analysis if conducted.
- Include link to recorded video walkthrough.
- Assign a priority tag to the PR if it needs to be done faster than 3 days.
- Assign reviewers and notify them directly (via Discord, Slack).
- Consider inviting persons outside the smart contract team (i.e. front-end) or non devs (marketing, etc) when demoing a new feature or an important change.

## 2. Recorded Code Walkthrough  

##### Code author will record a video walkthrough with no audience

- This supports an async working environment.  
- Explain the context and significance of change (demo where possible).
- For bug fixes, present a before and after for clear comprehension.
- Highlight areas of risk and complexity.
- Where necessary, tests should be explained - if they assist reviewer in comprehesion, or otherwise contain complexity themselves.
- Subsequent to this, reviewers may arrange for a follow-up video call if needed.
- Upload video to Google drive folder: _Google drive link_ (Filename synthax: PR_name_ddmmyy)

###### Recording Tips

- Breakdown the walkthrough into multiple parts and record them individually - this makes it an easier process for yourself.
- Approach the filming process as if you were doing a video tutorial: use of annotations, callouts, and other call to actions features.
- When combining the various parts together, do include visual seperators and bookmarks for ease of navigation. This is especially useful if its a long video.

**Free screen recorder software:**

- [Free Screen Recorder](https://www.apowersoft.com/free-online-screen-recorder)
- [Xbox Game bar](https://www.theverge.com/2020/4/21/21222533/record-screen-pc-windows-laptop-xbox-game-bar-how-to) (_Windows users_)
- [Camtasia](https://www.techsmith.com/download/camtasia/) (_30-day trial. Just remove and reinstall after_)
- [OBS](https://obsproject.com/)

> From my experience, Camtasia has been wonderfully easy for a new user to jump in and get started. OBS while versatile and free, has an initial steepness to the learning curve.

## 3. Final approval and merging of PR

##### The following must be completed by the code author before merging PR:

- Address any comments or changes requested, be sure to mark comments as resolved.
- Notify reviewers if any changes are made to obtain final approval.
- Must have at least 1 approval prior to merge.
- Merge PR and close related Github issue.

## For Reviewers

The implementer is only the first line of defense. As a reviewer, confirm that the implementer followed the Yieldcurity principals (test-per-state-transition, test-per-revert, fuzz test, and integration test). Review the documentation and ensure the implementation matches the documented behavior. If it does not, touch base with the implementer and confirm what needs to be updated.

- Reviewers should conduct an independent, thorough review.
- Ideally the review should be completed within 3 days of being notified (faster if priority tagged).
- Refer to the [Yield smart contract code quality standards](/yieldcurity_standard.md) code quality standards.

##### Make note of the kinds of variables that affect the feature:

- User input
- Existing state
- Other protocols
- Time

#### Some general guidelines:

- Construct a mental model of what you expect the contracts to look like before checking out the code.
    - Investigate areas that are surprising or deviate from expectations.
- Flesh out the risk surface (i.e. external contract interactions, adversarial actions).[^1]
    - Are new attack vectors introduced (if so, have they been mitigated)?
    - Are prior mitigated attack vectors, now unmitigated?
    - Look at areas that can involve **value exchange**.
    - Look at areas that **interface with external contracts**; ensure all assumptions about them are valid.
- Conduct another review from the perspective of an adversary with complete insider knowledge.
- Consider these additional Yield security precautions:
   - Never ever allow users to call arbitrary contracts from our contracts.
   - Either use auth or some kind of whitelist.

- Before reviewing tests, construct a mental model of how you would approach testing:
    - Are realistic scenarios/values used in tests?
    - Are all important edge cases and states being tested?
    - For bug fixes, is there a new test that would have caught the bug?
    - If there are any issues that should hold up merge, mark the PR as Changes Requested.

- After reviewing the code changes, see the [Code](/yieldcurity_standard.md#code), [Contract](/yieldcurity_standard.md#contract), and [DeFi](/yieldcurity_standard.md#defi) checklists.

***

# PR submission template

Code author creates pull request and completes this template, submitting as a comment at the start of the PR.
This is to assist not only the reviewer, but also to ensure that the code author is in compliance with the "Yieldcurity Standard" in a sensible manner.

**Template begins below:**

*** 

#### Objective: 
> State the objective of the PR - i.e. new feature/bug fix. Include a meaningful high-level overview explaining the changes made.

#### Dependencies: 
> List all libraries and imports utilized, e.g. Ownable.sol from OZ

#### Externalities: 
> List all external (to this PR) contracts that are interacted with - within Yield ecosystem and without.

> Reviewer to Note: Examine every component the feature may touch. Then go back through excluded components and ensure they cannot be affected.

#### Complexity: 
> Highlight areas of abstraction/complexity; e.g. functions that have complicated math or are deeply layered.

> Example: ContractA.sol, Function doSomeComplexMath()

#### Risk: 
> Highlight areas of risk. E.g. Assembly code blocks, use of unchecked, function call made to an external contract that lies outside of the Yield ecosystem. 

> Example: fn A in contractB.sol makes a call to someexternalcontract.sol


#### Contracts:
> This section serves as a brief for the reviewer, code author is to introduce contracts and elaborate on salient points

> ContractA.sol -> description + points

> ContractB.sol -> description + points

> Reference: See section 'New contracts and areas of risk'[^2]


#### Testing:
> Elaborate on your testing approach - assumptions made, concerns and any caveats.

> Map your state inheritance testing flow. This could be in the form of a text tree or an illustrated diagram (excalidraw/draw.io) 

> Negative tests, Positive tests, Adversarial and Transition tests should be listed out in each state node. [^3]  

> WHY: Offers a high-level risk surface of the testing coverage. Implementer can be certain they have not missed any possible states. Auditor quickly appreciate what has been tested, and spot any gaps.  

**Example:** 
```
(StateZeroTest) 
(actions available: deposit, withdraw, redeem)
- negative tests
- adversarial tests
- positive tests
- transition test

(StateSomeStateTest)
(actions available)
- etc,...
```

#### Open items not included at submission of this pr:
 - [ ] Item A
 - [ ] Item B
 - [ ] Item C

#### Changes requested by reviewer:
 - [ ] Change A
 - [ ] Change B
 - [ ] Change C

*** 

### Approval Sign-off:
 - [ ] Approver

> Mention whom signed off on this PR and provide verification evidence (screenshot, etc)


[^1]: https://github.com/runtimeverification/verified-smart-contracts/wiki/List-of-Security-Vulnerabilities
[^2]: https://github.com/yieldprotocol/yieldspace-tv/pull/3
[^3]: https://github.com/calnix/Collateralized-Vault/blob/AddedCode/Readme.md