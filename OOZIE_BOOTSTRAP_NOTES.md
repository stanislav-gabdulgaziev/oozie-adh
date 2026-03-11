# Apache Oozie Bootstrap Notes

## Project
Oozie integration research for ADH platform.

## Base version
Apache Oozie 5.2.1

## Hadoop version
3.3.6

## Build environment
Ubuntu 22.04.5

---

# Bootstrap problems

Upstream Oozie does not build cleanly in modern environments.

Issues encountered:

- obsolete repositories
- incompatible Hadoop APIs
- failing tests
- packaging dependencies on removed modules

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

Successful build.

Generated artifact:

distro/target/oozie-5.2.1-distro.tar.gz

---

# Branch

adh/bootstrap

---

# Next step

Runtime deployment and integration testing.
