# Project to reproduce StackOverflowError in Maven

Maven throws StackOverflowError when version ranges are unsolvable and the dependency graph contains
a cycle.

https://github.com/GoogleCloudPlatform/cloud-opensource-java/issues/842

This project reproduces StackOverflowError caused by Maven's ConflictResolver.

# Steps to Reproduce the Issue

Starting from the root of this project, run the following commands:

```
rm -rf ~/.m2/repository/suztomo # cleanup previously installed modules with groupId:suztomo
cd module-b-0
mvn install
cd ../module-a
mvn install
cd ../module-b-1
mvn install
cd ../module-b-2
mvn install
cd ../module-c
mvn install # this fails
```

The last `mvn install` at module-c fails with following error:

```
[INFO] --------------------------< suztomo:module-c >--------------------------
[INFO] Building module-c 1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.406 s
[INFO] Finished at: 2019-08-16T12:10:30-04:00
[INFO] ------------------------------------------------------------------------
...
Exception in thread "main" java.lang.StackOverflowError
	at org.eclipse.aether.graph.DefaultDependencyNode.accept(DefaultDependencyNode.java:341)
	at org.eclipse.aether.graph.DefaultDependencyNode.accept(DefaultDependencyNode.java:345)
	at org.eclipse.aether.graph.DefaultDependencyNode.accept(DefaultDependencyNode.java:345)
	at org.eclipse.aether.graph.DefaultDependencyNode.accept(DefaultDependencyNode.java:345)
	at org.eclipse.aether.graph.DefaultDependencyNode.accept(DefaultDependencyNode.java:345)
	at org.eclipse.aether.graph.DefaultDependencyNode.accept(DefaultDependencyNode.java:345)
...(omitting many lines)...
```

# Diagnosis

Because of a version conflict on grpc-core (1.21.0 v.s. 1.16.1),
[`org.eclipse.aether.util.graph.transformer.NearestVersionSelector.newFailure`](
https://github.com/apache/maven-resolver/blob/maven-resolver-1.4.0/maven-resolver-util/src/main/java/org/eclipse/aether/util/graph/transformer/NearestVersionSelector.java#L158
) tries to throw `UnsolvableVersionConflictExceptoin`. However, before throwing the exception
`PathRecordingDependencyVisitor` visits nodes in the dependency graph and the graph contains a
cycle. The visitor goes to infinite recursion in visiting the cyclic path, resulting in
StackOverflowError.

```
    private UnsolvableVersionConflictException newFailure( final ConflictContext context )
    {
        ...
        PathRecordingDependencyVisitor visitor = new PathRecordingDependencyVisitor( filter );
        context.getRoot().accept( visitor );
        return new UnsolvableVersionConflictException( visitor.getPaths() );
    }
```

The cyclic path consists of module-a and module-b as illustrated below:

```
module-c:1.0.0
+- module-b:2.0.0
   +- module-a:1.0.0
      +- module-b:0.0.1
      +- module-b:1.0.0
      |  +- module-a:1.0.0
      |     +- module-b:0.0.1
      |     +- module-b:1.0.0
            ...
```


# Environment

The error was reproduced in Maven 3.6.1 and Java 8 in Linux on amd 64.
