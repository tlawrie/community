---
status: proposed
title: Changing the way result parameters are stored
creation-date: '2021-09-27'
last-updated: '2021-09-27'
authors:
- '@tlawrie'
---

# TEP-0086: Changing the way result parameters are stored

<!--
**Note:** When your TEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary", and "Motivation" sections.
  These should be easy if you've preflighted the idea of the TEP with the
  appropriate Working Group.
- [ ] **Create a PR for this TEP.**
  Assign it to people in the SIG that are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the TEP clarified and merged quickly.  The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a TEP is merged does not mean it is complete or approved.  Any TEP
marked as a `proposed` is a working document and subject to change.  You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing TEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused.  If you disagree with what is already in a document, open a new PR
with suggested changes.

If there are new details that belong in the TEP, edit the TEP.  Once a
feature has become "implemented", major changes should get new TEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/teps/NNNN-TEP-template/README.md).

-->

<!--
This is the title of your TEP.  Keep it short, simple, and descriptive.  A good
title can help communicate what the TEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a TEP and for
highlighting any additional information provided beyond the standard TEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
  - [Use Cases (optional)](#use-cases-optional)
- [Requirements](#requirements)
- [Proposal](#proposal)
  - [Notes/Caveats (optional)](#notescaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
  - [User Experience (optional)](#user-experience-optional)
  - [Performance (optional)](#performance-optional)
- [Design Details](#design-details)
- [Test Plan](#test-plan)
- [Design Evaluation](#design-evaluation)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Infrastructure Needed (optional)](#infrastructure-needed-optional)
- [Upgrade &amp; Migration Strategy (optional)](#upgrade--migration-strategy-optional)
- [Implementation Pull request(s)](#implementation-pull-request-s)
- [References (optional)](#references-optional)
<!-- /toc -->

## Summary

To enhance the usage experience of result parameters by end users, we want to change the way result parameters are stored to allow for greater storage capacity yet with the current ease of reference and no specifical additional dependencies such as a storage mechansim.

## Motivation

<!--
This section is for explicitly listing the motivation, goals and non-goals of
this TEP.  Describe why the change is important and the benefits to users.  The
motivation section can optionally provide links to [experience reports][] to
demonstrate the interest in a TEP within the wider Tekton community.

[experience reports]: https://github.com/golang/go/wiki/ExperienceReports
-->

The ability to improve result parameter storage size as part of a TaskRun without needing to have access to additional storage. 

Additionally this will help projects that wrap/abstract Tekton where users understand how to reference result parameters between tasks and dont have the ability to adjust a Task to retrieve from a storage path.

### Goals

<!--
List the specific goals of the TEP.  What is it trying to achieve?  How will we
know that this has succeeded?
-->

* Allow larger storage than the current 4096 bytes of the Termination Message.
* Allow users to reference a result parameter in its current form
* Use existing objects (or standard ones incl CRDs) where the complexity _can_ be abstracted from a user.

### Non-Goals

<!--
What is out of scope for this TEP?  Listing non-goals helps to focus discussion
and make progress.
-->

* Not attempting to solve the storage of blobs

### Use Cases (optional)

<!--
Describe the concrete improvement specific groups of users will see if the
Motivations in this doc result in a fix or feature.

Consider both the user's role (are they a Task author? Catalog Task user?
Cluster Admin? etc...) and experience (what workflows or actions are enhanced
if this problem is solved?).
-->

## Requirements

<!--
Describe constraints on the solution that must be met. Examples might include
performance characteristics that must be met, specific edge cases that must
be handled, or user scenarios that will be affected and must be accomodated.
-->

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation.  The "Design Details" section below is for the real
nitty-gritty.
-->

This proposal provides options for handling a change to result parameter storage and potentially involves adjusting the `entrypoint` binary to write to this alternate implementation.

We need to consider both performance and security impacts of the changes and trade off with the amount of capacity we would gain from the option.

### Notes/Caveats (optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above.
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

* The storage of the result parameter may still be limited by a number of scenarios, incl
  - 1.5 MB CRD size
  - The total size of the PipelineRun _if_ it was to be patched back into an existing TaskRun object based on aggregate size of these objects.

* Eventually this may result in running into a limit with etcd, however not a problem for now. Can be solved via cleanups / offloading history.

* We want to try and minimize reimplementing access control in a webhook (in part because it means the webhook needs to know which task identities can update which task runs, which starts to get annoyingly stateful)

### Risks and Mitigations

<!--
What are the risks of this proposal and how do we mitigate. Think broadly.
For example, consider both security and how this will impact the larger
kubernetes ecosystem.

How will security be reviewed and by whom?

How will UX be reviewed and by whom?

Consider including folks that also work outside the WGs or subproject.
-->

* Concern on using etcd as a database

### User Experience (optional)

<!--
Consideration about the user experience. Depending on the area of change,
users may be task and pipeline editors, they may trigger task and pipeline
runs or they may be responsible for monitoring the execution of runs,
via CLI, dashboard or a monitoring system.

Consider including folks that also work on CLI and dashboard.
-->

* Users must be able to refer to these result parameters exactly as they would know and still be able to reference in subsequent tasks (i.e. read access)

### Performance (optional)

<!--
Consideration about performance.
What impact does this change have on the start-up time and execution time
of task and pipeline runs? What impact does it have on the resource footprint
of Tekton controllers as well as task and pipeline runs?

Consider which use cases are impacted by this change and what are their
performance requirements.
-->

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable.  This may include API specs (though not always
required) or even code snippets.  If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.

If it's helpful to include workflow diagrams or any other related images,
add them under "/teps/images/". It's upto the TEP author to choose the name
of the file, but general guidance is to include at least TEP number in the
file name, for example, "/teps/images/NNNN-workflow.jpg".
-->

1. N Configmaps: Per TaskRun or Per pipeline with Patch Merges (concern on parallelism even with queued Patch Merges)

  - Suppose hypothetically that the pipelines controller were to create N ConfigMaps** for each of N results, it could also grant the workload access to write to these results using one of these focused Roles: 
    - https://github.com/tektoncd/pipeline/blob/9c61cdf6d4b7b5e26c787d62447c0eed1c92b68f/config/200-role.yaml#L100
    - The ConfigMaps**, the Role, and the RoleBinding could all be OwnerRef'd to the *Run, to deal with cleanup.
  - This will result in the pipelines controller being given more power, i.e. to create and delete roles and rolebindings
    - Concern around having to create a new ConfigMap**, Role and RoleBinding per TaskRun, when at the end of the day we don't actually care about updating that ConfigMap**, but the TaskRun's results.
  - This option won't 'scale fail' which would happen with self definition update when the definition itself starts to get too big in unrelated areas (aggregate limit).

2. CRD

  - Would limit the chance of editing with `kubectl edit cm <results>`
  - Similar benefits to Configmap from a Role and Rolebinding perspective
  - Webhook to validate the write once immutability

3. A dedicated HTTP Service

  - Potential Auth and HA problems
  - Could run as part of the controller
  - Could be a separate HTTP server(s) which write to TaskRuns (or even config maps); task pods connect to this server to submit results, this server records the results (means result size would be limited to what can be stored in the CRD but there probably needs to be an upper bound on result size anyway)

4. Self-update / mutate the TaskRun via admission controller (with the various controls i.e. first write, subsequent read only)

  - Potential issue with self updating its own Status


## Test Plan

<!--
**Note:** *Not required until targeted at a release.*

Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all of the test cases, just the general strategy.  Anything
that would count as tricky in the implementation and anything particularly
challenging to test should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations).
-->

## Design Evaluation
<!--
How does this proposal affect the reusability, simplicity, flexibility 
and conformance of Tekton, as described in [design principles](https://github.com/tektoncd/community/blob/master/design-principles.md)
-->

## Drawbacks

<!--
Why should this TEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider and why did you rule them out?  These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

There is the current alternative of storing result parameters as data in a workspace, however Workspaces
- require there has to be a storage mechanism in the cluster that can be shared between Tasks. That can be complex, or have performance issues in itself if using an As A Service that orders the storage at spin-up time. Or forces Tasks to all run on the same node. etc. Storage is a big complex adoption hurdle.
- changes the way end users refer to the result parameter or pass between containers
- requires some tasks to be altered to retrieve data from the file system in a certain location. This makes it difficult to use a library of Tekton Tasks or an abstraction that doesn't provide access to where a parameter comes from.

## Infrastructure Needed (optional)

<!--
Use this section if you need things from the project/SIG.  Examples include a
new subproject, repos requested, github details.  Listing these here allows a
SIG to get the process for these resources started right away.
-->

## Upgrade & Migration Strategy (optional)

<!--
Use this section to detail wether this feature needs an upgrade or
migration strategy. This is especially useful when we modify a
behavior or add a feature that may replace and deprecate a current one.
-->

TBD. Potentially feature flag depending on the object used and security role changes.

## Implementation Pull request(s)

<!--
Once the TEP is ready to be marked as implemented, list down all the Github
Pull-request(s) merged.
Note: This section is exclusively for merged pull requests, for this TEP.
It will be a quick reference for those looking for implementation of this TEP.
-->

## References (optional)

[Original issue](https://github.com/tektoncd/pipeline/issues/4012)