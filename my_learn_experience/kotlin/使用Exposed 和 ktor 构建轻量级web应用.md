



## 使用`Exposed` 和 `ktor` 构建轻量级web应用	

`exposed`是`jetbrains` 一位成员业余作品。

`ktor` 是一个 非常组件化的一个web framework，它的所有功能都称为feature。

使用它们来构建应用程序非常轻量级，接下来我会用尽可能简单的方法来写一个`hello world`。

就像 `springboot` 有 `start.spring.io`，它也有 `start.ktor.io`，不过目前好像还没有 `ktor-cli`，组合后的`url`为(当前`ktor` 稳定版本在 `1.2.6`,为了展示其web能力，这里引入了之后并不使用的`jackson feature`)

>  https://start.ktor.io/#dependency=html-dsl&dependency=auth-jwt&dependency=ktor-jackson&project-type=gradle-kotlin-dsl&artifact-group=ink.rubi&artifact-name&artifact-version=0.0.1 

打开`build.gradle.kts` 添加国内源

```kotlin
repositories {
    maven { setUrl("http://maven.aliyun.com/nexus/content/groups/public/") }
    mavenLocal()
    jcenter()
    maven { url = uri("https://kotlin.bintray.com/ktor") }
}
```

一顿构建 ，`gradle`遂报错

```
ktor-demo: sync failed  
Can't find resource for bundle java.util.PropertyResourceBundle, key kotlin.gradle.testing.enabled
```

一顿谷歌，主要是因为 `idea` 的 bug，解决方案是

```
https://youtrack.jetbrains.com/issue/KT-33571?p=IDEA-221487
大概就是 在 <idea_installation_path>/lib/util.jar/misc/registry.properties
中加一行 kotlin.gradle.testing.enabled=false
```

在新版本的`idea`中并没有这个问题。

直接运行`Application.kt`中的 main方法，可以看到`ktor`在8080端口上启动了服务。日志大概如下

```plain
2019-12-02 11:24:21.663 [main] TRACE Application - {
    # application.conf @ file:/C:/Users/Administrator/Desktop/a/ktor-demo/out/production/resources/application.conf: 6
    "application" : {
        # application.conf @ file:/C:/Users/Administrator/Desktop/a/ktor-demo/out/production/resources/application.conf: 7
        "modules" : [
            # application.conf @ file:/C:/Users/Administrator/Desktop/a/ktor-demo/out/production/resources/application.conf: 7
            "ink.rubi.ApplicationKt.module"
        ]
    },
    # application.conf @ file:/C:/Users/Administrator/Desktop/a/ktor-demo/out/production/resources/application.conf: 2
    "deployment" : {
        # application.conf @ file:/C:/Users/Administrator/Desktop/a/ktor-demo/out/production/resources/application.conf: 3
        "port" : 8080
    },
    # Content hidden
    "security" : "***"
}

2019-12-02 11:24:22.741 [main] INFO  Application - No ktor.deployment.watch patterns specified, automatic reload is not active
2019-12-02 11:24:23.976 [main] INFO  Application - Responding at http://0.0.0.0:8080
```

现在你可以访问

```
http://localhost:8080/
http://localhost:8080/html-dsl
http://localhost:8080/json/jackson
```

这是默认构建的 `api`，接着我们需要来添加功能实现 `JWT` 认证的登录服务。因为不想引入`js`，我选择了 将 `token`通过响应头写入`cookie`的做法。

