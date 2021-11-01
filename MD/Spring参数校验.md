# Spring-boot参数校验（在Servlet之前把值绑定，所以对类型的判断需要另辟蹊径，比如AOP代理(Spring AOP又不会管理非spring管理的bean，需要静态织入的方式)）
## Hibernate Validator 注解约束
```text
@Range(min=, max=): 注释元素值必须在规定范围内。
@Length(min=, max=): 被注释字符串长度必须在指定范围内。
@URL(protocol=, host=, port=, regexp=): 被注释的字符串必须是一个有效的URL。
@SafeHtml: 判断提交的html是否安全。例如说，不能包含javascript脚本等。
....
```
## @Valid 和 @Validated
```text
@Valid: Bean Validation所定义，表示约束校验。
@Validated: Spring Validation定义，表示约束校验。
```
## 