



前后端代码：[GitHub地址](https://github.com/JZWAkihi/LoginDemo)



##### Vue-Cli创建前端项目

![image-20210518083619085](https://gitee.com/Akihij/PicGo/raw/master/img/20210518083626.png)





##### 前端项目目录结构

我们需要Login.vue和Home.vue组件，一个用于登录，一个用于登录成功之后的跳转。

我们需要封装一些函数，这些函数用于在前后端交互时请求与响应的拦截。定义api.js

我们还需要删除原有的组件，清除App.vue的内容（不能删除）。

![image-20210518091931031](https://gitee.com/Akihij/PicGo/raw/master/img/20210518091931.png)



##### 引入Element-ui

参考[element-ui官网](https://element.eleme.cn/#/zh-CN/component/installation)

在main.js文件中引入Element-ui

~~~javascript
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI)
~~~



在VScode终端输入npm i element-ui -S

![image-20210518085600049](https://gitee.com/Akihij/PicGo/raw/master/img/20210518085600.png)



在main.js中引入element-ui

~~~javascript
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);
~~~



可以在package.json中查看是否引入成功

~~~json
  "dependencies": {
    "element-ui": "^2.15.1",
    "vue": "^2.5.2",
    "vue-router": "^3.0.1"
  },
~~~



#### 代码实现：

##### Login.vue

~~~vue
<template>
  <div>
    <el-form ref="loginform" :rules="rules" :model="loginForm" class="loginContainer">
      <h3 class="loginTitle">系统登录</h3>
      <el-form-item prop="username">
        <!-- auto-complete="off"  禁止浏览器表单自动填充 -->
        <!-- placeholder  输入框占位文本 -->
        <el-input type="text" auto-complete="off" v-model="loginForm.username" placeholder="请输入用户名"></el-input>
      </el-form-item>

      <el-form-item prop="password">
        <el-input type="password" v-model="loginForm.password" auto-complete="off" placeholder="请输入密码" ></el-input>
      </el-form-item>

      <el-form-item prop="code">
        <el-input type="text" v-model="loginForm.code" size="normal" placeholder="点击图片更换验证码" auto-complete="off" style="width:250px;margin-right:5px"></el-input>
        <img :src="captchaUrl" @click="updataCaptcha">
      </el-form-item>

      <el-button type="primary" style="width:100%" @click="submitLogin">登录</el-button>

    </el-form>
  </div>
</template>

<script>
import {postRequest} from "../utils/api";
export default {
  name:"Login",
  data(){
    return{
      captchaUrl:'captcha?time' + new Date(),
      loginForm:{
        username:'',
        password:'',
        code:''
      },
      // rules  表单的验证  required   message：提示信息
      rules:{
        username:[{required:true,message:'请输入用户名',trigger:'blur'}],
        password:[{required:true,message:'请输入密码',trigger:'blur'}],
        code:[{required:true,message:'请输入验证码',trigger:'blur'}]
      }
    }
  },
  methods:{
    //点击更新图片
    updataCaptcha(){
      this.captchaUrl = 'captcha?time=' + new Date();
      },
    
    //登录
    submitLogin(){
      this.$refs.loginForm.validate((valid) => {
        if(valid){
          postRequest('/login',this.loginForm).then(resp => {
            console.log(resp);
            if(resp){
              const token = resp.object.tokenHead + resp.object.token;
              window.sessionStorage.setItem('tokenStr',token)
              this.$router.replace('/Home')
            }
          })
        }else{
          console.log('error submit');
          return false;
        }
      })
    }
  }
}
</script>

<style>
  .loginContainer{
        border-radius: 15px;
        background-clip: padding-box;
        margin: 180px auto;
        width: 350px;
        padding: 15px 35px 15px 35px;
        background:#fff;
        border: 1px solid #eaeaea;
        box-shadow: 0 0 25px #cac6c6;
  }

  .loginTitle{
        margin: 0px auto 40px auto;
        text-align: center;
  }
</style>
~~~



##### Api.js

~~~javascript
import axios from 'axios'
import { Message } from 'element-ui'
import router from '../router'


//请求拦截器
axios.interceptors.request.use(config => {
    if(window.sessionStorage.getItem('tokenStr')){
        //请求携带token
        config.headers['Authorization'] = window.sessionStorage.getItem('tokenStr');
    }
    return config
},error => {
    console.log(error);
})



//响应拦截器
// success 成功调到后端接口之后，但是接口不允许进行该操作
axios.interceptors.response.use(success =>{
    if(success.status && success.status == 200){
        if(success.data.code == 500 || success.data.code == 401 || success.data.code == 403){
            Message.error({message:success.data.message});
            return;
        }

        if(success.data.message){
            Message.success({message:success.data.message})
        }
    }
    return success.data;
},error=>{
    //没有访问到接口
    if(error.response.code == 504 || error.response.code == 404){
        Message.error({message:'服务器崩了'});
    }else if(error.response.code == 403){
        Message.error({message:'权限不足'})
    }else if(error.response.code == 401){
        Message.error({message:'未登录'})
        router.replace('/')
    }else{
        if(error.response.data.message){
            Message.error({message:error.response.data.message})
        }else{
            Message.error({message:'未知错误'})
        }
    }
})




let base = '';

export const postRequest = (url,params) => {
    return axios({
        method:'post',
        url:`${base}${url}`,
        data:params
    })
}
~~~





##### 跨域

在index.js中

~~~javascript
proxyTable: {
  '/ws':{
       ws:true,
       target: 'ws://localhost:8081'
   },
   '/':{
       ws: false,
       target: 'http://localhost:8081',
       changeOrigin: true,
       pathRewrite: {
       '^/': '/'
    }
}
~~~





后端：

> Config
>
> ​			CaptchaConfig： 验证码配置文件
>
> ​			JwtTokenFilter：JWT过滤器，判断受否拿到JWT，判断JWT是否有效
>
> ​			SecurityConfig：SpringSecurity 核心配置文件
>
> Controller
>
> ​			CaptchaController：生成验证码
>
> ​			LoginController：登录
>
> dao
>
> ​			AdminMapper：数据库查询
>
> pojo
>
> ​			Admin：实现	UserDetails接口
>
> ​			LoginAdmin：用于接收前端传来的信息，属性有：用户名，密码，验证码
>
> service
>
> ​			AdminServiceImpl：用于登录的主要逻辑
>
> utils
>
> ​			JWTUtils：生成JWT
>
> ​			Respbean：向前端返回结果类



项目框架

![image-20210518145507364](https://gitee.com/Akihij/PicGo/raw/master/img/20210518145514.png)



##### 引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--lombok 依赖-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>


    <!--mysql 依赖-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.2.0</version>
    </dependency>

    <!--   jwt依赖-->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.0</version>
    </dependency>

    <!--security 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!--验证码-->
    <dependency>
        <groupId>com.github.axet</groupId>
        <artifactId>kaptcha</artifactId>
        <version>0.0.9</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <scope>test</scope>
    </dependency>


</dependencies>
```



##### 配置文件

```properties
server.port=8081


# mybatis配置
spring.datasource.url=jdbc:mysql://localhost:3306/login
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root

# mybatis配置文件
mybatis.mapper-locations=classpath*:/Mappers/*.xml

# 日志
# logging.level.root=debug


# Jwt存储的请求头
jwt-tokenHeader=Authorization
# Jwt加密秘钥
jwt-secret=yeb-secret
# Jwt 的超期限时间（60*60）*24
jwt-expiration=604800
# Jwt负载中拿到开头  后端需要用来判断是不是需要的token
jwt-tokenHead=Bearer
```

##### LoginController

```java
@RestController
public class LoginController {

    @Autowired
    private AdminServiceImpl adminService;


    @PostMapping("/login")
    public RespBean login(@RequestBody LoginAdmin loginAdmin, HttpServletRequest request){
        return adminService.login(loginAdmin.getUsername(),loginAdmin.getPassword(),loginAdmin.getCode(),request);
    }
    
}
```



##### LoginServiceImpl

```java
@Service
public class AdminServiceImpl implements AdminService {

    @Autowired
    private AdminMapper adminMapper;

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private JWTUtils jwtUtils;

    @Value("${jwt-tokenHead}")
    private String tokenHead;

    @Override
    public Admin selectAdminByName(String name) {
        return adminMapper.selectOne(name);
    }

    @Override
    public RespBean login(String username, String password, String code, HttpServletRequest request) {
		//验证码验证
        String captcha = (String)request.getSession().getAttribute("captcha");
        System.out.println(captcha);
        if(StringUtils.isEmpty(code) || !captcha.equalsIgnoreCase(code)){
            return RespBean.error("验证码错误，请重新输入");
        }

        //得到用户信息
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        System.out.println(userDetails);
        //用户信息为空，且密码不正确 返回错误

        if(null == userDetails || !passwordEncoder.matches(password,userDetails.getPassword())){
            return RespBean.error("用户名或密码错误");
        }

        if(!userDetails.isEnabled()){
            return RespBean.error("账号被禁用,请联系管理员");
        }

        //更新security登录用户对象

        /***
         * 以UsernamePasswordAuthenticationToken实现的带用户名和密码以及权限的Authentication
         */
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
		
        //将当前登录的信息设置到Spring Security 上下文
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
		
        //创建token 
        String token = jwtUtils.generateToken(userDetails);

        //将token和tokenHead传入前端
        Map<String, String> map = new HashMap<>();
        map.put("token",token);
        map.put("tokenHead",tokenHead);
		
        return RespBean.success("登录成功",map);
    }
}
```



SecurityConfig

```java
/***
 * SpringSecurity  配置类
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AdminServiceImpl adminService;

    
    //重写loadUserByUsername()  lambda表达式
    @Override
    @Bean
    protected UserDetailsService userDetailsService() {
        return username -> {
            Admin admin = adminService.selectAdminByName(username);

            if(null != admin){
                return admin;
            }

            throw new UsernameNotFoundException("用户名或密码不正确");
        };
    }


    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    //根据传入的自定义UserDetailsService添加身份验证。
    // 允许自定义身份验证。
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService()).passwordEncoder(passwordEncoder());
    }

    @Bean
    public JwtTokenFilter jwtTokenFilter(){
        return new JwtTokenFilter();
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf()
                .disable()
                //基于token 不需要session
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                //允许登录访问
                .antMatchers("/login","/logout")
                .permitAll()
                //除了上面，所有请求都要求认证
                .anyRequest()
                .authenticated()
                .and()
                .headers()
                .cacheControl();


        //添加jwt验证过滤器
        http.addFilterBefore(jwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```















