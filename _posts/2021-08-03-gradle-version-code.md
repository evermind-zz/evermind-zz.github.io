---
layout: post
title:  "Android Gradle Flavor - versioning of Flavors"
author: evermind
categories: [ android, gradle ]
---
### Unique versionName
I created the NewPipe fork BraveNewPipe and wanted to have a special
versioning scheme. I want to keep NewPipe's scheme as I will pull their
commits from time to time. But I still want to track my changes that are
mainly only in [BraveNewPipe/NewPipeExtractor](https://github.com/bravenewpipe/NewPipeExtractor)
in the version scheme. So I came up with
`NEWPIPE_VERSION-BRAVENEWPIPE_VERSION` eg: `0.29.9-1.0.1`

### Unique versionCode
To keep the `versionCode` unique and still having NewPipe's `defaultConfig.versionCode`
somehow integrated in my fork I came up with the idea to prepend BraveNewPipe version Code`

At the moment the `versionCode` for NewPipe has 3 digits eg `973`.
To 'prepend' BraveNewPipe's version code I set it for the first version to 1000
and instructed gradle to calculate the `versionCode`. The unique `versionCode`
will be `1973`. See below:

```gradle
// use productFlavors to keep the name/version changes AFAP for BraveNewPipe
// more separate in hope of not getting to many merge conflicts
flavorDimensions 'default'
productFlavors {
    // the amount of trailing zeros depends on the amount of digits the
    // defaultConfig.versionCode has -> we just prepend our increasing
    // versionCode before those zeros.
    def braveVersionCode = 1000
    // -> our versionName will be added as suffix to defaultConfig.versionName
    // We use major.minor.patch
    def braveVersionName = "1.0.0"

    brave {
        dimension 'default'
        applicationId "com.github.bravenewpipe"
        resValue "string", "app_name", "BraveNewPipe"
        versionCode defaultConfig.versionCode + braveVersionCode
        versionName "${defaultConfig.versionName}-${braveVersionName}"
    }
}
```

### sources
[https://medium.com/@manas/manage-your-android-app-s-versioncode-versionname-with-gradle-7f9c5dcf09bf](https://medium.com/@manas/manage-your-android-app-s-versioncode-versionname-with-gradle-7f9c5dcf09bf)
