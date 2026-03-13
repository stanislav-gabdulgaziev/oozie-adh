
## D-004
Temporarily exclude sharelib/pig from bootstrap build.

Reason:
- Pig actions are not required for the current ADH migration scope.
- sharelib/pig fails to compile due to missing Hadoop API classes in compile classpath.
- Excluding this module allows proceeding to required modules such as webapp and server.

## D-005
Temporarily exclude examples module from bootstrap build.

Reason:
- Example applications are not required for the current ADH migration scope.
- examples module fails to compile due to missing Hadoop API classes in compile classpath.
- Excluding examples allows moving forward to required runtime modules such as webapp and server.

## D-006
Use -Dmaven.test.skip=true for bootstrap build.

Reason:
- oozie-tools test sources are incompatible with Hadoop 3.3.6 API in the current build environment.
- The failure is in test compilation, not in main runtime code.
- Skipping test compilation allows proceeding to server and distro artifacts required for ADH integration.

## D-007
Temporarily exclude zookeeper-security-tests from bootstrap build.

Reason:
- The module is not required for the current ADH integration scope.
- The failure is caused by legacy transitive dependency resolution through conjars.
- Core runtime artifacts (server, distro, webapp) already build successfully without this module.

## D-010
Include Hadoop runtime client jars into distro lib/.

Reason:
- runtime startup on Hadoop 3.3.6 failed with:
  NoClassDefFoundError: org/apache/hadoop/mapred/JobConf
- successful manual workaround required:
  cp -v libext/*.jar lib/
- distro packaging is updated so Hadoop runtime jars are copied during build
  and included directly into /lib in the resulting runtime distribution.

Artifacts included:
- hadoop-auth
- hadoop-common
- hadoop-hdfs-client
- hadoop-mapreduce-client-common
- hadoop-mapreduce-client-core
- hadoop-mapreduce-client-jobclient
- hadoop-yarn-api
- hadoop-yarn-common

## D-011
Exclude docs artifact from distro assembly.

Reason:
- distro assembly expected docs/target/oozie-docs-<version>-docs.zip
- assembly failed while packaging distro on this artifact
- docs bundle is not required for runtime startup and ADH compatibility smoke testing
- focus of current bootstrap is a reproducible runtime distribution

## D-012
Exclude nested client tarball from distro assembly.

Reason:
- distro assembly expected client/target/oozie-client-<version>-client.tar.gz
- assembly failed while packaging distro on this artifact
- nested client tarball is not required for runtime startup and ADH compatibility smoke testing
- focus of current bootstrap is a reproducible runtime distribution

## D-013
Bind sharelib assembly to package phase.

Reason:
- distro assembly expects sharelib/target/oozie-sharelib-<version>.tar.gz
- sharelib module configured maven-assembly-plugin but did not bind it to package
- as a result, sharelib tarball was not produced during normal reactor build
- sharelib tarball is required for later Oozie integration scenarios on Hadoop/ADH

Scope:
- bootstrap/runtime packaging
- keeps sharelib artifact available for subsequent sharelib create / HDFS upload steps

## D-014
Include missing runtime scripts into distro bin/.

Reason:
- fresh runtime distro was missing scripts required by startup and CLI usage:
  ooziedb.sh, oozie, oozie-diag-bundle-collector.sh, instrumentation-log-parser.py
- oozie-sys.sh invokes bin/ooziedb.sh during startup
- oozie CLI script is sourced from client/src/main/bin/oozie
- auxiliary runtime scripts are sourced from tools/src/main/bin

Scope:
- runtime/bootstrap packaging for reproducible Oozie startup and CLI availability

## D-015
Bind tools assembly to package phase.

Reason:
- fresh runtime distro was missing libtools content required by ooziedb.sh
- OozieDBCLI class is packaged in tools/target/oozie-tools-<version>.jar
- distro assembly expects expanded tools bundle under tools/target/oozie-tools-<version>-tools/...
- tools module configured assembly plugin but did not bind it to package

## D-016
Include commons-cli in distro lib/.

Reason:
- fresh runtime distro failed during ooziedb.sh execution with:
  ClassNotFoundException: org.apache.commons.cli.ParseException
- old working runtime contained lib/commons-cli-1.2.jar
- fresh distro contained commons-cli only inside embedded webapp, which is insufficient for CLI tools

Scope:
- runtime/bootstrap packaging for reproducible DB setup and startup


## D-017
Include core Oozie runtime jars in distro lib/.

Reason:
- fresh runtime distro failed during ooziedb.sh execution with missing Oozie classes
- BuildInfo.class is packaged in oozie-client-<version>.jar
- CLI/bootstrap path relies on lib/* and libtools/*, not on embedded webapp WEB-INF/lib
- include oozie-client and oozie-core jars directly into distro lib for reproducible CLI/runtime startup

Scope:
- runtime/bootstrap packaging only

## D-018
Avoid conjars-only pentaho aggdesigner dependency in Hive dependency paths.

Reason:
- `org.apache.hive:hive-exec` pulls `org.pentaho:pentaho-aggdesigner-algorithm:5.1.5-jhyde`
  through Calcite-era transitive dependencies
- that artifact depends on `conjars.org`, which is not reliable for reproducible builds
- Oozie already uses the replacement pattern in other modules:
  exclude `pentaho-aggdesigner-algorithm` and add `net.hydromatic:aggdesigner-algorithm:6.0`
- the issue was reachable through two paths in this repo:
  `sharelib/hcatalog` and the full `core` reactor path via HCatalog/WebHCat dependencies

Scope:
- [`/home/codexvpn/oozie/sharelib/hcatalog/pom.xml`](/home/codexvpn/oozie/sharelib/hcatalog/pom.xml)
- [`/home/codexvpn/oozie/core/pom.xml`](/home/codexvpn/oozie/core/pom.xml)

Verification:
- `mvn -Dmaven.repo.local=/tmp/m2repo_home -pl core -am package -DskipTests`
- result: `BUILD SUCCESS`

## D-019
Treat ADH 4.1.0 as the compatibility baseline for Oozie integration work.

Reason:
- the project goal is not a generic upstream Oozie 5.2.1 build
- the actual target is opensource Oozie deployed as an external service in
  Arenadata Hyperwave (ADH) 4.1.0
- ADH 4.1.0 defines a materially newer ecosystem baseline than upstream Oozie:
  Hadoop 3.3.6, Hive 4.0.1, Tez 0.10.4, Spark 3.5.x, Flink 1.20.x, Hue 4.11.x,
  Ozone 2.0.x
- the current upstream Oozie dependency baseline in this repository is
  significantly older:
  Hadoop 2.6.0, Hive 1.2.2, Tez 0.8.4, Spark 1.6.1, Scala 2.10
- therefore the work must be treated as a compatibility adaptation effort, not
  as a simple version bump or packaging-only exercise

Scope:
- build-time dependency alignment for Hadoop 3.3.6
- Hive / HCatalog adaptation for Hive 4.0.1 and Tez 0.10.4
- Spark integration and sharelib adaptation for Spark 3.5.x
- runtime validation in two modes:
  without Kerberos and with Kerberos / Active Directory enabled

Non-goals:
- Flink does not require a native Oozie action for this scope;
  shell-based orchestration is acceptable
- Hue and Ozone are primarily runtime integration concerns and are not, by
  themselves, reasons to introduce direct compile-time dependencies into Oozie

Verification impact:
- successful validation requires both build adaptation and runtime integration
  scenarios against an actual ADH 4.1.0-compatible stack
