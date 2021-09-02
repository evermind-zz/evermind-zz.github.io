---
layout: post
title:  "Android Gradle Flavor - adjust generated tasks"
author: evermind
categories: [ android, gradle ]
---
If you add productFlavors to an android `build.gradle` file, you have to take
care to adjust the use of generated property like `'preDebugBuild'`.

If we have a flavor called `brave` the property `preDebugBuild` will no longer
be available and replaced by `preBraveDebugBuild`. So we have to also reflect that
in our `build.gradle` file.

So if we want to call `.dependsOn` we have to refer to the correct property:
```diff
--- a/app/build.gradle
+++ b/app/build.gradle
@@ -189,7 +189,7 @@ afterEvaluate {
-    preDebugBuild.dependsOn formatKtlint, runCheckstyle, runKtlint
+    preBraveDebugBuild.dependsOn formatKtlint, runCheckstyle, runKtlint
 }
```
