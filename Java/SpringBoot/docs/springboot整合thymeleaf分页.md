`SpringBoot`结合`Thymeleaf`实现分页，很方便。
## 效果如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904194138219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RkeHlncQ==,size_16,color_FFFFFF,t_70)

<!-- more -->

# 后台代码
项目结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904194533406.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RkeHlncQ==,size_16,color_FFFFFF,t_70)
## 1. 数据库Config
由于`hibernate`自动建表字符集为`latin`不能插入中文，故需要在`application.properties`中指定：`spring.jpa.properties.hibernate.dialect=com.ikeguang.paging.config.MysqlConfig`。
`MysqlConfig.java`代码：
```java
package com.ikeguang.paging.config;

import org.hibernate.dialect.MySQL5Dialect;
import org.springframework.stereotype.Component;

/**
 * @ Author: keguang
 * @ Date: 2019/7/16 9:58
 * @ version: v1.0.0
 * @ description: 解决hibernate自动建表字符集为latin不能插入中文的问题。
 */
@Component
@SuppressWarnings("deprecation")
public class MysqlConfig extends MySQL5Dialect{

    @Override
    public String getTableTypeString() {
        return "ENGINE=InnoDB DEFAULT CHARSET=utf8";
    }
}
```
### 2. 实体类Model
```java
package com.ikeguang.paging.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import java.io.Serializable;

/**
 * @ Author: keguang
 * @ Date: 2019/6/24 20:18
 * @ version: v1.0.0
 * @ description:
 */
@Entity
public class User implements Serializable{
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue
    private Long id;
    @Column(nullable = false, unique = true)
    private String userName;
    @Column(nullable = false)
    private String passWord;
    @Column(nullable = false, unique = true)
    private String email;
    @Column(nullable = true, unique = true)
    private String nickName;
    @Column(nullable = false)
    private String regTime;

    public User(){}

    public User(String userName, String passWord, String email, String nickName, String regTime) {
        this.userName = userName;
        this.passWord = passWord;
        this.email = email;
        this.nickName = nickName;
        this.regTime = regTime;
    }
    // 省略了必须的getter、setter方法
}
```
### 3. Jpa操作数据库
`UserRepository.java`代码
```java
package com.ikeguang.paging.repository;

import com.ikeguang.paging.model.User;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @ Author: keguang
 * @ Date: 2019/7/18 10:23
 * @ version: v1.0.0
 * @ description:
 */
public interface UserRepository extends JpaRepository<User, Long>{

    User findById(long id);

    void deleteById(long id);
}
```
### 4. service层
`UserService`代码
```java
package com.ikeguang.paging.service;

import com.ikeguang.paging.model.User;
import org.springframework.data.domain.Page;

/**
 * @ Author: keguang
 * @ Date: 2019/7/18 10:26
 * @ version: v1.0.0
 * @ description:
 */
public interface UserService {

    Page<User> getUserList(int pageNum, int pageSize);

    User findUserById(long id);

    void save(User user);

    void edit(User user);

    void delete(long id);
}
```
#### service实现层
`UserServiceImpl.java`代码
```java
package com.ikeguang.paging.service.impl;

import com.ikeguang.paging.model.User;
import com.ikeguang.paging.repository.UserRepository;
import com.ikeguang.paging.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;

/**
 * @ Author: keguang
 * @ Date: 2019/7/18 10:27
 * @ version: v1.0.0
 * @ description:
 */
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository userRepository;

    @Override
    public Page<User> getUserList(int pageNum, int pageSize) {

        Sort sort = new Sort(Sort.Direction.DESC, "id");
        Pageable pageable = PageRequest.of(pageNum, pageSize, sort);
        Page<User> users = userRepository.findAll(pageable);

        return users;
    }

    @Override
    public User findUserById(long id) {
        return userRepository.findById(id);
    }

    @Override
    public void save(User user) {
        userRepository.save(user);
    }

    @Override
    public void edit(User user) {
        userRepository.save(user);
    }

    @Override
    public void delete(long id) {
        userRepository.deleteById(id);
    }
}
```
### 5. Controller层
`UserController .java`代码
```java
package com.ikeguang.paging.web;

import com.ikeguang.paging.model.User;
import com.ikeguang.paging.service.UserService;
import org.springframework.data.domain.Page;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.annotation.Resource;
import java.util.Iterator;

/**
 * @ Author: keguang
 * @ Date: 2019/7/18 10:29
 * @ version: v1.0.0
 * @ description:
 */
@Controller
public class UserController {

    @Resource
    UserService userService;

    @RequestMapping("/")
    public String index() {
        return "redirect:/list";
    }

    @RequestMapping("/list")
    public String list(Model model, @RequestParam(value = "pageNum", defaultValue = "0") int pageNum, @RequestParam(value = "pageSize", defaultValue = "2") int pageSize) {
        System.out.println("============================");
        Page<User> users=userService.getUserList(pageNum, pageSize);
        System.out.println("总页数" + users.getTotalPages());
        System.out.println("当前页是：" + pageNum);

        System.out.println("分页数据：");
        Iterator<User> u = users.iterator();
        while (u.hasNext()){

            System.out.println(u.next().toString());
        }

        model.addAttribute("users", users);


        return "user/list";
    }

    @RequestMapping("/toAdd")
    public String toAdd() {
        return "user/userAdd";
    }

    @RequestMapping("/add")
    public String add(User user) {
        userService.save(user);
        return "redirect:/list";
    }

    @RequestMapping("/toEdit")
    public String toEdit(Model model,Long id) {
        User user=userService.findUserById(id);
        model.addAttribute("user", user);
        return "user/userEdit";
    }

    @RequestMapping("/edit")
    public String edit(User user) {
        userService.edit(user);
        return "redirect:/list";
    }


    @RequestMapping("/delete")
    public String delete(Long id) {
        userService.delete(id);
        return "redirect:/list";
    }
}
```
# application.properties配置文件
主要配制了`mysql`数据源，数据库驱动`com.mysql.cj.jdbc.Driver`，对于`mysql-connector-java`用的`6.0`以上的，如果用`com.mysql.jdbc.Driver`，就会报错。
```
spring.datasource.url=jdbc:mysql://localhost/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# 表不存在则新建表
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=com.ikeguang.paging.config.MysqlConfig
spring.jpa.show-sql= true

spring.thymeleaf.cache=false
```
# 模板文件
这里用了`bootstrap.css`里面的样式。这里主要展示一下分页代码，前面的`table`主要装一个`Pageable`的`N`条数据，接着是一个`add`添加数据的按钮，最下面就是分页部分，主要有`5`部分：`首页，上一页，中间页，下一页，尾页`。
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>userList</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}"></link>

