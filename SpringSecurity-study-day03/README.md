# springSecurity-study
## 1.pom.xml引入jar包
```java
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
```
## 2.@EnableWebSecurity配置类
```java
package com.example.demo.config;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.JdbcUserDetailsManager;

/**
 * @author   czq
 * @Date 2020-08-17 15:50:57    
 */
/**
 * 步骤:2
 * @EnableWebSecurity已经集成@config
 * EnableWebSecurity注解有两个作用,
 * 1: 加载了WebSecurityConfiguration配置类, 配置安全认证策略。
 * 2: 加载了AuthenticationConfiguration, 配置了认证信息
 * 
 * @author czq
 *
 */
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{

	@Autowired
	private DataSource dataSource;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception{
		http.authorizeRequests()
		.antMatchers("/admin/api/**").hasRole("ADMIN")
		.antMatchers("/user/api/**").hasRole("USER")
		.antMatchers("/app/api/**").permitAll()
		.anyRequest().authenticated()
		.and()
		.formLogin().permitAll()
		.and()
		.csrf().disable();
	}
	
	@Override
	@Bean
	public UserDetailsService userDetailsService() {
		JdbcUserDetailsManager manager=new JdbcUserDetailsManager();
		manager.setDataSource(dataSource);
		if(!manager.userExists("user")) {
			manager.createUser(User.withUsername("user").password("123").roles("USER").build());
		}
		if(!manager.userExists("admin")) {
			manager.createUser(User.withUsername("admin").password("123").roles("USER","ADMIN").build());
		}
		
		return manager;
	}
	
	@Bean
	public PasswordEncoder passwordEncoder() {
		return NoOpPasswordEncoder.getInstance();
	}

}
```
WebSecurityConfigurerAdapter有3中configure配置情况，我们取一个就好
protected void configure(AuthenticationManagerBuilder auth)
public void configure(WebSecurity web) throws Exception
protected void configure(HttpSecurity http) throws Exception

## 3.application.properties，增加数据库连接
```
spring.datasource.url=jdbc:mysql://192.168.11.10:3306/test?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true
spring.datasource.username=mms
spring.datasource.password=qwe123
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=1
```
## 4.插入数据库，用的默认数据库
```
SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for `authorities`
-- ----------------------------
DROP TABLE IF EXISTS `authorities`;
CREATE TABLE `authorities` (
  `username` varchar(50) NOT NULL,
  `authority` varchar(50) NOT NULL,
  UNIQUE KEY `ix_auth_username` (`username`,`authority`),
  CONSTRAINT `fk_authorities_users` FOREIGN KEY (`username`) REFERENCES `users` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of authorities
-- ----------------------------
INSERT INTO `authorities` VALUES ('admin', 'ROLE_ADMIN');
INSERT INTO `authorities` VALUES ('admin', 'ROLE_USER');
INSERT INTO `authorities` VALUES ('user', 'ROLE_USER');

-- ----------------------------
-- Table structure for `users`
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` varchar(500) NOT NULL,
  `enabled` tinyint(1) NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of users
-- ----------------------------
INSERT INTO `users` VALUES ('admin', '123', '1');
INSERT INTO `users` VALUES ('user', '123', '1');
```
## 5.登录跳转成功后，会访问/路径
```java
@RestController
@RequestMapping(value = "/admin/api")
public class AdminController {


	@RequestMapping(method = RequestMethod.GET,value = "hello")
	public String hello() {
		return "hello,admin";
	}
}
@RestController
@RequestMapping(value = "/app/api")
public class AppController {


	@RequestMapping(method = RequestMethod.GET,value = "hello")
	public String hello() {
		return "hello,app";
	}
}
@RestController
@RequestMapping(value = "/user/api")
public class UserController {


	@RequestMapping(method = RequestMethod.GET,value = "hello")
	public String hello() {
		return "hello,user";
	}
}
```
有权限的情况下，才能访问
## 4.以下是demo
访问：http://localhost:8080/app/api/hello，直接跳转，因为所有权限都能访问
![Image text](https://github.com/JustShowTime/springSecurity-study/raw/master/images/day2-step1.png)
访问：http://localhost:8080/admin/api/hello，到登陆页面，只有访问这个路径的用户才能访问
用user用户登录
![Image text](https://github.com/JustShowTime/springSecurity-study/raw/master/images/day2-step2.png)
成功登录，但是跳转失败，无权限访问
![Image text](https://github.com/JustShowTime/springSecurity-study/raw/master/images/day2-step3.png)
用admin用户登录
![Image text](https://github.com/JustShowTime/springSecurity-study/raw/master/images/day2-step4.png)
成功登录，并返回hello,admin
![Image text](https://github.com/JustShowTime/springSecurity-study/raw/master/images/day2-step5.png)
