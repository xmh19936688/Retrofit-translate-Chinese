# Retrofit-translate-Chinese
:cn:Retrofit官网翻译

*写在前面：1.如果发现问题欢迎Issue；2.有些不知道对不对的地方给出了原文；3.如果你有更好的建议欢迎Issue；*

#Retrofit
Android和Java平台下类型安全的HTTP客户端

##简介
Retrofit将HTTP API转换为Java接口
```
public interface GitHubService{
	@GET("user/{user}/repos")
	Call<List<Repo>> listRepos(@Path("user") String user);
}
```
`Retrofit`类生成`GitHubService`接口的一个实例
```
Retrofit retrofit = new Retrofit.Builder()
	.baseUrl("https://api.github.com/")
	.build();
	
GitHubService service = retrofit.create(GitHubService.class);
```
实例化后的`GitHubService`中每个`Call`都会发起一个同步或异步的HTTP请求到远程服务器
```
Call<List<Repo>> repos = service.listRepos("octocat");
```
使用注解来描述HTTP请求:
- URL参数替换与请求参数支持
- 请求体类型转换（如JSON，协议缓存）
- 多请求体与文件上传

*注意：*本文仍旧用来说明2.0APIs。

##API说明
接口中的注解和参数哦表明一个请求如何被处理。

###请求类型
每个方法必须有一个HTTP注解来说明请求类型与相对URL。已提供的5种注解：`GET`，`POST`，`PUT`，`DELETE`，`HEAD`。相对URL需要在注解内容中被指定。
```
@GET("users/list")
```
可以在URL中指定请求参数
```
@GET("users/list?sort=desc")
```

###URL变更
可以在方法中使用占位符和参数动态更新URL。占位符是被`{}`包裹的字符串表达式。相应的参数必须用同样的字符串通过`@Path`注解。
```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```
也可以添加请求参数
```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```
对于复杂的请求参数可以用`Map`组织起来
```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

###请求体
一个对象可以通过`@Body`注解在HTTP请求体中指定。
```
@POST("user/new")
Call<User> createUser(@Body User user);
```
这个对象同样会被在`Retrofit`实例中指定的转换器转换。如果没有添加转换器，则只会使用`RequestBody`。

###表单和多组数据
同样可以定义方法来发送表单和多组数据。
如果方法前有`@FormUrlEncoded`注解，则可以发送表单数据。每个键值对都包含在`@Field`注解中，注解中需要包含名称和对象提供的值。*（本句原文：Each key-value pair is annotated with @Field containing the name and the object providing the value.）*
```
@FormUrlEncoded
@Post("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
如果方法前有`@Multipart`注解，则可以发送多组数据。数据组使用`@Part`注解声明。
```
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```
多组数据的数据组使用`Retrofit`的转换器进行序列化，可以通过实现`RequestBody`来自行处理序列化。

###Header变更
可以给方法设置静态headers通过`@Headers`注解。
```
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```
```
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("user/{username}")
Call<User> getUser(@Path("username") String username);
```
注意headers不会相互覆盖。所有使用相同name的headers都会被包含到请求中。
可以使用`@Header`注解动态更新请求头，相应的参数必须用`@Header`提供。如果该值为空，则对应header会被省略。否则该值会通过`toString`转换为字符串被使用。
```
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
需要被添加到每个请求的Headers会通过[OkHttp interceptor]被指定。
[OkHttp interceptor]:https://github.com/square/okhttp/wiki/Interceptors

###同步VS异步
`Call`实例用同步异步都可以执行。每个实例都只能被用一次，但调用`clone()`将创建一个可用的新实例。
在Android上，回调将在主线程中执行。在JVM上，回调执行所在线程与HTTP请求执行线程相同。

##Retrofit配置
`Retrofit`是通过API接口转换得到的可调用对象。默认情况下，Retrofit将根据所用平台自动配置，但也允许自定义。

###转换器
默认情况下，Retrofit只能将HTTP实体反序列化为OkHttp的`ResponseBody`类型，并且只通过`@Body`来接受它的`RequestBody`类型。*（本句原文：By default, Retrofit can only deserialize HTTP bodies into OkHttp's ResponseBody type and it can only accept its RequestBody type for @Body.）*
可以添加指示器来支持其他类型。下面提供6个流行的序列化库。
- [Gson]: com.squareup.retrofit2:converter-gson
[Gson]:https://github.com/google/gson
- [Jackson]: com.squareup.retrofit2:converter-jackson
[Jackson]:http://wiki.fasterxml.com/JacksonHome
- [Moshi]: com.squareup.retrofit2:converter-moshi
[Moshi]:https://github.com/square/moshi/
- [Protobuf]: com.squareup.retrofit2:converter-protobuf
[Protobuf]:https://developers.google.com/protocol-buffers/
- [Wire]: com.squareup.retrofit2:converter-wire
[Wire]:https://github.com/square/wire
- [Simple XML]: com.squareup.retrofit2:converter-simplexml
[Simple XML]:http://simple.sourceforge.net/
- Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars

下面给出一个在反序列化时通过Gson使用`GsonConverterFactory`来实例化`GitHubService`的例子。
```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service =retrofit.create(GitHubService.class);
```

###自定义转换器
如果你需要通过使用了Retrofit不支持的格式化内容（如YAML、txt、自定义格式）调用API，或者需要用一个不同的库来使用已有的格式化规则。创建一个继承自`Converter.Factory`的类并在绑定adapter的时候传递一个实例。

##下载
**[下载最新JAR包]**
[下载最新JAR包]:https://search.maven.org/remote_content?g=com.squareup.retrofit2&a=retrofit&v=LATEST
Retrofit的源码、例子、本网站都在能在[GitHub]上找到。
[GitHub]:https://github.com/square/retrofit

###MAVEN
```
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>(写上最新版本)</version>
</dependency>
```

###GRADLE
```
compile 'com.squareup.retrofit2:retrofit:(写上最新版本)'
```
Retrofit需要Java7以上或Android2.3以上。

###混淆
如果在project中使用混淆需要将如下内容添加到配置中：
```
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }
-keepattributes Signature
-keepattributes Exceptions
```

##贡献
如果你想贡献代码，你可以再GitHub上fork一下然后pull下来。
当提交代码时，请保证尽可能遵从现有约定和格式来保证保持代码可读性。同时请保证你的代码通过运行`mvn clean verfy`来编译。
在你的代码被接纳之前，你必须签上[CLA].
[CLA]:http://bit.ly/YqSdrU?cc=862837c08457717d060dfe823fb16ec8

##许可
```
Copyright 2013 Square, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