```kotlin
package ink.rubi

import com.auth0.jwt.*
import com.auth0.jwt.algorithms.Algorithm
import com.fasterxml.jackson.databind.SerializationFeature
import io.ktor.application.Application
import io.ktor.application.call
import io.ktor.application.install
import io.ktor.auth.*
import io.ktor.auth.jwt.*
import io.ktor.features.ContentNegotiation
import io.ktor.html.respondHtml
import io.ktor.http.Cookie
import io.ktor.http.auth.parseAuthorizationHeader
import io.ktor.jackson.jackson
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.util.date.GMTDate
import kotlinx.html.*
import java.util.*
import kotlin.time.ExperimentalTime
import kotlin.time.seconds

const val jwtAuth = "jwt-auth"
const val formAuth = "form-auth"
const val COOKIE_JWT_KEY_NAME = "JWT"
const val TOKEN_AUTH_SCHEMES = "token"
const val JWT_SECRET = "asdkjkajsdkjqwne"
const val INPUT_USERNAME = "username"
const val INPUT_PASSWORD = "password"
const val ISSUER = "ktor-demo"

fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

@ExperimentalTime
@Suppress("unused") // Referenced in application.conf
@kotlin.jvm.JvmOverloads
fun Application.module(testing: Boolean = false) {
    install(Authentication) {
        jwt(name = jwtAuth) {
            authHeader { call ->
                call.request.cookies[COOKIE_JWT_KEY_NAME]?.let { parseAuthorizationHeader(it) }
            }
            authSchemes(TOKEN_AUTH_SCHEMES)
            verifier(JWTUtil.provideVerifier(ISSUER))

            validate { credential ->
                with(credential.payload) {
                    if (subject == null) return@validate null
                }
                return@validate JWTPrincipal(credential.payload)
            }
            challenge { _, _ ->
                call.respond("token核验不通过!!")
            }
        }
        form(name = formAuth) {
            userParamName = INPUT_USERNAME
            passwordParamName = INPUT_PASSWORD

            validate { credentials ->
                return@validate UserService.getUserByName(credentials.name)?.let {
                    if (it.password == credentials.password) {
                        return@let UserIdPrincipal(it.username)
                    }
                    null
                }
            }
            challenge {
                call.respondText { "账号或者密码错误" }
            }
        }
    }

    install(ContentNegotiation) {
        jackson {
            enable(SerializationFeature.INDENT_OUTPUT)
        }
    }

    routing {
        authenticate(formAuth) {
            post("/login") {
                val name = call.authentication.principal<UserIdPrincipal>()!!.name
                val token = JWTUtil.createToken(ISSUER, name)
                call.response.cookies.append(
                    Cookie(
                        name = COOKIE_JWT_KEY_NAME, value = "$TOKEN_AUTH_SCHEMES $token",
                        expires = GMTDate(JWTUtil.expiredAt + System.currentTimeMillis()),
                        httpOnly = true
                    )
                )
                call.respondRedirect("/test.html")
            }
        }
        get("/login.html") {
            call.respondHtml {
                head { unsafe { +"登陆页面" } }
                body {
                    h1 {
                        +"登录"
                    }
                    form {
                        method = FormMethod.post;action = "http://localhost:8080/login"
                        encType = FormEncType.multipartFormData
                        textInput { name = INPUT_USERNAME;placeholder = "请输入账号" }
                        br { }
                        passwordInput { name = INPUT_PASSWORD;placeholder = "请输入密码" }
                        br { }
                        submitInput { value = "登录" }
                    }
                }
            }
        }
        authenticate(jwtAuth) {
            get("/test.html") {
                call.respondHtml {
                    head { unsafe { +"登陆成功" } }
                    body {
                        h1 { +"登录" }
                        span { +"您好,${call.authentication.principal<JWTPrincipal>()!!.payload.subject}" }
                    }
                }
            }
        }
    }
}

@ExperimentalTime
object JWTUtil {
    private val algorithm = Algorithm.HMAC512(JWT_SECRET)!!
    val expiredAt = 10.seconds.inMilliseconds.toLong()

    fun provideVerifier(jwtIssuer: String): JWTVerifier = JWT.require(algorithm).withIssuer(jwtIssuer).build()

    fun createToken(issuer: String, name: String): String =
        JWT.create().withIssuer(issuer)
            .withSubject(name)
            .withExpiresAt(Date(System.currentTimeMillis() + expiredAt))
            .sign(algorithm)
}

object UserService {
    private val users = listOf(User("rubi", "123"), User("test", "123"))

    fun getUserByName(name: String): User? {
        return users.firstOrNull { name == it.username }
    }
}

data class User(val username: String, val password: String)
```

这里并未使用文档给出的官方文档给出的 `UserHashedTableAuth`，为了之后直接修改`UserService`实现接入数据源。

现在已经可以跑了，使用以下用例登录可以发现刚开始登录成功。

| 账号   | 密码  |
| ------ | ----- |
| `rubi` | `123` |
| `test` | `123` |

但是在 成功页面 经过约 10秒，再次刷新页面时，显示“token核验不通过”。

> ```kotlin
> val expiredAt = 10.seconds.inMilliseconds.toLong()
> ```
>
> 这里指定了token的过期时间



## 引入`Exposed`

`build.gradle.kts`

```kotlin
val exposed_version: String by project
val mysql_driver_version: String by project

dependencies {
    compile("org.jetbrains.exposed:exposed:$exposed_version")
    compile("mysql:mysql-connector-java:$mysql_driver_version")
	//...other dependencies
}

```

`gradle.properties`

```properties
exposed_version=0.17.7
mysql_driver_version=5.1.48
```

修改`Application.kt`为 

