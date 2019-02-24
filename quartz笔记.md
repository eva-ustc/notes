# Quartz笔记

## Cron表达式

### 语法格式
* 秒 分 时 日期 月份 星期 年份（可选）

### 各字段简介
|字段|允许值|允许特殊字符|
|-|-|-|
|秒|0 - 59|, - * /|
|分|0 - 59|, - * /|
|时|0 - 23|, - * /|
|日期|1 - 31|, - * ? / L W C|
|月份|1 - 12 或 JAN-DEC|, - * /|
|星期|1 - 7(**1=周日**) 或 SUN-SAT|, - * ? / L C #|
|年份|1970 - 2099|, - * /|

### 特殊字符含义
* `*` 表示匹配任意值
* `?` 只能用在`日期`和`星期`，也表示匹配任意值。由于`日期`和`星期`会相互影响，因此固定了一个值的话，另一个值只能用`?`修饰。
* `-` 表示范围匹配。
* `/` 表示起始时间开始触发，然后每隔固定时间触发一次。例如：`5/10`，表示匹配`5 + 10n (n >= 0)`
* `,` 表示列出枚举值。例如`5, 10`表示匹配5和10。
* `L` 表示最后(last)，只能用在`日期`和`星期`。例如在星期使用`5L`表示最后一个星期四触发。
* `W` 表示有效工作日（周一到周五），只能出现在`日期`，系统将在离指定日期的最近的有效工作日触发。例如：在`日期`使用`5W`，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一到星期五中的一天，则就在5日触发。**注意，W的最近寻找不会跨过月份**。
* `LW` 两个字符连用，表示某月最后一个工作日，即每个月最后一个星期五
* `#` 用于匹配每个月第几个星期几，只能用在`星期`。例如`4#2`表示某月第二个星期三。

## 解决Job中无法Autowired问题
一般情况下，quartz的job使用autowired注解注入的对象为空，因此自己继承了工厂类，在job返回之前通过AutowireCapableBeanFactory为其绑定属性
``` java
public class MyJobFactory extends AdaptableJobFactory {

    @Autowired  
    private AutowireCapableBeanFactory capableBeanFactory;  
  
    @Override  
    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {  
        Object jobInstance = super.createJobInstance(bundle);  
        capableBeanFactory.autowireBean(jobInstance);
        return jobInstance;  
    }  
}
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <!-- 自己重写了工厂类，使得job能够使用autowired注解 -->
    <property name="jobFactory">
        <bean class="新的工厂类" />
    </property>
</bean>
```