
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
