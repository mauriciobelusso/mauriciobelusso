# ADR 0001: Virtual Threads for I/O-bound Java services

- **Status:** Accepted
- **Date:** 2025
- **Scope:** JVM microservices on Kubernetes, blocking I/O

## Context

A set of I/O-bound Java services sat on platform threads. Under load they paid for that model twice: a stack per request and CPU spent parking and unparking carriers. Growing the pool hid the wait and grew the pod.

Most of a request is time spent on the network or the database. That is the shape virtual threads were built for.

## Decision

Move I/O-bound services from Java 8 thread pools to Java 25 LTS and virtual threads. Keep the blocking style. Do not rewrite the stack as reactive just to hold more connections.

Treat `synchronized` and JNI as pinning risks. Measure carrier-thread pinning after the cutover, not only heap and CPU.

## Alternatives

1. **Stay on Java 8 / 11 pools.** Cheapest change. Does not fix the cost of a platform thread per blocking call.
2. **Spring WebFlux.** Fits the concurrency model. Forces a rewrite of every blocking library in the call path. Wrong trade for this codebase.
3. **Bigger pods, more replicas.** Works until the bill does.

## Consequences

- Average CPU dropped from about 100 mcores to 30 mcores (~70%).
- Resident memory went from 1.3 GB to 900 MB and stayed there under load.
- Pinning showed up where old code used `synchronized` around I/O. Those blocks moved to `j.u.c` locks or got shorter.
- Reactive is still the right tool for a few hot streaming paths. It is no longer the default answer to "we need more concurrency."

## What this is not

Not a client dump. Not a Loom-vs-WebFlux benchmark. The decision was for blocking, I/O-bound services that already ran on Spring.
