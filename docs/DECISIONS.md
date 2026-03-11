
## D-004
Temporarily exclude sharelib/pig from bootstrap build.

Reason:
- Pig actions are not required for the current ADH migration scope.
- sharelib/pig fails to compile due to missing Hadoop API classes in compile classpath.
- Excluding this module allows proceeding to required modules such as webapp and server.