</head>
<body class="container">
<br/>
<h1>用户列表</h1>
<br/><br/>
<div class="with:80%">
    <table class="table table-hover">
        <thead>
        <tr>
            <th>#</th>
            <th>userName</th>
            <th>passWord</th>
            <th>email</th>
            <th>nickName</th>
            <th>regTime</th>
            <th>Edit</th>
            <th>Delete</th>
        </tr>
        </thead>
        <tbody>
        <tr th:each="user : ${users}">
            <th scope="row" th:text="${userStat.index + 1}">1</th>
            <td th:text="${user.userName}">neo</td>
            <td th:text="${user.passWord}">Otto</td>
            <td th:text="${user.email}">6</td>
            <td th:text="${user.nickName}">6</td>
            <td th:text="${user.regTime}">6</td>
            <td><a th:href="@{/toEdit(id=${user.id})}">edit</a></td>
            <td><a th:href="@{/delete(id=${user.id})}">delete</a></td>
        </tr>
        </tbody>
    </table>

</div>
<div class="form-group">
    <div class="col-sm-2 control-label">
        <a href="/toAdd" th:href="@{/toAdd}" class="btn btn-info">add</a>
    </div>
</div>

<div class="modal-footer no-margin-top">
    <ul class="pagination pull-right no-margin">

        <!-- 首页 -->
        <li>
            <a th:href="'/list?pageNum=0'">首页</a>
        </li>

        <!-- 上一页 -->
        <li th:if="${users.hasPrevious()}">
            <a th:href="'/list?pageNum=' + ${users.previousPageable().getPageNumber()}" th:text="上一页"></a>
        </li>

        <!-- 中间页 -->
        <li th:each="pageNum:${#numbers.sequence(0, users.getTotalPages() - 1)}">
            <a th:href="'/list?pageNum=' + ${pageNum}" th:text="${pageNum + 1}" th:if="${pageNum ne users.pageable.getPageNumber()}"></a>
            <a th:href="'/list?pageNum=' + ${pageNum}" th:text="${pageNum + 1}" th:if="${pageNum eq users.pageable.getPageNumber()}" th:style="'font-weight:bold;background: #6faed9;'"></a>
        </li>

        <!-- 下一页 -->
        <li th:if="${users.hasNext()}">
            <a th:href="'/list?pageNum=' + ${users.nextPageable().getPageNumber()}" th:text="下一页"></a>
        </li>

        <!-- 尾页 -->
        <li>
            <a th:href="'/list?pageNum=' + ${users.getTotalPages() - 1}">尾页</a>
        </li>

    </ul>
</div>

</body>
</html>
```
## 代码的`Github`地址

[代码地址](https://github.com/ddxygq/spring-boot-learn/tree/master/spring-boot-paging)

