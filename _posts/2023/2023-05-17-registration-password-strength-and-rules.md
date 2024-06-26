---
layout: post
title:  注册 - 密码强度和规则
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本快速教程中，我们将了解如何在注册期间实现和**显示正确的密码约束**。比如密码应该包含一个特殊字符，或者它应该至少有8个字符长。

我们希望能够使用强大的密码规则-但我们不想实际手动实现这些规则。因此，我们将充分利用成熟的[Passay库](http://www.passay.org/)。

## 2. 自定义密码约束

首先，让我们创建一个自定义约束ValidPassword：

```java
@Documented
@Constraint(validatedBy = PasswordConstraintValidator.class)
@Target({ TYPE, FIELD, ANNOTATION_TYPE })
@Retention(RUNTIME)
public @interface ValidPassword {

    String message() default "Invalid Password";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

并在UserDto中使用它：

```java
@ValidPassword
private String password;
```

## 3. 自定义密码验证器

现在-让我们使用这个库来创建一些强大的密码规则，而不必实际手动实现它们中的任何一个。

我们将创建密码验证器PasswordConstraintValidator并定义密码规则：

```java
public class PasswordConstraintValidator implements ConstraintValidator<ValidPassword, String> {

    @Override
    public void initialize(ValidPassword arg0) {
    }

    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        PasswordValidator validator = new PasswordValidator(Arrays.asList(
              new LengthRule(8, 30),
              new UppercaseCharacterRule(1),
              new DigitCharacterRule(1),
              new SpecialCharacterRule(1),
              new NumericalSequenceRule(3,false),
              new AlphabeticalSequenceRule(3,false),
              new QwertySequenceRule(3,false),
              new WhitespaceRule()));

        RuleResult result = validator.validate(new PasswordData(password));
        if (result.isValid()) {
            return true;
        }
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(
                    Joiner.on(",").join(validator.getMessages(result)))
              .addConstraintViolation();
        return false;
    }
}
```

请注意我们如何在此处**创建新的约束违规并禁用默认违规**-以防密码无效。

最后，我们还要将Passay库添加到我们的pom中：

```xml
<dependency>
	<groupId>org.passay</groupId>
	<artifactId>passay</artifactId>
	<version>1.0</version>
</dependency>
```

关于一些历史信息，Passay是古老的vt-password Java库的后代。

## 4. JS密码表

现在服务器端已经完成，让我们看看客户端并**使用JavaScript实现一个简单的“密码强度”功能**。

我们将使用一个简单的jQuery插件-[用于Twitter Bootstrap的jQuery密码强度计](https://plugins.jquery.com/pwstrength-bootstrap/)在registration.html中显示密码强度：

```html
<input id="password" name="password" type="password"/>

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script src="pwstrength.js"></script>
<script type="text/javascript">
$(document).ready(function () {
    options = {
        common: {minChar:8},
        ui: {
            showVerdictsInsideProgressBar:true,
            showErrors:true,
            errorMessages:{
                wordLength: '<spring:message code="error.wordLength"/>',
                wordNotEmail: '<spring:message code="error.wordNotEmail"/>',
                wordSequences: '<spring:message code="error.wordSequences"/>',
                wordLowercase: '<spring:message code="error.wordLowercase"/>',
                wordUppercase: '<spring:message code="error.wordUppercase"/>',
                wordOneNumber: '<spring:message code="error.wordOneNumber"/>',
                wordOneSpecialChar: '<spring:message code="error.wordOneSpecialChar"/>'
            }
        }
    };
    $('#password').pwstrength(options);
});
</script>
```

## 5. 总结

就是这样-一种在客户端显示密码强度并在服务器端强制执行某些密码规则的简单但非常有用的方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)上获得。