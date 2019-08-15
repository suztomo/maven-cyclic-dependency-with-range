# Project to reproduce StackOverFlowError in Maven

https://github.com/GoogleCloudPlatform/cloud-opensource-java/issues/842

```
cd module-b-0
mvn install
cd ../module-a
mvn install
cd ../module-b-1
mvn install
cd ../module-b-2
mvn install
```