```kotlin
package ink.rubi

import com.auth0.jwt.*
import com.auth0.jwt.algorithms.Algorithm
import com.fasterxml.jackson.databind.SerializationFeature
import io.ktor.application.Application
import io.ktor.application.call
import io.ktor.application.install
import io.ktor.auth.*
import io.ktor.auth.jwt.*
import io.ktor.features.ContentNegotiation
import io.ktor.html.respondHtml
import io.ktor.http.Cookie
import io.ktor.http.auth.parseAuthorizationHeader
import io.ktor.jackson.jackson
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.util.date.GMTDate
import kotlinx.html.*
import org.jetbrains.exposed.dao.*
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.transaction
import java.util.Date
import kotlin.time.ExperimentalTime
import kotlin.time.seconds

const val jwtAuth = "jwt-auth"
const val formAuth = "form-auth"
const val COOKIE_JWT_KEY_NAME = "JWT"
const val TOKEN_AUTH_SCHEMES = "token"
const val JWT_SECRET = "asdkjkajsdkjqwne"
const val INPUT_USERNAME = "username"
const val INPUT_PASSWORD = "password"
const val ISSUER = "ktor-demo"

fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

@ExperimentalTime
@Suppress("unused") // Referenced in application.conf
@kotlin.jvm.JvmOverloads
fun Application.module(testing: Boolean = false) {

    Database.connect(
        url = "jdbc:mysql://localhost:3306/ktor_demo?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai",
        user = "root",
        password = "123456",
        driver = "com.mysql.jdbc.Driver"
    )
    transaction { SchemaUtils.createMissingTablesAndColumns(Users) }

    install(Authentication) {
        jwt(name = jwtAuth) {
            authHeader { call ->
                call.request.cookies[COOKIE_JWT_KEY_NAME]?.let { parseAuthorizationHeader(it) }
            }
            authSchemes(TOKEN_AUTH_SCHEMES)
            verifier(JWTUtil.provideVerifier(ISSUER))

            validate { credential ->
                with(credential.payload) {
                    if (subject == null) return@validate null
                }
                return@validate JWTPrincipal(credential.payload)
            }
            challenge { _, _ ->
                call.respond("token核验不通过!!")
            }
        }
        form(name = formAuth) {
            userParamName = INPUT_USERNAME
            passwordParamName = INPUT_PASSWORD

            validate { credentials ->
                return@validate UserService.getUserByName(credentials.name)?.let {
                    if (it.password == credentials.password) {
                        return@let UserIdPrincipal(it.username)
                    }
                    null
                }
            }
            challenge {
                call.respondText { "账号或者密码错误" }
            }
        }
    }

    install(ContentNegotiation) {
        jackson {
            enable(SerializationFeature.INDENT_OUTPUT)
        }
    }

    routing {
        authenticate(formAuth) {
            post("/login") {
                val name = call.authentication.principal<UserIdPrincipal>()!!.name
                val token = JWTUtil.createToken(ISSUER, name)
                call.response.cookies.append(
                    Cookie(
                        name = COOKIE_JWT_KEY_NAME, value = "$TOKEN_AUTH_SCHEMES $token",
                        expires = GMTDate(JWTUtil.expiredAt + System.currentTimeMillis()),
                        httpOnly = true
                    )
                )
                call.respondRedirect("/test.html")
            }
        }
        get("/login.html") {
            call.respondHtml {
                head { unsafe { +"登陆页面" } }
                body {
                    h1 {
                        +"登录"
                    }
                    form {
                        method = FormMethod.post;action = "http://localhost:8080/login"
                        encType = FormEncType.multipartFormData
                        textInput { name = INPUT_USERNAME;placeholder = "请输入账号" }
                        br { }
                        passwordInput { name = INPUT_PASSWORD;placeholder = "请输入密码" }
                        br { }
                        submitInput { value = "登录" }
                    }
                }
            }
        }
        authenticate(jwtAuth) {
            get("/test.html") {
                call.respondHtml {
                    head { unsafe { +"登陆成功" } }
                    body {
                        h1 { +"登录" }
                        span { +"您好,${call.authentication.principal<JWTPrincipal>()!!.payload.subject}" }
                    }
                }
            }
        }
    }
}

@ExperimentalTime
object JWTUtil {
    private val algorithm = Algorithm.HMAC512(JWT_SECRET)!!
    val expiredAt = 10.seconds.inMilliseconds.toLong()

    fun provideVerifier(jwtIssuer: String): JWTVerifier = JWT.require(algorithm).withIssuer(jwtIssuer).build()

    fun createToken(issuer: String, name: String): String =
        JWT.create().withIssuer(issuer)
            .withSubject(name)
            .withExpiresAt(Date(System.currentTimeMillis() + expiredAt))
            .sign(algorithm)
}

object Users : IntIdTable(name = "user") {
    val username = varchar("username", 50).uniqueIndex()
    val password = varchar("password", 100)
}

class User(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<User>(Users)
    var username by Users.username
    var password by Users.password
}

object UserService {
    fun getUserByName(username: String): User? {
        return transaction {
            User.find { Users.username eq username }.firstOrNull()
        }
    }
}


```



必要的是建立数据库，然后`exposed`会根据你定义的`Users` 自动生成表，第一运行之后 ，直接向生成的表中填入数据即可测试。

至此，一个极简的 web应用写好了。 🍻