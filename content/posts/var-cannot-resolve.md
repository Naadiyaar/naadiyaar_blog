---
author: "Nadiyar"
title: "Var Cannot Be Resloved To A Type"
date: 2024-10-06
draft: false
ShowToc: false
---

This is the error thrown at my face by LSP (jdtls) each time I used `var` to declare
a new variable. Even though this keyword is added to Java since JDK10 and
my code compiled with no issue, LSP never stooped complaining about it.  
I got this error while I was using Java 17 and didn't have any older
JDK installed on my machine so this couldn't be an issue about JDTLS using
the wrong version of compiler.  
And more interestingly this problem only exist for Maven projects and
projects generated via Gradle were acting OK.  

**So what was the problem?** Not specifying the *source* and *target* Java versions.  
It doesn't make total sense because it's not clear what version is JDTLS
assuming we're using. So Java 8 features are supported but Java 10's aren't?

Anyways, **the solution** is specifying *source* and *target* java versions
in the POM. For example you can set Java level for `maven-compiler-plugin`.  
For this, just add below properties to your `pom.xml`:
```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
```
