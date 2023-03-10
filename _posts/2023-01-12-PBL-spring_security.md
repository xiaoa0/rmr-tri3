---
toc: true
comments: false
layout: post
title:  Spring Security and Roles
description: Some tips on building Spring Security into your Web Application(s)
categories: []
type: pbl
week: 18
---

## Security Concepts
Last year Teacher spent time in Spring Security System for csa.nighthawkcodingsociety.com.  The implementation allows users (Person POJO) to login with specific roles (Role POJO). Two roles are present currently, ROLE_ADMIN and ROLE_STUDENT. In this implementation the ROLE_STUDENT is NOT able to delete users,  you must be ROLE_ADMIN to perform this task. 

Teacher took inspiration form [YouTube](https://www.youtube.com/watch?v=VVn9OG9nfH0&t=6350s).  This video covers the detail shared below in about 2 hours.

My issue with this starter code is I never had time to get to JWT and do not have a true web implementation.  Some thoughts the Teacher has on that topic are in JWT tech talk, most of the info is courtesy of ChatGPT.

### Main Ideas

Spring Security Code is focused in these areas: [Database Folder](https://github.com/nighthawkcoders/nighthawk_csa/tree/master/src/main/java/com/nighthawk/csa/mvc/database), [Security Folder](https://github.com/nighthawkcoders/nighthawk_csa/tree/master/src/main/java/com/nighthawk/csa/mvc/security), [Login Page](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/resources/templates/login.html) and [Navbar fragment](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/resources/templates/fragments/nav.html#L116-L126).

- Backend - Using Spring Security to Protect 
    - Defining POJOs
    - Back End Spring Security Overrides to use Person and Role POJOs
    - Authentication (LOGIN), requiring login to access certain parts of site
    - Authorization (ROLE), differentiation of permission of ROLE_ADMIN and ROLE_ADMIN on
    - Training Spring Security to work with POJOs and Security Config 
- Frontend - Building Configuration and Overrides to support Spring Security
    - Login Page (customizing your own)
    - Navbar with Control Login/Logout buttons
    - Error Page for 403 authorization failure


### Backend - Using Spring Security to Protect
The key work of the backend and Spring Security is to associate user (Person) and their (Roles) with access to Get and Post Mappings.  Planning needs to go into your POJO to have role(s) associated with each person.  

### Person POJO with roles.
> Observe Many to Many and role collection.

```java
@Entity
public class Person {
    // automatic unique identifier for Person record
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    // email, password, roles are key to login and authentication
    @NotEmpty
    @Size(min=5)
    @Column(unique=true)
    @Email
    private String email;

    @NotEmpty
    private String password;

    // roles are stored in a "related" table
    @ManyToMany(fetch = EAGER)
    private Collection<Role> roles = new ArrayList<>();

    ... 
}
```
### Role POJO.
> Contains String to store ROLE_ADMIN, ROLE_STUDENT as distinct rows.  All users have ability to obtain the same roles as known by the system.

```java
@Entity
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(unique=true)
    private String name;
}
```

###vBack End Code - Authentication and Authorization
> [SecurityConfig](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/java/com/nighthawk/csa/mvc/security/SecurityConfig.java) provided through Spring Security enables developer to Authorize and Authentication different portions of the site based on Login and Roles.  Additionally, you are able to direct the next page on operations like login and logout.  Review .antmatchers statements.  This file is being established to support http for both Web viewing and API access.

``` java
protected void configure(HttpSecurity http) throws Exception { //THis is going to be altered to use the JWT
        // security rules
        http
            .authorizeRequests()
                .antMatchers(POST, "/api/person/post/**").hasAnyAuthority("ROLE_STUDENT")
                .antMatchers(DELETE, "/api/person/delete/**").hasAnyAuthority("ROLE_ADMIN")
                .antMatchers("/database/personupdate/**").hasAnyAuthority("ROLE_STUDENT")
                .antMatchers("/database/persondelete/**").hasAnyAuthority("ROLE_ADMIN")
                .antMatchers( "/api/person/**").permitAll()
                .antMatchers( "/api/refresh/token/**").permitAll()
                .antMatchers("/", "/starters/**", "/frontend/**", "/mvc/**", "/database/person/**", "/database/personcreate", "/database/scrum/**", "/course/**").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/database/person")
                .permitAll()
        ;
        // Cross-Site Request Forgery needs to be disabled to allow activation of JS Fetch URIs
        http.csrf().disable();
    }
```

### Backend Code - Training Spring Security to work with POJO and Security Config 
> [ModelRepo](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/java/com/nighthawk/csa/mvc/database/ModelRepository.java) implements UserDetailsService.  This associates user (Person) and roles (Role) with Spring Security as you can see with return values from method.

```java
public class ModelRepository implements UserDetailsService
```

```java
/* UserDetailsService Overrides and maps Person & Roles POJO into Spring Security */
    @Override
    public org.springframework.security.core.userdetails.UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        Person person = personJpaRepository.findByEmail(email); // setting variable user equal to the method finding the username in the database
        if(person==null){
            throw new UsernameNotFoundException("User not found in database");
        }
        Collection<SimpleGrantedAuthority> authorities = new ArrayList<>();
        person.getRoles().forEach(role -> { //loop through roles
            authorities.add(new SimpleGrantedAuthority(role.getName())); //create a SimpleGrantedAuthority by passed in role, adding it all to the authorities list, list of roles gets past in for spring security
        });
        return new org.springframework.security.core.userdetails.User(person.getEmail(), person.getPassword(), authorities);
    }
```

> Password Encryption.  (BCrypt) is also managed with this file
```java
    // Setup Password style for Database storing and lookup
    @Autowired  // Inject PasswordEncoder
    private PasswordEncoder passwordEncoder;
    @Bean  // Sets up password encoding style
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
```

> Password is encoded before JPA save.  Spring Security does decoding within its system, later we will build login to send user and password to Spring Security.
```java
public void save(Person person) {
        person.setPassword(passwordEncoder.encode(person.getPassword()));
        personJpaRepository.save(person);
    }
```

### Frontend - Building Configuration and Overrides to support Spring Security

> Login Page (customizing your own).  If you override the form, it needs to be built returning a username and password.  The override was setup in [MvcConfig](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/java/com/nighthawk/csa/mvc/security/MvcConfig.java).  The purpose of override is so it follows Style guideline from the site.

```html
<form name="f" th:action="@{/login}" method="POST">
    <table>
        <tr th:if="${param.error}" class="alert alert-error">
            Invalid username and password.
        </tr>
        <tr th:if="${param.logout}" class="alert alert-success">
            You have been logged out.
        </tr>
        <!-- 'email' is mapped to 'username' for Spring Security -->
        <tr><th><label for="username">Email</label></th></tr>
        <tr><td><input type="email" id='username' name="username" size="20" required></td></tr>
        <tr><th><label for="password">Password</label></th></tr>
        <tr><td><input type="password" id="password" name="password" size="20" required></td></tr>
        <tr><th><input type="submit" value="Submit"></th></tr>
        <tr><td><a th:href="@{/database/personcreate}">Sign Up</a></td></tr>
</table>
```

> Navbar Customizations. Using Thymeleaf and Spring Security the Navbar toggles between Login/Logout based off of Spring Security status.  
```html
<!-- Login/Logout -->
    <th:block sec:authorize="isAnonymous()">
    <div class="px-3">
        <a th:href='@{/login}' class="link-dark">Login</a>
    </div>
    </th:block>
    <th:block sec:authorize="isAuthenticated()">
    <div class="px-3">
       <a th:href='@{/logout}' class="link-dark">Logout</a>
    </div>
    </th:block>
```

> Authorization error - 403. Following standard of 404 errors, a 403 page is inserted to provide better user experience versus Spring Boot error page.  [403.html](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/resources/templates/error/403.html).  Main purpose of this page is to direct the user back into the website with href.

```html
<div class="jumbotron jumbotron-fluid" style="text-align: center; ">
    <h2>You don't have permission to do that! (Insufficient Role)</h2>
        <a href="/">Go Home</a>
    </div>
</div>
```

## Hacks
> This blog should give you many considerations when considering how to design a login system with roles.  There are many features to consider, specifically what elements of site will be used by users/roles.  
- Start to design security features for your own project
- Make a clone of the Spring Project
- Try to restrict Delete on Scrum Page to ADMIN rule.
