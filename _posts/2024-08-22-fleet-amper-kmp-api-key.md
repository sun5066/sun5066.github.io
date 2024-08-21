---
layout: post
title: "Kotlin Multiplatform에서의 API 키 관리"
date: 2024-08-22 09:20:00 -0400 
categories: kmp
tags: kmp api key
comments: 1
---

Kotlin Multiplatform 프로젝트(KMP)를 개발하면서 API 키와 같은 민감한 정보를 안전하게 관리하는 것은 필수적입니다. 일반적으로 Android 프로젝트에서는 local.properties 파일에 API 키를 저장하고 Gradle을 통해 BuildConfig에 주입하여 사용하는 방법을 많이 사용합니다. 그러나 저는 Fleet IDE + Amper 를 사용한 KMP 개발을 하고있기 때문에 모든 플랫폼에서 공통적으로 API 키를 관리하고 사용하기 위한 방법이 필요합니다.

이번 포스팅에서는 기존 local.properties에 키를 저장해서 사용하는 방식대로 Gradle Task를 사용하여 API 키를 한 곳에서 관리하고, 여러 플랫폼에서 쉽게 접근할 수 있도록 Kotlin 파일을 자동으로 생성하는 방법을 소개하겠습니다.

## 1.local.properties에 API 키 저장

우선, API 키를 local.properties 파일에 저장합니다. 이 파일은 프로젝트의 루트 디렉토리에 위치하며, Git 저장소에 포함되지 않도록 .gitignore에 추가해야 합니다.

```
// local.properties
api_key=asdasdasdasd
```

이 파일에 API 키를 저장하고, Gradle 스크립트를 통해 API 키를 불러와서 사용할 수 있습니다.

## 2. Gradle Task 생성

이제 build.gradle.kts 파일에서 API 키를 읽어와 Kotlin 파일을 생성하는 커스텀 Task를 정의합니다. 이 Task는 local.properties 파일에서 API 키를 가져와 특정 위치에 ApiKeyConfig.kt 파일을 생성합니다.

먼저, gradle 파일이 동작하기 위해 root/source 라는 jvm 라이브러리 모듈을 생성합니다.

```
// source/module.yaml
product:
    type: lib
    platforms:
        - jvm
```

그 다음 custom task를 작성하기 위한 gradle 파일을 하나 생성해줍니다.
```
// source/build.gradle.kts
import java.util.*

val Project.localProperties: Properties
    get() = Properties().apply {
        val localPropertiesFile = rootProject.file("local.properties")

        if (localPropertiesFile.exists()) {
            this.load(localPropertiesFile.inputStream())
        }
    }


tasks.register("generateApiKey") {
    doLast {
        val apiKey = localProperties["api_key"]?.toString() ?: "No API Key"
        
        val outputDir = file("${rootProject.projectDir}/shared/src/config")
        val apiKeyFile = File(outputDir, "ApiKeyConfig.kt")

        if (!outputDir.exists()) {
            outputDir.mkdirs()
        }

        apiKeyFile.writeText("""
            package com.example.config
            
            object ApiKeyConfig {
                const val API_KEY = "$apiKey"
            }
        """.trimIndent())
        
        println("ApiKeyConfig.kt 파일이 ${apiKeyFile.absolutePath}에 생성되었습니다.")
    }
}
```

## 3. run.json을 통해 Task 실행

JetBrains Fleet 또는 CI/CD 환경에서 이 Task를 실행하기 위해 run.json을 구성할 수 있습니다.

```
// .fleet/run.json
{
  "configurations": [
    {
        "type": "gradle",
        "name": "Generate API Key",
        "tasks": ["generateApiKey"],
    }
  ],
  ...
}
```

## 4. 생성된 Kotlin 파일 사용

이제 KMP 프로젝트의 공통 모듈에서 ApiKeyProvider 객체를 사용하여 API 키에 접근할 수 있습니다. 생성된 ApiKeyProvider.kt 파일은 다음과 같은 구조를 가집니다

```
package com.example.config

object ApiKeyConfig {
    const val API_KEY = "asdasdasdasd"
}
```

이제 Kotlin 코드에서 아래와 같이 API 키를 사용할 수 있습니다.

```
fun getArticles() = intent {
    reduce { state.copy(isLoading = true) }

    try {
        val articles = articleRepository.getArticles(
            country = "us",
            category = "business",
            apiKey = ApiKeyConfig.NEWS_API_KEY
        )
        reduce { state.copy(articles = articles, isLoading = false) }
    } catch (e: Exception) {
        reduce { state.copy(isLoading = false, error = e.message) }
    }
}
```

이번 포스팅에서는 Fleet IDE와 Amper를 사용한 Kotlin Multiplatform 프로젝트에서 API 키를 안전하게 관리하고, 여러 플랫폼에서 공통으로 사용할 수 있도록 자동으로 Kotlin 파일을 생성하는 방법을 소개했습니다. local.properties 파일을 통해 민감한 정보를 관리하고, 커스텀 Gradle Task를 사용하여 API 키를 쉽게 주입할 수 있었습니다.

이 방식을 통해 기존 안드로이드에서 사용했던 BuildConfig 방식과 유사하게 사용할 수 있었습니다.