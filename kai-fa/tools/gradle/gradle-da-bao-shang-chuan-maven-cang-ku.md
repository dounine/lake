---
description: 如何把自己的jar包上传到maven上给别人使用？本文一步一步教你。
---

# Gradle 打包上传Maven仓库

生成gpg密钥

```text
$ gpg --full-generate-key
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥类型：
   (1) RSA 和 RSA （默认）
   (2) DSA 和 Elgamal
   (3) DSA（仅用于签名）
   (4) RSA（仅用于签名）
您的选择是？ 4
RSA 密钥的长度应在 1024 位与 4096 位之间。
您想要使用的密钥长度？(2048) 
请求的密钥长度是 2048 位                
请设定这个密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0) 0
密钥永远不会过期                
这些内容正确吗？ (y/N) y
                                
GnuPG 需要构建用户标识以辨认您的密钥。

真实姓名： dounine
电子邮件地址： xxxxx@gamil.com
注释： for lake                         
您选定了此用户标识：
    “dounine (for lake) <xxxxx@gamil.com>”

更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？ O
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘                                  
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
gpg: 密钥 87FC6DD218033D5F 被标记为绝对信任
gpg: 吊销证书已被存储为‘/Users/huanghuanlai/.gnupg/openpgp-revocs.d/9C70D54B941D7831E4B97C8387FC6DD218033D5F.rev’
公钥和私钥已经生成并被签名。

请注意这个密钥不能用于加密。您可能想要使用“--edit-key”命令来
生成一个用于此用途的子密钥。
pub   rsa2048 2019-01-07 [SC]
      9C70D54B941D7831E4B97C8387FC6DD218033D5F
uid                      dounine (for lake) <xxxxx@gamil.com>
```

上传公钥到两台服务器上 keys.gnupg.net 与 keyserver.ubuntu.com

```text
$ gpg --keyserver keys.gnupg.net --send-keys 9C70D54B941D7831E4B97C8387FC6DD218033D5F
gpg: 正在发送密钥 87FC6DD218033D5F 到 hkp://hkps.pool.sks-keyservers.net
gpg: 发送至公钥服务器失败：No route to host
gpg: 发送至公钥服务器失败：No route to host
# 不要慌，使用IP地扯即可 ping keys.gnupg.net 得到 51.38.91.189 地扯
$ gpg --keyserver 51.38.91.189 --send-keys 9C70D54B941D7831E4B97C8387FC6DD218033D5F
$ gpg --keyserver keyserver.ubuntu.com --send-keys 9C70D54B941D7831E4B97C8387FC6DD218033D5F
```

{% hint style="info" %}
pub 是一个40位的密钥，如果你把它复制上传肯定会报这么一个错

The key ID must be in a valid form ...
{% endhint %}

keyId 其实就是pub最后面8位 或者使用`gpg --list-keys --keyid-format short`查看

~/.gradle/gradle.properties 配置

```text
signing.keyId=18033D5F
signing.password=gpg设置的密码
signing.secretKeyRingFile=/Users/lake/.gnupg/secring.gpg
#gpg 版本如果是2.0以上的话需要手动导出secring.gpg
gpg --export-secret-keys > ~/.gnupg/secring.gpg

NEXUS_USERNAME=NEXUS帐号
NEXUS_PASSWORD=NEXUS密码
NEXUS_EMAIL=email
```

build.gradle 演示案例配置

```text
group = 'com.dounine.scala'
version = '1.0.0'
apply plugin: 'signing'
apply plugin: 'scala'
apply plugin: 'maven-publish'
sourceCompatibility = 1.8
task sourcesJar(type: Jar) {
	from sourceSets.main.allJava
	classifier = 'sources'
}
task javadocJar(type: Jar) {
	from javadoc
	classifier = 'javadoc'
}
publishing {
	publications {
		mavenJava(MavenPublication) {
			artifactId = 'scala-filebeat'
			from components.java
			artifact sourcesJar
			artifact javadocJar
			pom {
				name = 'scala-filebeat'
				description = 'scala version filebeat'
				url = 'https://github.com/dounine/scala-filebeat'
				licenses {
					license {
						name = 'The Apache License, Version 2.0'
						url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
					}
				}
				developers {
					developer {
						id = 'lake'
						name = 'lake'
						email = 'hello@gmail.com'
					}
				}
				scm {
					connection = 'scm:git:git://github.com/dounine/scala-filebeat.git'
					developerConnection = 'scm:git:ssh://github.com/dounine/scala-filebeat.git'
					url = 'https://github.com/dounine/scala-filebeat'
				}
			}
		}
	}
	repositories {
		maven {
			def releasesRepoUrl = "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
			def snapshotsRepoUrl = "https://oss.sonatype.org/content/repositories/snapshots/"
			url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
			credentials {
				username NEXUS_USERNAME
				password NEXUS_PASSWORD
			}
		}
	}
}
jar {
	manifest {
		attributes 'Implementation-Title': 'ScalaFilebeat',
				'Implementation-Version': version
	}
}
signing {
	sign publishing.publications.mavenJava
}
repositories {
	mavenLocal()
	mavenCentral()
}
dependencies {
	compile 'org.scala-lang:scala-library:2.11.12'
	compile group: 'commons-io', name: 'commons-io', version: '2.6'
}
```

打包

```text
gradle clean build -xtest -Ppro
```

发布到NEXUS仓库

```text
gradle publish
```

最后一步

登录 [NEXUS仓库](https://oss.sonatype.org/#stagingRepositories) 找到自己上传的项目，点击close

![](../../../.gitbook/assets/image%20%2842%29.png)

等待close状态全部通过即可发布Release版本

![](../../../.gitbook/assets/image%20%2841%29.png)

