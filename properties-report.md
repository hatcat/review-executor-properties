---
title: Executors review -- Properties
document: PXXXXR0
date: today
audience:
  - Library Evolution Working Group
author:
  - name: Guy Davidson
    email: <guy.cpp.wg21@gmail.com>
  - name: Ben Craig
    email: ben.craig@gmail.com
  - name: Robert Leahy
    email: rleahy@rleahy.ca
  - name: Michał Dominiak
    email: griwes@griwes.info
  - name: Alexey Kukanov
    email: alexey.kukanov@intel.com
  - name: Hartmut Kaiser
    email: hartmut.kaiser@gmail.com
  - name: Daisy Hollman (author)
    email: dshollm@sandia.gov
  - name: Jared Hoberock (author)
    email: jhoberock@nvidia.com
  - name: Gordon Brown (author)
    email: gordon@codeplay.com
---

# Abstract

This is the report of the Executors review group 3: Properties, of
paper [@P0443R13].

Over four one hour meetings we reviewed sections 2.1, 2.2.2, 2.2.9 and 2.3.

Recommended actions were largely editorial:

1. Migrate the inheritance of receiver_invocation_error from the header synopsis to the class synopsis

2. Italicise unspecified items

3. Withdraw hyphens from exposition only types unless there is precedent

4. Provide examples of user code that relies on concepts

5. Provide an annex of new identifiers being introduced

The findings below are a summary of the minutes, also attached for reference.

/*
We found a few issues with the paper. Some have to do with a general lack of
examples and the fact that the paper isn't finished. Due to the seeming
generality of the mechanism and lack of example guidance and discussion,
it is difficult to program against it without running into subtle impedance
mismatches in interfaces. Clearer guidance on "how you should use this" as
opposed to just concept definitions would be very welcome.

As expected, the paper authors were extremely responsive during the review,
and have already taken a number of issues under advisement.

We are looking forward to the next iteration of the paper.
*/

# Findings

## 2.1.1 General

The first question was about the nature of a thread pool. This was described as an execution context, in the same way that a GPU runtime is also an execution context. It is a long lived resource. An executor is a handle to the execution context, and an execution agent is what the exectuor allocates: a function invoking some place with some properties.

Secondly, the lifetime of an execution agent has its own paragraph, and a question was raised about why this is warranted. This is because some of the properties refer to the lifetime of the agent. Sometimes each execution agent gets its own thread, or a unique thread that was already present. During the drafting of P0443 it was insisted that this be stated.

Sometimes the agent might have a binding to a particular resource or create resources. Describing lifetimes of the things dependent on the execution agent is assisted by this definition. The execution context is long-lived, while the agent is created to execute a particular function. It isn't reused. An agent could reuse a given resource, but each agent is unique.

## 2.1.2 Header <execution> synopsis

The long namespace, execution, is a cause for concern.

It was observed that all of the customisation points look for member functions before free functions.

Nine concepts are defined. There was discussion about whether they need to be named or if they could be exposition-only. However, using SFINAE is very inconvenient in terms of implementation. Concepts may appear in interfaces: the executor concept definitely will. Algorithms designed around the will need to use these concepts to compose a chain of work, where you want to ensure compatibility along the chain. They are necessary for overload sets for senders and receivers. Additionally, this allows non-standard libraries to have a common vocabulary amongst each other. Even if they weren't concepts, they would need to be named requirements.

The relationship between sender and receiver was discussed at length. To determine that something is a sender, you need a receiver too. If you just have the sender in isolation and can't test against a receiver there needs to be a way to advertise that something is a sender. Without that, sender_traits can't be used. Having said that, the issue of whether or not traits classes are discouraged in modern C++ was raised.

It is important to standardise these properties because some of these are likely to be used by the standard library. For example, the parallel STL will need bulk_execute, while the networking TS will need others. Combinations of properties must make sense: not every executor needs to be concerned with the properties outlined.

The memory allocation properties gave reviewers pause. std::execution::allocator is an instance, while std::allocator is a template. However, it enables statements like
auto ex_with_alloc = require(ex, execution::allocator(alloc));
which is seen as valuable.

There was discussion about executor_shape. In general, it is a hierarchy of nested multiple dimensions. There was consideration of having the shape as an interval; some said that bulk_execute should take a range rather than a single integer, but platform APIs do better with a single integer. Having bulk_execute produce a contiguous set of indices allows it to be used more generically.

## 2.2.2 Invocable archetype