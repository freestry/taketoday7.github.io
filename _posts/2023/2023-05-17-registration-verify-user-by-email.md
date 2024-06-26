---
layout: post
title:  注册 – 通过电子邮件激活新帐户
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

本文继续进行中的[Spring Security注册系列](https://www.baeldung.com/spring-security-registration)，其中包含注册过程中缺失的部分-**验证用户的电子邮件以确认他们的帐户**。

注册确认机制强制用户回复注册成功后发送的“**确认注册**”电子邮件，以验证他的电子邮件地址并激活其帐户。用户通过单击通过电子邮件发送给他们的唯一激活链接来执行此操作。

按照这个逻辑，新注册的用户将无法登录系统，直到此过程完成。

## 2. 验证令牌

我们将使用一个简单的验证令牌作为验证用户的关键组件。

### 2.1 VerificationToken实体

VerificationToken实体必须满足以下条件：

1.  它必须链接回用户(通过单向关系)
2.  它将在注册后立即创建
3.  它将在创建后**24小时内过期**
4.  具有**唯一的、随机生成的值**

要求2和3是注册逻辑的一部分。另外两个在一个简单的VerificationToken实体中实现：

```java
@Entity
public class VerificationToken {
    private static final int EXPIRATION = 60 * 24;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String token;

    @OneToOne(targetEntity = User.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_id")
    private User user;

    private Date expiryDate;

    private Date calculateExpiryDate(int expiryTimeInMinutes) {
        Calendar cal = Calendar.getInstance();
        cal.setTime(new Timestamp(cal.getTime().getTime()));
        cal.add(Calendar.MINUTE, expiryTimeInMinutes);
        return new Date(cal.getTime().getTime());
    }

    // standard constructors, getters and setters
}
```

请注意User上的nullable=false以确保VerificationToken<->User关联中的数据完整性和一致性。

### 2.2 将enabled字段添加到User

最初，当用户注册时，此enabled字段将设置为false。在帐户验证过程中如果成功-它将变为true。

让我们首先将字段添加到我们的User实体：

```java
public class User {
    // ...
    @Column(name = "enabled")
    private boolean enabled;

    public User() {
        super();
        this.enabled=false;
    }
    ...
}
```

请注意我们如何将此字段的默认值设置为false。

## 3. 账户注册期间

让我们向用户注册用例添加两个额外的业务逻辑：

1.  为用户生成VerificationToken并持久化
2.  发送用于帐户确认的电子邮件消息-其中包括带有VerificationToken值的确认链接

### 3.1 使用Spring事件创建令牌并发送验证电子邮件

这两个额外的逻辑不应由控制器直接执行，因为它们是“附带的”后端任务。

控制器将发布一个Spring ApplicationEvent来触发这些任务的执行。这就像注入ApplicationEventPublisher然后使用它来发布注册完成一样简单。

下例显示了这个简单的逻辑：

```java
@Autowired
ApplicationEventPublisher eventPublisher

@PostMapping("/user/registration")
public ModelAndView registerUserAccount(@ModelAttribute("user") @Valid UserDto userDto, HttpServletRequest request, Errors errors) {
    try {
        User registered = userService.registerNewUserAccount(userDto);
        String appUrl = request.getContextPath();
        eventPublisher.publishEvent(new OnRegistrationCompleteEvent(registered, request.getLocale(), appUrl));
    } catch (UserAlreadyExistException uaeEx) {
        ModelAndView mav = new ModelAndView("registration", "user", userDto);
        mav.addObject("message", "An account for that username/email already exists.");
        return mav;
    } catch (RuntimeException ex) {
        return new ModelAndView("emailError", "user", userDto);
    }

    return new ModelAndView("successRegister", "user", userDto);
}
```

需要注意的另一件事是围绕事件发布的try-catch块。每当事件发布后执行的逻辑出现异常时，这段代码就会显示错误页面，在本例中是发送电子邮件。

### 3.2 事件和监听器

现在让我们看看我们的控制器发出的这个新的OnRegistrationCompleteEvent的实际实现，以及将要处理它的监听器：

```java
public class OnRegistrationCompleteEvent extends ApplicationEvent {
    private String appUrl;
    private Locale locale;
    private User user;

    public OnRegistrationCompleteEvent(User user, Locale locale, String appUrl) {
        super(user);

        this.user = user;
        this.locale = locale;
        this.appUrl = appUrl;
    }

    // standard getters and setters
}
```

**RegistrationListener处理OnRegistrationCompleteEvent**：

```java
@Component
public class RegistrationListener implements ApplicationListener<OnRegistrationCompleteEvent> {

    @Autowired
    private IUserService service;

    @Autowired
    private MessageSource messages;

    @Autowired
    private JavaMailSender mailSender;

    @Override
    public void onApplicationEvent(OnRegistrationCompleteEvent event) {
        this.confirmRegistration(event);
    }

    private void confirmRegistration(OnRegistrationCompleteEvent event) {
        User user = event.getUser();
        String token = UUID.randomUUID().toString();
        service.createVerificationToken(user, token);

        String recipientAddress = user.getEmail();
        String subject = "Registration Confirmation";
        String confirmationUrl
              = event.getAppUrl() + "/regitrationConfirm?token=" + token;
        String message = messages.getMessage("message.regSucc", null, event.getLocale());

        SimpleMailMessage email = new SimpleMailMessage();
        email.setTo(recipientAddress);
        email.setSubject(subject);
        email.setText(message + "\r\n" + "http://localhost:8080" + confirmationUrl);
        mailSender.send(email);
    }
}
```

在这里，confirmRegistration方法将接收OnRegistrationCompleteEvent，从中提取所有必要的用户信息，创建验证令牌，持久化，然后将其作为参数发送到“确认注册”链接。

如上所述，JavaMailSender抛出的任何javax.mail.AuthenticationFailedException都将由控制器处理。

### 3.3 处理验证令牌参数

当用户收到“确认注册”链接时，他们应该点击它。

一旦他们这样做了-控制器将在生成的GET请求中提取token参数的值，并将使用它来启用User。

RegistrationController处理注册确认：

```java
@Autowired
private IUserService service;

@GetMapping("/regitrationConfirm")
public String confirmRegistration (WebRequest request, Model model, @RequestParam("token") String token) {
 
    Locale locale = request.getLocale();
    
    VerificationToken verificationToken = service.getVerificationToken(token);
    if (verificationToken == null) {
        String message = messages.getMessage("auth.message.invalidToken", null, locale);
        model.addAttribute("message", message);
        return "redirect:/badUser.html?lang=" + locale.getLanguage();
    }
    
    User user = verificationToken.getUser();
    Calendar cal = Calendar.getInstance();
    if ((verificationToken.getExpiryDate().getTime() - cal.getTime().getTime()) <= 0) {
        String messageValue = messages.getMessage("auth.message.expired", null, locale)
        model.addAttribute("message", messageValue);
        return "redirect:/badUser.html?lang=" + locale.getLanguage();
    } 
    
    user.setEnabled(true); 
    service.saveRegisteredUser(user); 
    return "redirect:/login.html?lang=" + request.getLocale().getLanguage(); 
}
```

如果出现以下情况，用户将被重定向到包含相应消息的错误页面：

1.  由于某种原因VerificationToken不存在
2.  VerificationToken已过期

badUser.html：

```html
<html>
<body>
<h1 th:text="${param.message[0]}>Error Message</h1>
    <a th:href=" @{/registration.html}"
th:text="#{label.form.loginSignUp}">signup</a>
</body>
</html>
```

如果未发现错误，则启用该用户。

在处理VerificationToken检查和过期场景方面有两个改进机会：

1.  **我们可以使用Cron作业**在后台检查令牌是否过期
2.  我们可以**让用户有机会在过期后获得新令牌**

我们将在以后的文章中推迟新令牌的生成，并假设用户确实在此处成功验证了他们的令牌。

## 4. 在登录过程中添加帐户激活检查

我们需要添加代码来检查用户是否已启用：

```java
@Autowired
UserRepository userRepository;

public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
    boolean enabled = true;
    boolean accountNonExpired = true;
    boolean credentialsNonExpired = true;
    boolean accountNonLocked = true;
    try {
        User user = userRepository.findByEmail(email);
        if (user == null) {
            throw new UsernameNotFoundException("No user found with username: " + email);
        }
        
        return new org.springframework.security.core.userdetails.User(
            user.getEmail(), 
            user.getPassword().toLowerCase(), 
            user.isEnabled(), 
            accountNonExpired, 
            credentialsNonExpired, 
            accountNonLocked, 
            getAuthorities(user.getRole()));
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

正如我们所看到的，现在MyUserDetailsService不使用用户的enabled标志-因此它只允许启用的用户进行身份验证。

现在，我们将添加一个AuthenticationFailureHandler来自定义来自MyUserDetailsService的异常消息。我们的CustomAuthenticationFailureHandler为：

```java
@Component
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Autowired
    private MessageSource messages;

    @Autowired
    private LocaleResolver localeResolver;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request,
                                        HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        setDefaultFailureUrl("/login.html?error=true");

        super.onAuthenticationFailure(request, response, exception);

        Locale locale = localeResolver.resolveLocale(request);

        String errorMessage = messages.getMessage("message.badCredentials", null, locale);

        if (exception.getMessage().equalsIgnoreCase("User is disabled")) {
            errorMessage = messages.getMessage("auth.message.disabled", null, locale);
        } else if (exception.getMessage().equalsIgnoreCase("User account has expired")) {
            errorMessage = messages.getMessage("auth.message.expired", null, locale);
        }

        request.getSession().setAttribute(WebAttributes.AUTHENTICATION_EXCEPTION, errorMessage);
    }
}
```

我们需要修改login.html以显示错误消息：

```html
<div th:if="${param.error != null}"
     th:text="${session[SPRING_SECURITY_LAST_EXCEPTION]}">error
