# Project to reproduce StackOverFlowError in Maven

https://github.com/GoogleCloudPlatform/cloud-opensource-java/issues/842

```
rm -rf ~/.m2/repository/suztomo
cd module-b-0
mvn install
cd ../module-a
mvn install
cd ../module-b-1
mvn install
cd ../module-b-2
mvn install
cd ../module-c
mvn install
cd ../module-d
mvn install
```

Modify constraint in module-a


In my attempt, module-b does not have module-a as children:
https://screenshot.googleplex.com/MzeJC1AR1zf.png

