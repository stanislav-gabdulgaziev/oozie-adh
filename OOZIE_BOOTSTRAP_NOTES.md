# Apache Oozie Bootstrap Notes

## Project
Oozie integration research for ADH platform.

## Base version
Apache Oozie 5.2.1

## Target platform
Arenadata Hyperwave (ADH) 4.1.0

## Build environment
Ubuntu 22.04.5

---

# Target requirement

The target scope is not a generic upstream Oozie build.

The actual goal is to validate opensource Oozie 5.2.1 as an external service
working with ADH 4.1.0 for the customer requirement set.

Mandatory platform components from the requirement:

- HDFS
- YARN
- MapReduce
- Tez
- Hive
- Spark
- Flink
- Hue
- Ozone

Mandatory security requirement:

- each component must support authentication via corporate Active Directory
  using Kerberos

Test result expectations for the final report:

- deployability and operability of opensource Oozie in ADH 4.1.0
- correct integration of Oozie with required Hadoop ecosystem components
- operation both without Kerberos and with Kerberos / Active Directory enabled

---

# ADH 4.1.0 baseline

The official ADH 4.1.0 service matrix must be treated as the target baseline
for build and runtime compatibility work.

Relevant component versions:

- Apache Hadoop / HDFS / YARN / MapReduce: 3.3.6
- Apache Hive: 4.0.1_arenadata2
- Apache Tez: 0.10.4_arenadata1
- Apache Spark3: 3.5.4_arenadata2
- Apache Flink: 1.20.1_arenadata1
- Hue: 4.11.0_arenadata4
- Apache Ozone: 2.0.0_arenadata1

Security / identity context in ADH docs:

- Kerberos is a supported deployment mode
- MS Active Directory integration is a supported deployment mode

Practical implication:

- Oozie build and runtime validation must target the ADH stack above
- assumptions based on the historical Hadoop 2.x ecosystem are not sufficient

---

# Upstream gap

Current upstream Oozie 5.2.1 build baseline in this repository is significantly
older than ADH 4.1.0.

Key version properties currently present in the build:

- Hadoop: 2.6.0
- Hive: 1.2.2
- Tez: 0.8.4
- Spark: 1.6.1
- Spark Scala binary: 2.10

This means the work is not a simple packaging exercise.

It is a compatibility adaptation effort for:

- Hadoop 3.3.6 APIs and runtime layout
- Hive 4.0.1 dependency graph and HCatalog path
- Tez 0.10.4 execution path
- Spark 3.5.x sharelib / launcher integration

Important boundary:

- Flink integration does not require a native Oozie action
- the acceptable approach is shell-based orchestration from Oozie
- Hue and Ozone are mainly integration-test concerns, not compile-time
  dependency drivers for the Oozie build

---

# Engineering direction

The build should be adapted in stages, not with a single bulk version bump.

Recommended technical order:

1. Move the general build baseline to Hadoop 3.3.6.
2. Fix compile and packaging issues caused by Hadoop 3.x API and artifact changes.
3. Adapt Hive / HCatalog integration to Hive 4.0.1 and Tez 0.10.4.
4. Adapt Spark integration and sharelib packaging to Spark 3.5.x.
5. Validate runtime deployment.
6. Validate integration scenarios without Kerberos.
7. Repeat key scenarios with Kerberos / Active Directory enabled.

---

# Bootstrap problems

Upstream Oozie does not build cleanly in modern environments.

Issues encountered:

- obsolete repositories
- incompatible Hadoop APIs
- failing tests
- packaging dependencies on removed modules
- historical dependency assumptions from Hadoop 2.x / Hive 1.x / Spark 1.x
- mismatch between upstream Oozie dependency baseline and ADH 4.1.0 stack

---

# Engineering decisions

## Remove Pig module

Module:

sharelib/pig

Reason:

Pig not required in ADH platform.

---

## Fix webapp dependencies

File:

webapp/pom.xml

Dependencies aligned with Hadoop 3.x.

---

## Disable incompatible tests

Test disabled:

TestECPolicyDisabler.java

Reason:

Hadoop API changes.

---

## Remove examples module

examples module removed from build.

---

## Fix distro assembly

File:

src/main/assemblies/distro.xml

Removed dependency on:

oozie-examples-${project.version}-examples.tar.gz

---

# Result

Partial bootstrap success.

Currently confirmed:

- `core` build succeeds
- `tools` build succeeds
- `docs` build succeeds
- `webapp` build succeeds
- dependency path for `conjars` / pentaho aggdesigner was fixed in Hive-related
  modules

Current distro packaging status:

- runtime packaging path progressed significantly
- one distro blocker was identified and corrected:
  `hadoop-hdfs-client:2.6.0` had to be aligned to `hadoop-hdfs:2.6.0`
- further distro/runtime validation still remains part of ongoing work

Current conclusion:

- the branch is useful as a bootstrap and compatibility research branch
- it is not yet a final ADH 4.1.0 validated Oozie distribution

---

# Branch

adh/bootstrap

---

# Next step

Continue adapting the build from the historical upstream baseline toward the
actual ADH 4.1.0 target stack, starting from Hadoop 3.3.6 alignment and then
moving to Hive / Tez / Spark compatibility and runtime integration tests.
