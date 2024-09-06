---
layout: post
title: Kotlin - Coroutines and channels
date: 2024-09-06 14:58 +0900
description: 코루틴을 이용해 네트워크 리퀘스트 처리 실습
image:
category: ["kotlin", "coroutine"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

> [공식문서의 실습](https://kotlinlang.org/docs/coroutines-and-channels.html)을 참고하여 학습하며 작성한 글입니다.

[https://github.com/kotlin-hands-on/intro-coroutines](https://github.com/kotlin-hands-on/intro-coroutines) 레포지토리를 clone하여 실습을 진행했습니다.

![https://kotlinlang.org/docs/images/initial-window.png](https://kotlinlang.org/docs/images/initial-window.png)

clone한 프로젝트를 실행하면 다음과 같은 화면이 나오고, 깃허브 유저네임과 api 토큰을 입력하면 kotlin organization에 contribute한 개발자들 리스트를 확인할 수 있습니다.

## 프로젝트 코드 분석

clone한 프로젝트 구조는 다음과 같습니다.

<img width="265" alt="Screenshot 2024-09-06 at 15 29 46" src="https://github.com/user-attachments/assets/29fed255-f4f6-49ad-9013-c82fd83589cc">

### main.kt

main 파일은 매우 간단하게 구성되어있습니다.
```kotlin
package contributors

fun main() {
    setDefaultFontSize(18f)
    ContributorsUI().apply {
        pack()
        setLocationRelativeTo(null)
        isVisible = true
    }
}
```

`ContributorsUI`를 실행하여 UI를 생성합니다.
```kotlin
class ContributorsUI : JFrame("GitHub Contributors"), Contributors {
  /* ... */

  init {
        // Create UI
        rootPane.contentPane = JPanel(GridBagLayout()).apply {
            addLabeled("GitHub Username", username)
            addLabeled("Password/Token", password)
            addWideSeparator()
            addLabeled("Organization", org)
            addLabeled("Variant", variant)
            addWideSeparator()
            addWide(JPanel().apply {
                add(load)
                add(cancel)
            })
            addWide(resultsScroll) {
                weightx = 1.0
                weighty = 1.0
                fill = GridBagConstraints.BOTH
            }
            addWide(loadingStatus)
        }
        // Initialize actions
        init()
    }

    /* ... *.
}
```

`CotributorsUI`는 초기화단계에서 `Contributors.init()`을 호출하는 것을 확인할 수 있습니다.

```kotlin
interface Contributors: CoroutineScope {

    fun init() {
        // Start a new loading on 'load' click
        addLoadListener {
            saveParams()
            loadContributors()
        }

        // Save preferences and exit on closing the window
        addOnWindowClosingListener {
            job.cancel()
            saveParams()
            exitProcess(0)
        }

        // Load stored params (user & password values)
        loadInitialParams()
    }
}
```

`addLoadListener` 함수에 람다 표현식을 전달해 `load` 버튼이 눌렸을 때, `saveParams()` 와 `loadContributors`가 실행되게 하는 것을 확인할 수 있습니다.

```kotlin
    fun saveParams() {
        val params = getParams()
        if (params.username.isEmpty() && params.password.isEmpty()) {
            removeStoredParams()
        }
        else {
            saveParams(params)
        }
    }

    fun loadContributors() {
        val (username, password, org, _) = getParams()
        val req = RequestData(username, password, org)

        clearResults()
        val service = createGitHubService(req.username, req.password)

        val startTime = System.currentTimeMillis()
        when (getSelectedVariant()) {
            BLOCKING -> { // Blocking UI thread
                val users = loadContributorsBlocking(service, req)
                updateResults(users, startTime)
            }
            BACKGROUND -> { // Blocking a background thread
                loadContributorsBackground(service, req) { users ->
                    SwingUtilities.invokeLater {
                        updateResults(users, startTime)
                    }
                }
            }

            /* ... */
        }
    }
```

`saveParams` 함수로 사용자가 입력한 파라미터를 저장하고, `loadContributors`함수로 kotlin organization의 contributor를 조회하는 것을 알 수 있습니다. 이때 variant 값으로 로직의 실행 방식을 지정할 수 있습니다.

가능한 varinat 값들은 다음과 같습니다.
```kotlin
    BLOCKING,         // Request1Blocking
    BACKGROUND,       // Request2Background
    CALLBACKS,        // Request3Callbacks
    SUSPEND,          // Request4Coroutine
    CONCURRENT,       // Request5Concurrent
    NOT_CANCELLABLE,  // Request6NotCancellable
    PROGRESS,         // Request6Progress
    CHANNELS          // Request7Channels
```

깃허브로 부터 요청을 보내고 데이터를 받는 기능은 `GithubService`에 구현되어 있습니다.

```kotlin
interface GitHubService {
    @GET("orgs/{org}/repos?per_page=100")
    fun getOrgReposCall(
        @Path("org") org: String
    ): Call<List<Repo>>

    @GET("repos/{owner}/{repo}/contributors?per_page=100")
    fun getRepoContributorsCall(
        @Path("owner") owner: String,
        @Path("repo") repo: String
    ): Call<List<User>>
}
```

`Contributors.loadContributor` 함수에서 `createGithubService`로 `GithubService`객체를 생성하고, 생성된 서비스 객체가 깃허브로부터 repository 정보, 그리고 repository의 contributor 정보를 받아옵니다.

## blocking
 
가장 먼저 blocking 방식으로 코드를 실행해보겠습니다.

blocking 방식으로 동작하는 코드는 다음과 같습니다.

```kotlin
fun loadContributorsBlocking(service: GitHubService, req: RequestData) : List<User> {
    val repos = service
        .getOrgReposCall(req.org)
        .execute() // Executes request and blocks the current thread
        .also { logRepos(req, it) }
        .body() ?: emptyList()

    return repos.flatMap { repo ->
        service
            .getRepoContributorsCall(req.org, repo.name)
            .execute() // Executes request and blocks the current thread
            .also { logUsers(repo, it) }
            .bodyList()
    }.aggregate()
}

fun <T> Response<List<T>>.bodyList(): List<T> {
    return body() ?: emptyList()
}
```

위 코드는 먼저 주어진 organization의 레포지토리를 `repos` 리스트에 저장합니다. 그리고 각 레포지토리에 대하여 contributor를 조회하는 방식으로 동작합니다.

`getOrgReposCall()` 그리고 `getRepoContributrosCall()` 모두 `*Call` 인스턴스를 리턴합니다. 이시점에서 리퀘스트는 보내지지 않습니다.
`*Call.execute()`가 실행되어 리퀘스트를 전송합니다. `execute()`는 기저 스레드를 차단하는 동기적 방식으로 동작합니다.

`logRepos()` 그리고 `logUsers()`로 호출 결과를 로깅합니다. 

실행 결과는 다음과 같습니다.

<img width="537" alt="Screenshot 2024-09-06 at 15 52 40" src="https://github.com/user-attachments/assets/def29e24-8a0c-4977-b9ec-a53fb94df3be">

```text
189 [AWT-EventQueue-0] INFO  Contributors - Clearing result
3304 [AWT-EventQueue-0] INFO  Contributors - kotlin: loaded 100 repos
3730 [AWT-EventQueue-0] INFO  Contributors - kotlin-eclipse: loaded 30 contributors
4138 [AWT-EventQueue-0] INFO  Contributors - kotlin-examples: loaded 30 contributors
4496 [AWT-EventQueue-0] INFO  Contributors - ts2kt: loaded 11 contributors
4787 [AWT-EventQueue-0] INFO  Contributors - kotlin-koans: loaded 43 contributors
5131 [AWT-EventQueue-0] INFO  Contributors - dokka: loaded 100 contributors
5471 [AWT-EventQueue-0] INFO  Contributors - kotlin-benchmarks: loaded 10 contributors
5881 [AWT-EventQueue-0] INFO  Contributors - anko: loaded 69 contributors
6289 [AWT-EventQueue-0] INFO  Contributors - kotlinx.html: loaded 32 contributors
6697 [AWT-EventQueue-0] INFO  Contributors - anko-example: loaded 2 contributors
7108 [AWT-EventQueue-0] INFO  Contributors - kotlin-spec: loaded 23 contributors
7518 [AWT-EventQueue-0] INFO  Contributors - kotlinx.reflect.lite: loaded 11 contributors
7926 [AWT-EventQueue-0] INFO  Contributors - obsolete-kotlin-jdbc: loaded 12 contributors
8334 [AWT-EventQueue-0] INFO  Contributors - obsolete-kotlin-swing: loaded 8 contributors
8746 [AWT-EventQueue-0] INFO  Contributors - kotlinx.dom: loaded 5 contributors
9142 [AWT-EventQueue-0] INFO  Contributors - kotlin-koans-edu-obsolete: loaded 5 contributors
9487 [AWT-EventQueue-0] INFO  Contributors - KEEP: loaded 65 contributors
9772 [AWT-EventQueue-0] INFO  Contributors - kotlinx.support: loaded 1 contributors
10077 [AWT-EventQueue-0] INFO  Contributors - coroutines-examples: loaded 21 contributors
10487 [AWT-EventQueue-0] INFO  Contributors - kotlinx.collections.immutable: loaded 19 contributors
10793 [AWT-EventQueue-0] INFO  Contributors - kotlin-style-guide: loaded 2 contributors
11206 [AWT-EventQueue-0] INFO  Contributors - kotlinx.coroutines: loaded 100 contributors
11512 [AWT-EventQueue-0] INFO  Contributors - kotlin-jupyter: loaded 33 contributors
11815 [AWT-EventQueue-0] INFO  Contributors - kotlin-frontend-plugin: loaded 20 contributors
12227 [AWT-EventQueue-0] INFO  Contributors - workshop: loaded 8 contributors
12531 [AWT-EventQueue-0] INFO  Contributors - kotlin-fullstack-sample: loaded 10 contributors
12822 [AWT-EventQueue-0] INFO  Contributors - kotlin-interactive-shell: loaded 14 contributors
13149 [AWT-EventQueue-0] INFO  Contributors - kotlin-in-action: loaded 1 contributors
13561 [AWT-EventQueue-0] INFO  Contributors - kotlin-by-example: loaded 83 contributors
13972 [AWT-EventQueue-0] INFO  Contributors - kotlinx.serialization: loaded 100 contributors
14276 [AWT-EventQueue-0] INFO  Contributors - kotlinx-atomicfu: loaded 37 contributors
14583 [AWT-EventQueue-0] INFO  Contributors - kotlinconf-spinner: loaded 14 contributors
14992 [AWT-EventQueue-0] INFO  Contributors - kotlinx-cli: loaded 22 contributors
15403 [AWT-EventQueue-0] INFO  Contributors - kotlinx-io: loaded 22 contributors
15811 [AWT-EventQueue-0] INFO  Contributors - js-externals: loaded 5 contributors
16221 [AWT-EventQueue-0] INFO  Contributors - kotlin-playground-wp-plugin: loaded 7 contributors
16528 [AWT-EventQueue-0] INFO  Contributors - kotlin-native-calculator-sample: loaded 3 contributors
16837 [AWT-EventQueue-0] INFO  Contributors - kmp-basic-sample: loaded 16 contributors
17245 [AWT-EventQueue-0] INFO  Contributors - dukat: loaded 16 contributors
17657 [AWT-EventQueue-0] INFO  Contributors - kotlinx-benchmark: loaded 37 contributors
18064 [AWT-EventQueue-0] INFO  Contributors - website-grammar-generator: loaded 5 contributors
18372 [AWT-EventQueue-0] INFO  Contributors - grammar-tools: loaded 6 contributors
18679 [AWT-EventQueue-0] INFO  Contributors - kotlinx.team.infra: loaded 7 contributors
18985 [AWT-EventQueue-0] INFO  Contributors - xcode-compat: loaded 6 contributors
19397 [AWT-EventQueue-0] INFO  Contributors - web-site-samples: loaded 1 contributors
19746 [AWT-EventQueue-0] INFO  Contributors - io2019-serverside-demo: loaded 4 contributors
20012 [AWT-EventQueue-0] INFO  Contributors - coroutines-workshop: loaded 1 contributors
20420 [AWT-EventQueue-0] INFO  Contributors - kotlin-koans-edu: loaded 21 contributors
20726 [AWT-EventQueue-0] INFO  Contributors - kotlin-js-demo-1.3.50: loaded 1 contributors
21032 [AWT-EventQueue-0] INFO  Contributors - kotlinx-datetime: loaded 27 contributors
21446 [AWT-EventQueue-0] INFO  Contributors - kotlin-numpy: loaded 2 contributors
21748 [AWT-EventQueue-0] INFO  Contributors - kotlinx-browser: loaded 2 contributors
22162 [AWT-EventQueue-0] INFO  Contributors - kotlinx-knit: loaded 9 contributors
22570 [AWT-EventQueue-0] INFO  Contributors - binary-compatibility-validator: loaded 28 contributors
22979 [AWT-EventQueue-0] INFO  Contributors - kotlin-script-examples: loaded 10 contributors
23389 [AWT-EventQueue-0] INFO  Contributors - full-stack-web-jetbrains-night-sample: loaded 9 contributors
23800 [AWT-EventQueue-0] INFO  Contributors - kotlindl: loaded 26 contributors
24105 [AWT-EventQueue-0] INFO  Contributors - kotlinx-nodejs: loaded 5 contributors
24370 [AWT-EventQueue-0] INFO  Contributors - dataframe: loaded 32 contributors
24823 [AWT-EventQueue-0] INFO  Contributors - multik: loaded 16 contributors
25233 [AWT-EventQueue-0] INFO  Contributors - kotlin-spark-api: loaded 17 contributors
25642 [AWT-EventQueue-0] INFO  Contributors - kotlin-spark-shell: loaded 4 contributors
25949 [AWT-EventQueue-0] INFO  Contributors - kmp-with-cocoapods-sample: loaded 3 contributors
26358 [AWT-EventQueue-0] INFO  Contributors - kmp-with-cocoapods-xcode-two-kotlin-libraries-sample: loaded 4 contributors
26672 [AWT-EventQueue-0] INFO  Contributors - kmp-with-cocoapods-multitarget-xcode-sample: loaded 4 contributors
26977 [AWT-EventQueue-0] INFO  Contributors - kmp-production-sample: loaded 12 contributors
27391 [AWT-EventQueue-0] INFO  Contributors - dokka-plugin-template: loaded 7 contributors
27693 [AWT-EventQueue-0] INFO  Contributors - kmp-integration-sample: loaded 8 contributors
28215 [AWT-EventQueue-0] INFO  Contributors - kotlin-grammar-gpl2: loaded 5 contributors
28511 [AWT-EventQueue-0] INFO  Contributors - kotlin-libs-publisher: loaded 1 contributors
28818 [AWT-EventQueue-0] INFO  Contributors - react-redux-js-ir-todo-list-sample: loaded 2 contributors
29125 [AWT-EventQueue-0] INFO  Contributors - kotlin-js-inspection-pack-plugin: loaded 3 contributors
29405 [AWT-EventQueue-0] INFO  Contributors - kotlin-jupyter-libraries: loaded 27 contributors
29662 [AWT-EventQueue-0] INFO  Contributors - kotlinx-kover: loaded 19 contributors
29928 [AWT-EventQueue-0] INFO  Contributors - full-stack-spring-collaborative-todo-list-sample: loaded 5 contributors
30356 [AWT-EventQueue-0] INFO  Contributors - kdoctor: loaded 8 contributors
30764 [AWT-EventQueue-0] INFO  Contributors - kotlin-build-report-sample: loaded 2 contributors
31073 [AWT-EventQueue-0] INFO  Contributors - kandy: loaded 4 contributors
31377 [AWT-EventQueue-0] INFO  Contributors - kotlin-in-action-2e: loaded 2 contributors
31787 [AWT-EventQueue-0] INFO  Contributors - kotlindl-app-sample: loaded 3 contributors
32192 [AWT-EventQueue-0] INFO  Contributors - kotlin-wasm-examples: loaded 11 contributors
32606 [AWT-EventQueue-0] INFO  Contributors - kotlin-wasm-benchmarks: loaded 3 contributors
32912 [AWT-EventQueue-0] INFO  Contributors - api-guidelines: loaded 12 contributors
33323 [AWT-EventQueue-0] INFO  Contributors - kotlin-in-action-2e-jkid: loaded 1 contributors
33631 [AWT-EventQueue-0] INFO  Contributors - community-project-gradle-plugin: loaded 4 contributors
34039 [AWT-EventQueue-0] INFO  Contributors - multiplatform-library-template: loaded 8 contributors
34347 [AWT-EventQueue-0] INFO  Contributors - KMP-App-Template: loaded 4 contributors
34759 [AWT-EventQueue-0] INFO  Contributors - KMP-App-Template-Native: loaded 4 contributors
35163 [AWT-EventQueue-0] INFO  Contributors - kmp-native-wizard: loaded 2 contributors
35474 [AWT-EventQueue-0] INFO  Contributors - kotlinx-rpc: loaded 11 contributors
35995 [AWT-EventQueue-0] INFO  Contributors - llvm-project: loaded 100 contributors
36290 [AWT-EventQueue-0] INFO  Contributors - kotlin-wasm-browser-template: loaded 2 contributors
36599 [AWT-EventQueue-0] INFO  Contributors - kotlin-wasm-nodejs-template: loaded 2 contributors
36906 [AWT-EventQueue-0] INFO  Contributors - kotlin-wasm-wasi-template: loaded 2 contributors
37213 [AWT-EventQueue-0] INFO  Contributors - kotlin-wasm-compose-template: loaded 2 contributors
37624 [AWT-EventQueue-0] INFO  Contributors - swift-export-sample: loaded 3 contributors
37933 [AWT-EventQueue-0] INFO  Contributors - JsonToKotlinClass: loaded 29 contributors
38238 [AWT-EventQueue-0] INFO  Contributors - kotlin-jupyter-http-util: loaded 1 contributors
38546 [AWT-EventQueue-0] INFO  Contributors - k2-performance-metrics: loaded 10 contributors
38952 [AWT-EventQueue-0] INFO  Contributors - kotlin.github.io: loaded 2 contributors
39262 [AWT-EventQueue-0] INFO  Contributors - Storytale: loaded 4 contributors
39263 [AWT-EventQueue-0] INFO  Contributors - Updating result with 1622 rows
```

로그 실행 결과로부터 모든 로그가 같은 스레드에서 찍혔음을 알 수 있습니다. `BLOCKING` 옵션으로 코드를 실행하면, UI는 로딩이 종료될 때까지 사용자 입력에 반응하지 않습니다. 모든 요청들은 `loadContributorsBlocking()`을 호출한 스레드에서 실행됩니다. 이경우에는 `loadContributorsBlocking()`을 호출한 스레드와 UI를 실행한 스레드가 동일하기에, UI가 아무런 동작을 하지 않게 됩니다.

![https://kotlinlang.org/docs/images/blocking.png](https://kotlinlang.org/docs/images/blocking.png)

`loadContributorsBlocking()`의 실행이 끝나고 나서야 실행결과가 UI창에도 업데이트 됩니다.

```kotlin
BLOCKING -> { // Blocking UI thread
                val users = loadContributorsBlocking(service, req)
                updateResults(users, startTime)
            }
```

`updateResults()` 가 실행되어야 UI가 업데이트되는데, `loadContributorsBlocking`의 실행이 끝나야 `updateResults`가 실행되기에, 늦게 업데이트 되는 것입니다.

## callbacks

blocking 방식으로 동작해도 실행이 되긴하지만, UI가 반응하지 않는 문제가 있습니다. 이것을 피하는 전통적인 방식 중 하나는 콜백을 사용하는 것입니다.

어떤 작업이 끝난 이후에 실행되야할 코드를 호출하기보단, 실행되야할 코드를 콜백으로 분리하고, 이것을 람다로 전달하여 이후에 호출되게 할 수 있습니다.

UI가 반응하게 만들기 위해서는 작업 전체를 다른 스레드가 옮기거나, `RetrofitAPI`를 콜백을 사용하도록 변경할 수 있습니다.

### background thread

`Request2Background`파일을 확인해보면 다음과 같습니다. `loadContributorsBlocking`을 다른 스레드에서 호출하는 것을 확인할 수 있습니다.

```kotlin
fun loadContributorsBackground(service: GitHubService, req: RequestData, updateResults: (List<User>) -> Unit) {
    thread {
        loadContributorsBlocking(service, req)
    }
}
```

`thread` 함수는 새로운 스레드를 시작합니다.
![https://kotlinlang.org/docs/images/background.png](https://kotlinlang.org/docs/images/background.png)
로딩이 다른 스레드로 옮겨졌기에, 메인 스레드는 다른 작업을 수행할 수 있습니다.
`loadContributorsBackground` 함수 시그니처를 확인해보면, `updateResults()`를 콜백 함수로 전달 받는 것을 알 수 있습니다.

```kotlin
loadContributorsBackground(service, req) { users ->
    SwingUtilities.invokeLater {
        updateResults(users, startTime)
    }
}
```
`loadContributorsBackground` 함수를 호출하고 `updateResults`를 직후에 호출하는 것이 아니라, 콜백 함수로 인자로 넘겨주고 있음을 확인할 수 있습니다.

`BACKGROUND`옵션으로 실행해보면, `BLOCKING`으로 실행했을 때와는 다르게 UI창에서 Loading되는 것을 확인할 수 있습니다.

## Retrofit callback

이전 솔루션에서 모든 로딩 로직을 background thread로 옮겼지만, 아직 리소스를 최적으로 쓰고 있지 않습니다. 모든 로딩 요청들은 순차적으로 처리되고, 스레드는 요청의 결과를 기다리는 동안 block 됩니다. 스레드가 요청의 결과를 기다리는 동안 block 되기에 리소스를 최적으로 쓰고 있지 않는 것입니다. 요청의 결과를 받기 전에 새로운 요청을 보낼 수도 있기 때문입니다.

각 레포지토리의 데이터를 핸들링하는 과정은 2단계로 나눌 수 있습니다. 
- loading and processing the resulting response

결과를 처리하는 과정은 콜백으로 빠질 수 있습니다.

각 레포지토리에 요청을 보내는 것은 이전 레포지토리 요청에 대한 결과를 받기 전에 시작될 수 있습니다.

![https://kotlinlang.org/docs/images/callbacks.png](https://kotlinlang.org/docs/images/callbacks.png)

Retrofit callback API는 이것을 가능하게 해줍니다. `Call.enqueue()` 함수는 http 리퀘스트를 시작하고 콜백을 인자로 받습니다. 콜백에서 각 요청이 끝난 이후에 어떤 작업을 처리해야하는지 명시해야합니다.

```kotlin
fun loadContributorsCallbacks(service: GitHubService, req: RequestData, updateResults: (List<User>) -> Unit) {
    service.getOrgReposCall(req.org).onResponse { responseRepos ->
        logRepos(req, responseRepos)
        val repos = responseRepos.bodyList()
        val allUsers = mutableListOf<User>()
        for (repo in repos) {
            service.getRepoContributorsCall(req.org, repo.name).onResponse { responseUsers ->
                logUsers(repo, responseUsers)
                val users = responseUsers.bodyList()
                allUsers += users
            }
        }
        // TODO: Why this code doesn't work? How to fix that?
        updateResults(allUsers.aggregate())
    }
}

inline fun <T> Call<T>.onResponse(crossinline callback: (Response<T>) -> Unit) {
    enqueue(object : Callback<T> {
        override fun onResponse(call: Call<T>, response: Response<T>) {
            callback(response)
        }

        override fun onFailure(call: Call<T>, t: Throwable) {
            log.error("Call failed", t)
        }
    })
}

```
편의를 위해서 `onResponse`라는 익스텐션 함수를 이용해서 콜백 등록을 처리했습니다.

위 코드를 테스트해보기 위해 `CALLBACK`옵션으로 프로그램을 실행해보면, 동작하지 않습니다.

로그를 확인해보면 다음과 같습니다.
```text
169 [AWT-EventQueue-0] INFO  Contributors - Clearing result
3273 [OkHttp https://api.github.com/...] INFO  Contributors - kotlin: loaded 100 repos
3294 [AWT-EventQueue-0] INFO  Contributors - Clearing result
4216 [OkHttp https://api.github.com/...] INFO  Contributors - ts2kt: loaded 11 contributors
4218 [OkHttp https://api.github.com/...] INFO  Contributors - kotlin-eclipse: loaded 30 contributors
4221 [OkHttp https://api.github.com/...] INFO  Contributors - kotlin-koans: loaded 43 contributors
4223 [OkHttp https://api.github.com/...] INFO  Contributors - kotlin-examples: loaded 30 contributors
....
```

모든 레포지토리에 대하여 contributor를 조회한 후 `updateResults`가 실행되어야하는데, 레포지토리에 대한 조회가 끝난 직후에 바로 `updateResults`가 실행된 모습입니다.

우선 레포지토리를 조회하는 Retrofit 호출도 결과를 콜백을 이용해서 처리되게 작성되었습니다. 
레포지토리를 조회하는 요청을 ui 스레드에서 시작하고 요청에 대한 결과 처리는 다른 스레드에서 실행된 것을 확인할 수 있습니다. 
contributor 조회 요청 역시 콜백을 이용해서 처리하므로, 반복문을 돌면서 각 레포지토리에 contributor 요청을 ui 스레드에서 하게 됩니다. 
모든 요청을 호출한 이후 ui 스레드에서는 blocking되지 않기에 곧 바로 다음 라인 코드를 실행하게 되고, 아직 응답을 받지 못한 상태이기에 빈 결과를 화면에 호출하게 됩니다.

ui 스레드가 콜백 스레드에서 작업을 처리함을 기다려야하기에, 다음과 같이 코드를 수정해보았습니다.
```kotlin
fun loadContributorsCallbacks(service: GitHubService, req: RequestData, updateResults: (List<User>) -> Unit) {
    service.getOrgReposCall(req.org).onResponse { responseRepos ->
        logRepos(req, responseRepos)
        val repos = responseRepos.bodyList()
        val allUsers = mutableListOf<User>()
        val count = AtomicInteger(0)
        for (repo in repos) {
            service.getRepoContributorsCall(req.org, repo.name).onResponse { responseUsers ->
                logUsers(repo, responseUsers)
                val users = responseUsers.bodyList()
                allUsers += users
                count.incrementAndGet()
            }
        }
        // TODO: Why this code doesn't work? How to fix that?
        while (count.get() < repos.size) {
            continue
        }
        updateResults(allUsers.aggregate())
    }
}
```

공식 문서에 적힌 최종 해답은 다음과 같습니다.
```kotlin
val countDownLatch = CountDownLatch(repos.size)
for (repo in repos) {
    service.getRepoContributorsCall(req.org, repo.name)
        .onResponse { responseUsers ->
            // processing repository
            countDownLatch.countDown()
        }
}
countDownLatch.await()
updateResults(allUsers.aggregate())
```

## suspending functions

위와 같은 로직을 suspending function을 이용해서 구현할 수도 있습니다. 
`GithubService`가 `Call<List<Repo>>`를 리턴하는대신, 다음과 같이 `suspending function`으로 정의할 수 있습니다.

```kotlin
interface GitHubService {
    @GET("orgs/{org}/repos?per_page=100")
    suspend fun getOrgRepos(
        @Path("org") org: String
    ): List<Repo>
}
```

- `getOrgRepos()`는 `suspend`함수로 정의됐습니다. `suspending function`으로 리퀘스트를 실행하면, underlying thread는 block되지 않습니다. 
- `getOrgRepos()`는 `Call`이 아닌 실제 결과를 리턴합니다. 만약 성공적이지 못했다면, 예외가 발생합니다.

Retrofit은 호출 결과를 `Response`로 감싸서 리턴할 수 있습니다. 이 경우, `Response`객체를 통해 에러가 발생했는지 여부를 확인가능합니다.

## coroutines

suspending function을 이용한 코드는 blocking 방식과 매우 유사합니다.

```kotlin
    @GET("orgs/{org}/repos?per_page=100")
    suspend fun getOrgRepos(
        @Path("org") org: String
    ): Response<List<Repo>>

    @GET("repos/{owner}/{repo}/contributors?per_page=100")
    suspend fun getRepoContributors(
        @Path("owner") owner: String,
        @Path("repo") repo: String
    ): Response<List<User>>

    suspend fun loadContributorsSuspend(service: GitHubService, req: RequestData): List<User> {
      val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .body() ?: emptyList()

      return repos.flatMap { repo ->
          service
            .getRepoContributors(req.org, repo.name)
            .also { logUsers(repo, it) }
            .bodyList()
      }.aggregate()
    }
```

blocking 방식과 가장 큰 차이점은 thread가 block되지 않고, coroutine이 suspend된다는 것입니다.
```text
block -> suspend
thread -> coroutine
```

### 새로운 coroutine 실행하기

`Contributors`에서는 `loadContributorsSuspend()`를 다음과 같이 `launch`함수를 이용해서 호출하고 있습니다.
```kotlin
launch {
    val users = loadContributorsSuspend(req)
    updateResults(users, startTime)
}
```

`launch` 함수는 데이터를 로딩하고, 결과를 보여주는 책임을 가진 새로운 작업을 실행합니다. 작업은 중단 가능합니다. 네트워크 요청을 전송한 후, 작업은 잠시 중단됩니다.
요청에 대한 응답이 돌아오면 작업은 계속 진행됩니다.

이런 중단 가능한 작업을 `coroutine`이라 부릅니다. 위 경우에는 `launch`함수로 새로운 코루틴을 실행해 데이터를 로딩하고 작업의 결과를 출력합니다.

코루틴은 스레드위에서 실행되고, 중단가능합니다. 코루틴이 중단되면, 코루틴에서 진행하던 작업은 중지 되고, 스레드에서 제거되고, 메모리에 저장됩니다. 스레드는 그럼 다른 작업들을 이어서 진행 할 수 있습니다.

![https://kotlinlang.org/docs/images/suspension-process.gif](https://kotlinlang.org/docs/images/suspension-process.gif)

중단된 코루틴이 다시금 이어서 작업을 진행할 수 있는 상태가 되면, 스레드로 돌아갑니다. (꼭 작업이 중단되었던 스레드로 돌아가지 않아도 됩니다.)

`loadContributorsSuspend`의 예시에서, 각 contributors 조회 요청은 suspension 방식으로 실행됩니다. 첫번째 요청이 전송되고 요청에 대한 응답을 대기할 때 해당 코루틴은 중단되게 됩니다.
중단된 요청에 대한 응답이 돌아오면, 코루틴은 작업을 이어서 진행합니다.

![https://kotlinlang.org/docs/images/suspend-requests.png](https://kotlinlang.org/docs/images/suspend-requests.png)

요청에 대한 응답을 기다리는 동안 스레드는 다른 작업들을 처리할 수 있습니다. UI스레드에서 요청 작업들을 처리하면서도 반응 가능한 UI를 제공합니다.

## concurrency

코틀린 코루틴은 스레드보다 더 적은 리소스를 사용합니다. 어떤 작업을 비동기적으로 실행할고 싶을 때, 스레드를 새로 생성하는 대신 코루틴을 새로 생성하는 것이 좋습니다.

새로운 코루틴을 시작하려면,  **coroutine builder** 중 하나를 사용하는 것이 좋습니다. `launch`, `async`, `runBlocking`

`async`는 새로운 코루틴을 시작하고 `Deferred` 오브젝트를 리턴합니다. `Deferred`는 `Future`, `Promise`같은 이름으로 알려진 개념의 오브젝트입니다. 
It stores a computation, but it **defers** the moment you get the final result; it **promises** the result sometime in the **future**

`async`와 `launch`의 차이점은 `launch`는 특정 결과를 리턴할 것이라고 기대되지 않는 연산을 실행할 때 사용합니다. `launch`는 `Job`을 리턴하는데, `Job.join()`을 호출해 해당 코루틴이 종료될때까지 기다릴 수 있습니다.

`Deferred`는 `Job`을 확장한 제네릭 타입입니다. `async`는 `Deferred<Int>`를 리턴하거나 `Deferred<CustomType>`을 리턴할 수 있습니다. 

코루틴의 결과를 받기 위해서는 `await()`을 호출할 수 있습니다. 결과를 기다리는 동안, 코루틴은 중단됩니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val deferred: Deferred<Int> = async {
        loadData()
    }
    println("waiting...")
    println(deferred.await())
}

suspend fun loadData(): Int {
    println("loading...")
    delay(1000L)
    println("loaded!")
    return 42
}
```

`runBlocking`은 일반 함수와 suspending 함수를 연결하는 함수로 주로 쓰입니다. 주로 `main()`함수에 쓰이거나 테스트에 쓰이도록 만들어졌습니다.

만약 deferred 오브젝트의 리스트가 존재한다면, `awaitAll()`을 호출해 모든 결과를 기다릴 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val deferreds: List<Deferred<Int>> = (1..3).map {
        async {
            delay(1000L * it)
            println("Loading $it")
            it
        }
    }
    val sum = deferreds.awaitAll().sum()
    println("$sum")
}
```

각 contributor 조회 요청이 새로운 코루틴에서 실행되면, 모든 요청은 비동기적으로 시작됩니다. 전송한 요청에 대한 결과를 받기 전에, 새로운 요청이 실행될 수 있습니다. 
![https://kotlinlang.org/docs/images/concurrency.png](https://kotlinlang.org/docs/images/concurrency.png)

CALLBACK 방식으로 동작했을 때와 총 실행 시간은 비슷합니다. 하지만 아무런 콜백도 필요하지 않습니다.


