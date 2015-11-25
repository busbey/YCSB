The YCSB project uses the version numbers on our releases to set expectations about the impact of upgrading for downstream users. Each release will have a version number that looks like _x.y.z_. Starting with version 1.0.0, these numbers map directly to:

* _Major_ version x: incremented when pre-existing custom Workloads or Database bindings would break [JVM Binary Compatibility](http://docs.oracle.com/javase/specs/jls/se7/html/jls-13.html). incremented when a change in how statistics are gathered or presented would impact performance analysis.
* _Minor_ version y: incremented when new Workloads or Database bindings are added. incremented when we add to the statistics gathered or the means to present them.
* _Patch_ version z: incremented when backwards and forwards compatible bug fixes are made.

**Note** that prior to version 1.0.0, minor version increments may introduce changes normally reserved for major versions.

Even when using the same major version, remember that YCSB is best suited for comparing systems given the same workloads and the same hardware configuration.