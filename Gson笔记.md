# Gson笔记
* **序列化 指 object转json**
* **反序列化 指 json转object**
## 注解
### @SerializedName
* value 默认项，序列化和反序列化时映射的名称
* alternate，反序列化时备选名
### @Expose
* 需要配合`GsonBuilder.excludeFieldsWithoutExposeAnnotation()`使用
* deserialize，反序列化时是否使用该属性
* serialize，序列化时是否使用该属性
### @Since, @Until
* 需要配合`GsonBuilder.setVersion(Double)`使用
* 当前版本(GsonBuilder中设置的版本) 大于等于Since的值时该字段导出，小于Until的值时该该字段导出。**注：当一个字段被同时注解时，需两者同时满足条件。**
## GsonBuilder配置
* serializeNulls()导出null值的键
* setDateFormat("yyyy-MM-dd")格式化输出日期
* disableInnerClassSerialization()禁止序列化内部类
* disableHtmlEscaping()禁止转义html标签
* setPrettyPrinting()格式化输出
* excludeFieldsWithoutExposeAnnotation()排除添加了`@Expose`注释的属性
* excludeFieldsWithModifiers(Modifier.FINAL, Modifier.STATIC, Modifier.PRIVATE)排除个别修饰符
* addSerializationExclusionStrategy()/addDeserializationExclusionStrategy()设置序列化/反序列化策略，需要自己实现一个类
* setFieldNamingPolicy()设置属性命名策略**注意： `@SerializedName`注解拥有最高优先级，在加有`@SerializedName`注解的字段上`FieldNamingStrategy`不生效！**

|FieldNamingPolicy|结果（仅输出emailAddress字段）|
|-|-|
|IDENTITY|{"emailAddress":"ikidou@example.com"}|
|LOWER_CASE_WITH_DASHES|{"email-address":"ikidou@example.com"}|
|LOWER_CASE_WITH_UNDERSCORES|{"email_address":"ikidou@example.com"}|
|UPPER_CAMEL_CASE|{"EmailAddress":"ikidou@example.com"}|
|UPPER_CAMEL_CASE_WITH_SPACES|{"Email Address":"ikidou@example.com"}|
* registerTypeAdapter()自己实现一个类接管某种类型的序列化和反序列化过程