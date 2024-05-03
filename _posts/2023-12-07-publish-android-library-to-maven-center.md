---
show_subscribe: false	

title: '上传 Android Library 到 Maven Center'
date: 2023-12-06
modified: 2023-12-08
excerpt: '创建一个本地的 Android Library, 发布到 Maven 的流程'
tags:
  - Android
  - Maven
---


> 版本：1.0.1
> 
> 修改日期: 2023-12-08

## 文档
1. [Gradle signing plugin](https://docs.gradle.org/current/userguide/signing_plugin.html)
2. [GnuPG downloads](https://www.gnupg.org/download/index.html)
3. [Open pgp](https://keys.openpgp.org/)

## 准备工作

我们上传本地 Library 到 Maven Central, 需要一下几项工作：

1. 创建 [Sonatype](https://issues.sonatype.org/secure/Dashboard.jspa) 账号，创建Issue
2. 签名 [OpenPGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy#OpenPGP)
3. Library build.gradle 配置
4. 上传 Library 到 [Sonatype](https://issues.sonatype.org/secure/Dashboard.jspa)

### Sonatype 配置

#### 创建 [Sonatype](https://issues.sonatype.org) 账号

打开 [https://issues.sonatype.org/secure/Signup!default.jspa](https://issues.sonatype.org/secure/Signup!default.jspa) 创建账号
![sonatype-start](/assets/images/sonatype-start.png)

#### 创建 Issue

点击 **Create**按钮创建 Issue. 如果你用Github 创建Issue 这几个选项需要按如下规则填写：

- **Group Id:**io.github.username
- **Project URL:**[https://github.com/username/](https://github.com/fengshenyanyi/Logcat)project
- **SCM url:**[https://github.com/username/project.git](https://github.com/fengshenyanyi/Logcat.git)
![sonatype-create](/assets/images/sonatype-create.png)

否则他会提示你创建错误，让你进行修改

```text
[Permalink](https://issues.sonatype.org/browse/OSSRH-86976?focusedCommentId=1224850&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-1224850)
[Bot Central-OSSRH](https://issues.sonatype.org/secure/ViewProfile.jspa?name=central-ossrh) added a comment - 2 days ago
To register this Group Id you must prove ownership of the domain leo.com. Please complete the following steps to continue:

1. **Add a DNS TXT record to your domain** with the text: OSSRH-86555. Please read [https://central.sonatype.org/faq/how-to-set-txt-record/](https://central.sonatype.org/faq/how-to-set-txt-record/)
2. **Edit this ticket** and set Status to Open.

More info: [https://central.sonatype.org/publish/](https://central.sonatype.org/publish/)
If you do not own this domain, **you may also choose a different Group Id that reflects your project hosting. io.github.username would be valid based on your Project URL.**
To continue the registration process, please follow these steps:

1. Create a temporary, public repository called [https://github.com/username/OSSRH-86555](https://github.com/username/OSSRH-86555) to verify github account ownership.
2. **Edit this ticket**, update the Group ID field with the new GroupId, and set Status to Open.

More info: [https://central.sonatype.org/publish/requirements/coordinates/](https://central.sonatype.org/publish/requirements/coordinates/)
```

Issue **RESOLVED**后，你就可以登录 [https://s01.oss.sonatype.org/#welcome](https://s01.oss.sonatype.org/#welcome) 准备上传Lirary了。

### OpenPGP 签名

Gradle [Signing Plugin](https://docs.gradle.org/current/userguide/signing_plugin.html) 现在只支持 [OpenPGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy#OpenPGP) 签名，同时这也是 Macven central 要求的签名方式。

> The Signing Plugin currently only provides support for generating [OpenPGP signatures](https://en.wikipedia.org/wiki/Pretty_Good_Privacy#OpenPGP) (which is the signature format [required for publication to the Maven Central Repository](http://central.sonatype.org/pages/requirements.html#sign-files-with-gpgpgp)).


我们使用 Gradle [in-memory ascii-armored keys](https://docs.gradle.org/current/userguide/signing_plugin.html#sec:in-memory-keys) 进行签名. 所以只需要配置 signingKey 和 signingPassword 即可。

```groovy
signing {
  	def signingKey = findProperty("GPG_SIGNING_KEY")
    def signingPassword = findProperty("GPG_SIGNING_PASSWORD")
    useInMemoryPgpKeys(signingKey, signingPassword)
    sign publishing.publications.release
}
```

#### 1: 下载 GnuPG

执行`brew install gnupg@2.2`下载GnuPG，也可以直接到 [Gradle signing plugin](https://docs.gradle.org/current/userguide/signing_plugin.html) 下载。
brew 下载需要版本号，如果不确定版本号可以执行 `brew search gnupg` 搜索

```shell
/: brew search gnupg
==> Formulae
gnupg                      gnupg@1.4                  gnu-go
gnupg-pkcs11-scd           gnupg@2.2
/: brew install gnupg@2.2
Running `brew update --preinstall`...
```

下载完成后可以通过 `gpg --version` 查看是否安装成功。

```shell
/: gpg --version
gpg (GnuPG) 2.3.8
libgcrypt 1.10.1
Copyright (C) 2021 Free Software Foundation, Inc.
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /Users/username/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
CAMELLIA128, CAMELLIA192, CAMELLIA256
AEAD: EAX, OCB
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

#### 2：生成秘钥

执行`gpg --gen-key`生成秘钥，如果不会可以参照 [Generating a new GPG key](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)。

```shell
/: gpg --gen-key
gpg (GnuPG) 2.3.8; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name: Leo
Email address: your-email@gmail.com
You selected this USER-ID:
"Leo <your-email@gmail.com>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

输入 O 后会提示你设置密码
![sonatype-password](/assets/images/sonatype-password.png)

1. 执行`gpg -K`查看生成的秘钥.
其中 sec 的 **58C3084CA8EF5DA8724421A0F50141723D0E90A8** 为KeyID

```shell
/: gpg -K
/Users/username/.gnupg/pubring.kbx
----------------------------------------
sec   rsa1024 2022-12-14 [SC]
      58C3084CA8EF5DA8724421A0F50141723D0E90A8
uid           [ultimate] Leo (Logcat) <email@gmail.com>
ssb   rsa1024 2022-12-14 [E]
```

#### 3：添加 GPG_SIGNING_KEY/GPG_SIGNING_PASSWORD 到 gradle.properties

添加 GPG_SIGNING_KEY 到 /Users/leo.zhang/.gradle/gradle.properties

```shell
/: gpg --armor --export-secret-keys email@gmail.com  | awk 'NR == 1 { print "GPG_SIGNING_KEY=" } 1' ORS='\\n\\n' >> gradle.properties
```

添加 GPG_SIGNING_PASSWORD 到 /Users/leo.zhang/.gradle/gradle.properties

```groovy
// 执行命令后自动添加的 private key.
GPG_SIGNING_KEY=\n\n-----BEGIN PGP PRIVATE KEY BLOCK-----private key content-----END PGP PRIVATE KEY BLOCK-----\n\n
//手动添加，your password 是生成秘钥时你输入的 passphrase.
GPG_SIGNING_PASSWORD=your password
```

#### 4：上传公钥到 Openpgp

因为 [Nexus repository manager](https://s01.oss.sonatype.org/#view-repositories) 需要校验签名，所以我们需要把公钥上传到 [Open pgp](https://keys.openpgp.org/)。
执行`gpg --keyserver [https://keys.openpgp.org/](https://keys.openpgp.org/) --send-keys 58C3084CA8EF5DA8724421A0F50141723D0E90A8`

```shell
/: gpg --keyserver https://keys.openpgp.org/ --send-keys 58C3084CA8EF5DA8724421A0F50141723D0E90A8
gpg: sending key 111C6075260DD515 to https://keys.openpgp.org
```

## 配置 build.gradle

### 添加 plugin

```groovy
plugins {
    id 'maven-publish'
    id 'signing'
}
```

### 添加 publish task

参考文档: [Android publish library](https://developer.android.com/studio/publish-library)

1. project gradle.properties 配置

```groovy
# Logcat Maven
logcatGroupId=io.github.username
logcatArtifactId=com.leo.android.library.project
logcatVersion=1.0.0
```

2. build.gradle android module 配置

```groovy
android {
    namespace = 'com.leo.android.project'
    publishing {
        singleVariant('release') {
            withSourcesJar()
        }
    }

    defaultConfig {
        compileSdk 32
        minSdk 8
        versionCode 1
        versionName "1.0.0"
        version "1.0.0-release"

        aarMetadata {
            minCompileSdk = 8
        }

        testFixtures {
            enable = false
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}
```

3. build.gradle publish 配置

```groovy
// https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven
project.afterEvaluate {
    publishing {
        def logcatGroupId = project.property("logcatGroupId")
        def logcatArtifactId = project.property("logcatArtifactId")
        def logcatVersion = project.property("logcatVersion")
        def mavenUserName = project.property("mavenUserName")
        def mavenPassword = project.property("mavenPassword")

        publications {
            release(MavenPublication) {
                groupId = logcatGroupId
                artifactId = logcatArtifactId
                version = logcatVersion
                from components.release

                pom {
                    name = "Project"
                    description = "Your project description."
                    url = "https://github.com/username/project"
                    licenses {
                        license {
                            name = "The Apache License, Version 2.0"
                            url = "http://www.apache.org/licenses/LICENSE-2.0.txt"
                        }
                    }
                    developers {
                        developer {
                            id = "userid"
                            name = "username"
                            email = "username@gmail.com"
                        }
                    }
                    scm {
                        connection = "scm:git:https://github.com/username/Project.git"
                        developerConnection = "scm:git:https://github.com/username/Project.git"
                        url = "https://github.com/username/Project.git"
                    }
                }
            }

            snapshot(MavenPublication) {
                groupId = logcatGroupId
                artifactId = logcatArtifactId
                version = logcatVersion
                from components.release

                pom {
                    name = "Project"
                    description = "Your project description."
                    url = "https://github.com/username/project"
                    licenses {
                        license {
                            name = "The Apache License, Version 2.0"
                            url = "http://www.apache.org/licenses/LICENSE-2.0.txt"
                        }
                    }
                    developers {
                        developer {
                            id = "userid"
                            name = "username"
                            email = "username@gmail.com"
                        }
                    }
                    scm {
                        connection = "scm:git:https://github.com/username/Project.git"
                        developerConnection = "scm:git:https://github.com/username/Project.git"
                        url = "https://github.com/username/Project.git"
                    }
                }
            }
        }

        repositories {
            maven {
                def releasesRepoUrl = 'https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/'
                def snapshotsRepoUrl = 'https://s01.oss.sonatype.org/content/repositories/snapshots/'
                url = version.endsWith('release') ? releasesRepoUrl : snapshotsRepoUrl

                credentials {
                    username = mavenUserName
                    password = mavenPassword
                }
            }
        }
    }

    // https://docs.gradle.org/current/userguide/signing_plugin.html#sec:signing_publications
    signing {
        def signingKey = findProperty("GPG_SIGNING_KEY")
        def signingPassword = findProperty("GPG_SIGNING_PASSWORD")
        useInMemoryPgpKeys(signingKey, signingPassword)

        sign publishing.publications.release
    }
}
```
> Task :library:preBuild UP-TO-DATE
> Task :library:preReleaseBuild UP-TO-DATE
> Task :library:compileReleaseAidl NO-SOURCE
> Task :library:mergeReleaseJniLibFolders UP-TO-DATE
> Task :library:mergeReleaseNativeLibs NO-SOURCE
> Task :library:stripReleaseDebugSymbols NO-SOURCE
> Task :library:copyReleaseJniLibsProjectAndLocalJars UP-TO-DATE
> Task :library:compileReleaseRenderscript NO-SOURCE
> Task :library:generateReleaseResValues UP-TO-DATE
> Task :library:extractDeepLinksForAarRelease UP-TO-DATE
> Task :library:generateReleaseBuildConfig UP-TO-DATE
> Task :library:generateReleaseResources UP-TO-DATE
> Task :library:packageReleaseResources UP-TO-DATE
> Task :library:parseReleaseLocalResources UP-TO-DATE
> Task :library:processReleaseManifest UP-TO-DATE
> Task :library:generateReleaseRFile UP-TO-DATE
> Task :library:compileReleaseKotlin UP-TO-DATE
> Task :library:extractReleaseAnnotations UP-TO-DATE
> Task :library:javaPreCompileRelease UP-TO-DATE
> Task :library:compileReleaseJavaWithJavac UP-TO-DATE
> Task :library:mergeReleaseGeneratedProguardFiles UP-TO-DATE
> Task :library:mergeReleaseConsumerProguardFiles UP-TO-DATE
> Task :library:mergeReleaseShaders UP-TO-DATE
> Task :library:compileReleaseShaders NO-SOURCE
> Task :library:generateReleaseAssets UP-TO-DATE
> Task :library:packageReleaseAssets UP-TO-DATE
> Task :library:packageReleaseRenderscript NO-SOURCE
> Task :library:prepareLintJarForPublish UP-TO-DATE
> Task :library:prepareReleaseArtProfile UP-TO-DATE
> Task :library:processReleaseJavaRes NO-SOURCE
> Task :library:mergeReleaseJavaResource UP-TO-DATE
> Task :library:syncReleaseLibJars UP-TO-DATE
> Task :library:writeReleaseAarMetadata UP-TO-DATE
> Task :library:bundleReleaseAar UP-TO-DATE
> Task :library:sourceReleaseJar UP-TO-DATE
> Task :library:generateMetadataFileForReleasePublication
> Task :library:generatePomFileForReleasePublication
> Task :library:signReleasePublication

### 上传 Library

gradle rebuild 之后，查看 gradle task
![gradle-task](/assets/images/gradle-task.png)

点击 `publishReleasePublicationToMavenRepository`，执行以下 tasks 后就可以去 [stagingRepositories](https://s01.oss.sonatype.org/#stagingRepositories) 查看 Android studio 上传上了的包了。

```shell
> Task :library:preBuild UP-TO-DATE
> Task :library:preReleaseBuild UP-TO-DATE
> Task :library:compileReleaseAidl NO-SOURCE
> Task :library:mergeReleaseJniLibFolders UP-TO-DATE
> Task :library:mergeReleaseNativeLibs NO-SOURCE
> Task :library:stripReleaseDebugSymbols NO-SOURCE
> Task :library:copyReleaseJniLibsProjectAndLocalJars UP-TO-DATE
> Task :library:compileReleaseRenderscript NO-SOURCE
> Task :library:generateReleaseResValues UP-TO-DATE
> Task :library:extractDeepLinksForAarRelease UP-TO-DATE
> Task :library:generateReleaseBuildConfig UP-TO-DATE
> Task :library:generateReleaseResources UP-TO-DATE
> Task :library:packageReleaseResources UP-TO-DATE
> Task :library:parseReleaseLocalResources UP-TO-DATE
> Task :library:processReleaseManifest UP-TO-DATE
> Task :library:generateReleaseRFile UP-TO-DATE
> Task :library:compileReleaseKotlin UP-TO-DATE
> Task :library:extractReleaseAnnotations UP-TO-DATE
> Task :library:javaPreCompileRelease UP-TO-DATE
> Task :library:compileReleaseJavaWithJavac UP-TO-DATE
> Task :library:mergeReleaseGeneratedProguardFiles UP-TO-DATE
> Task :library:mergeReleaseConsumerProguardFiles UP-TO-DATE
> Task :library:mergeReleaseShaders UP-TO-DATE
> Task :library:compileReleaseShaders NO-SOURCE
> Task :library:generateReleaseAssets UP-TO-DATE
> Task :library:packageReleaseAssets UP-TO-DATE
> Task :library:packageReleaseRenderscript NO-SOURCE
> Task :library:prepareLintJarForPublish UP-TO-DATE
> Task :library:prepareReleaseArtProfile UP-TO-DATE
> Task :library:processReleaseJavaRes NO-SOURCE
> Task :library:mergeReleaseJavaResource UP-TO-DATE
> Task :library:syncReleaseLibJars UP-TO-DATE
> Task :library:writeReleaseAarMetadata UP-TO-DATE
> Task :library:bundleReleaseAar UP-TO-DATE
> Task :library:sourceReleaseJar UP-TO-DATE
> Task :library:generateMetadataFileForReleasePublication
> Task :library:generatePomFileForReleasePublication
> Task :library:signReleasePublication
```

## Release

登录 [stagingRepositories](https://s01.oss.sonatype.org/#stagingRepositories)，点击你的 respository, 点击 Activity。如果校验成功点击 Close, 然后点击 Release.
Lirary 就 release 成功了。等几个小时后就可以去 [Maven Central Search](https://search.maven.org/) 中 搜索你的 Library 了。
![maven-release](/assets/images/maven-release.png)

