AuthInterceptor
```java
// 声明一个名为AuthInterceptor的切面类  
@Aspect  
@Component  
public class AuthInterceptor {  
  
    // 使用@Resource注解注入UserService实例，这是一个用户服务类，用于获取和处理用户相关的数据  
    @Resource  
    private UserService userService;  
  
    /**  
     * 定义一个环绕通知方法，该方法将在带有AuthCheck注解的方法执行前后被调用  
     *   
     * @param joinPoint 切点，表示被拦截的方法  
     * @param authCheck 用于拦截带有此注解的方法  
     * @return 返回被拦截方法的执行结果  
     * @throws Throwable 如果发生异常，则抛出异常  
     */  
    @Around("@annotation(authCheck)")  
    public Object doInterceptor(ProceedingJoinPoint joinPoint, AuthCheck authCheck) throws Throwable {  
        // 从AuthCheck注解中获取必须的角色值  
        String mustRole = authCheck.mustRole();  
        // 获取当前请求的属性，包括请求头、参数等  
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();  
        // 从请求属性中获取HttpServletRequest对象，用于获取请求相关的信息，如URL、参数等  
        HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();  
        // 通过UserService的getLoginUser方法获取当前登录的用户信息  
        // 当前登录用户  
        User loginUser = userService.getLoginUser(request);  
        // 如果必须的角色值不为空，则进行以下校验  
        if (StringUtils.isNotBlank(mustRole)) {  
            // 尝试将必须的角色值转换为枚举类型，如果转换失败，则抛出业务异常，表示没有找到对应的权限角色  
            UserRoleEnum mustUserRoleEnum = UserRoleEnum.getEnumByValue(mustRole);  
            if (mustUserRoleEnum == null) {  
                throw new BusinessException(ErrorCode.NO_AUTH_ERROR);  
            }  
            // 获取当前登录用户的角色值  
            String userRole = loginUser.getUserRole();  
            // 如果必须的角色是"封号"，则直接抛出业务异常，拒绝请求  
            if (UserRoleEnum.BAN.equals(mustUserRoleEnum)) {  
                throw new BusinessException(ErrorCode.NO_AUTH_ERROR);  
            }  
            // 如果必须的角色是"管理员"，则检查当前登录用户的角色是否与必须的角色匹配，如果不匹配，则抛出业务异常  
            if (UserRoleEnum.ADMIN.equals(mustUserRoleEnum)) {  
                if (!mustRole.equals(userRole)) {  
                    throw new BusinessException(ErrorCode.NO_AUTH_ERROR);  
                }  
            }  
        }  
        // 如果上述所有校验都通过，则继续执行被拦截的方法，并返回其结果  
        return joinPoint.proceed();  
    }  
}
```
这段代码是一个用于权限拦截的Java类，名为AuthInterceptor。它使用了Spring AOP（Aspect-Oriented Programming）来拦截特定注解的方法，并进行权限校验。以下是代码的详细解释：

1. **注解和组件声明**：
- @Aspect：声明该类是一个切面类，用于定义跨多个点的横切关注点。
- @Component：使该类成为Spring组件，便于在其他地方通过@Autowired注入。
1. **​**