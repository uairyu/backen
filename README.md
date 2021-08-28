# SSM + Vue2.0 的项目（Demo）

	backen.war用于放在tomcat/webapps目录下
	interview-VUE.zip 为单独的前端文件，目前配置的虚拟路径为/backen，可在src/config/_GConfig.js修改virtualPath
	backen-interview-JAVA-SSM.zip 为包括打包好前端了的后端文件
	开发环境：idea
## 前端:

* 整体上使用了flex布局，配合Element-UI组件库
* 主要有4个视图：主页、文章内容页、用户中心页、注册/登陆账号页面
* 主页：
	* 由三部分组成：Header、Main-Home、Footer
	* Header主要展示用户信息，快捷导航功能
	* Main-Home用于展示主体内容
	* Footer用于展示归属等信息
* 文章内容页：
	* 用于展示用户发布的文章内容
* 用户中心：
	* 主要管理用户信息、功能开放等
* 目前实现的功能：
	* 大部分的ajax请求通过`axios`实现
	* 主页在`created`阶段请求简略文章信息获取、文章页请求详细文章内容
	* 修改用户名
	* 登陆和退出登陆，修改用户名称时使用EventBus通知Header更新用户信息
	* 使用分页组件完成文章的分割
	* 注册/登陆页面的账号密码框使用正则表达式进行验证数据合法性，目前使用比较简单的匹配规则`/^[\w]+$/`
	* 文章的id通过动态路由匹配出id值
	* 通过自己手动创建的vue.config.js中，配置publicPath解决布署到后端时（如果有）有虚拟路径的问题，通过引入自己其它位置的配置文件中根据`process.env.NODE_ENV`解决开发阶段与实际布署阶段时，不同的axios请求的url，目前有2种方案解决这个问题：
		1. 每次使用axios时通过一个全局变量来获得请求的域名 + 实际请求的路径
		2. 在main.js中先创建一个axios对象，传入`axios.defaults.baseURL`属性（也可根据`process.env.NODE_ENV`来动态赋值）绑定全局axios对象到Vue的原型上
	* 通过配置全局前置导航守卫监听将要访问路径的`matched`数组是否有某一个路径节点，包含的meta信息里面的登陆校验字段，未登陆跳转到注册/登陆页面，并且在跳转地址中加入redirect参数，当用户登陆之后再判断当前`this.$route.query.redirect`是否有值来返回原来的页面
	* 通过dayjs来转换从后端传过来的文章发布时间的时间戳

## 后端：
* 以SSM（Spring+SpringMVC+MyBatis）作为主要框架
* 使用阿里的Druid作用数据库的连接池，Mysql作为数据库
* 实现的功能：
	* 以Controller + Service + Dao 三层分工处理不同的逻辑和数据
	* 部分配置信息通过配置文件的形式（如数据库连接信息）让程序动态读取，避免每次都重新编译程序，程序中大部分地方采用注解方式配置、注入Bean
	* 配置CORS策略：由于在开发阶段中，前端的文件还未布署，但需要测试接口获取数据时就会违反浏览器的同源策略（在本机上主要为端口号不同，Vue由于通过npm run serve运行，也是http+localhost），目前暂时先简单地通过过滤器来为为一个拦截的请求添加响应头处理
	* 通过PageHelper插件来处理通过数据库获取文章列表时，动态添加limit子句减少读取数量
	* 通过POJO来解决前端的请求参数封装对象问题

## 后续可继续进行增加的功能
* Service层使用spring的声明式事务管理（主要用注解）
* Dao层再细化出一层缓存层，使用redis实现（例如存储文章简略信息等，但要做好相应更新修改写入等维护工作）
* 使用AOP对部分功能进行日志管理（如用户修改密码时间）
* 可以把_GConfig.js放在static目录下，动态修改虚拟路径而不是以打包方式