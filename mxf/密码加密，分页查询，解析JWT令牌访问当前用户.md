- 在service层，如果传来的数据和实体化类的所有属性名称一样，可以使用
`BeanUtils.copyProperties(employeeDTO,employee);`
其中前一个参数是传入的数据，第二个参数是需要实例化的对象
实体类的属性应该更多，所有剩下的需要自己手动设置
- 密码加密
`employee.setPassword(DigestUtils.md5DigestAsHex("123456".getBytes()));`
- 解析JWT令牌访问当前用户，方法：ThreadLocal，每次请求成为一次线程，可设置类似全局变量
```java
package com.sky.context;

public class BaseContext {

    public static ThreadLocal<Long> threadLocal = new ThreadLocal<>();

    public static void setCurrentId(Long id) {
        threadLocal.set(id);
    }

    public static Long getCurrentId() {
        return threadLocal.get();
    }

    public static void removeCurrentId() {
        threadLocal.remove();
    }

}
```
	`BaseContext.setCurrentId(empId);`设置变量
        `employee.setUpdateUser(BaseContext.getCurrentId());`使用变量

- 分页查询
先将分页的页码和大小传入
`PageHelper.startPage(employeePageQueryDTO.getPage(),employeePageQueryDTO.getPageSize());`
调用Mapper层
`Page<Employee> page=employeeMapper.pageQuery(employeePageQueryDTO);`<>里为实体类，可以通过get函数获取参数
`List<Employee> records = page.getResult();`
```java
    @GetMapping("/page")
    @ApiOperation("员工分页查询")
    public Result<PageResult> page(EmployeePageQueryDTO employeePageQueryDTO){
        PageResult pageResult = employeeService.pageQuery(employeePageQueryDTO);
        return Result.success(pageResult);
```
```java
    @Override
    public PageResult pageQuery(EmployeePageQueryDTO employeePageQueryDTO) {
        //分页查询
        PageHelper.startPage(employeePageQueryDTO.getPage(),employeePageQueryDTO.getPageSize());
        Page<Employee> page = employeeMapper.pageQuery(employeePageQueryDTO);
        long total = page.getTotal();
        List<Employee> records = page.getResult();

        return new PageResult(total,records);
    }
```
```java
Page<Employee> pageQuery(EmployeePageQueryDTO employeePageQueryDTO);
```
```java
<select id="pageQuery" resultType="com.sky.entity.Employee">
        select * from employee
        <where>
            <if test="name != null and name != ''">
                and name like concat('%',#{name},'%')
            </if>
        </where>
        order by create_time desc
    </select>
```
