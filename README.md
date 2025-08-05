# Ro-Crate Extensibility Specification


## Purpose

This repository contains a RO-Crate Profile specification.

## Structure

Each specification is in a separate directory.

This directory contains a different version.

```
/A.B.x/
```

Each version contains a specification file, `spec.md`. 

The specification file indicates which RO-Crate versions it is compatible with. 

PATCH versions are contained in the same directory, PATCHES are intended to fix issues that do not change the workings of the specification, e.g. typos. 

Aside from this, the specifications are not changed after release.

```
/A.B.x/spec.md
```

It may also contain library code in both source and compiled forms, if applicable.

```
/A.B.x/spec.md/lib/
```

And examples, if applicable.

```
/A.B.x/spec.md/examples/
```