</div>
```

## 5. 适配持久层

现在让我们提供其中一些涉及验证令牌和用户的操作的实际实现。

我们将涵盖：

1.  一个新的VerificationTokenRepository
2.  IUserInterface中的新方法及其对需要的新CRUD操作的实现

VerificationTokenRepository：

```java
public interface VerificationTokenRepository extends JpaRepository<VerificationToken, Long> {
    VerificationToken findByToken(String token);

    VerificationToken findByUser(User user);
}
```

IUserService接口：

```java
public interface IUserService {
    User registerNewUserAccount(UserDto userDto) throws UserAlreadyExistException;

    User getUser(String verificationToken);

    void saveRegisteredUser(User user);

    void createVerificationToken(User user, String token);

    VerificationToken getVerificationToken(String VerificationToken);
}
```

UserService：

```java
@Service
@Transactional
public class UserService implements IUserService {
    @Autowired
    private UserRepository repository;

    @Autowired
    private VerificationTokenRepository tokenRepository;

    @Override
    public User registerNewUserAccount(UserDto userDto) throws UserAlreadyExistException {
        if (emailExist(userDto.getEmail())) {
            throw new UserAlreadyExistException("There is an account with that email adress: " + userDto.getEmail());
        }

        User user = new User();
        user.setFirstName(userDto.getFirstName());
        user.setLastName(userDto.getLastName());
        user.setPassword(userDto.getPassword());
        user.setEmail(userDto.getEmail());
        user.setRole(new Role(Integer.valueOf(1), user));
        return repository.save(user);
    }

    private boolean emailExist(String email) {
        return userRepository.findByEmail(email) != null;
    }

    @Override
    public User getUser(String verificationToken) {
        User user = tokenRepository.findByToken(verificationToken).getUser();
        return user;
    }

    @Override
    public VerificationToken getVerificationToken(String VerificationToken) {
        return tokenRepository.findByToken(VerificationToken);
    }

    @Override
    public void saveRegisteredUser(User user) {
        repository.save(user);
    }

    @Override
    public void createVerificationToken(User user, String token) {
        VerificationToken myToken = new VerificationToken(token, user);
        tokenRepository.save(myToken);
    }
}
```

## 6. 总结

在本文中，我们扩展了注册过程以包括基于电子邮件的帐户激活过程。

帐户激活逻辑需要通过电子邮件向用户发送验证令牌，以便他们可以将其发送回控制器以验证他们的身份。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)上获得。