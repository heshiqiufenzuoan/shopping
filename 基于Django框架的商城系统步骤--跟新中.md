# 				    基于Django框架的商城系统步骤

### 一、准备

- 导入虚拟环境：pip install virtualenv
- 在导入的虚拟环境中创建虚拟环境：mkvirtualenv  -p python shopping_vir
- 查看是否创建成功虚拟环境：workon
- 使用虚拟环境: workon shopping_virtual

### 二、创建商城功能模块

* 创建商城项目：django-admin startproject shopping
* 创建APP：python manage.py startapp  libs[存放第三方包]/log[]/static[静态资源：图片等]/templates[HTML文件等模块]/utils[公共类]

### 三、配置连接

#### 1.  添加APP

```python
# 添加子应用
'apps.users',  # 或'apps.users.apps.UsersConfig'
'apps.contents',
'apps.verifications'
```

#### 2.  配置jinja2引擎

```python
# 在TEMPLATE 模板中配置jinja2引擎
TEMPLATES = [        
    {
        # 配置使用jinja2模板引擎，因为他的速度比Django快十多倍
        'BACKEND': 'django.template.backends.jinja2.Jinja2',
        'DIRS': [os.path.join(BASE_DIR, "templates")],  #  BASE_DIR项目的根目录，templates：放模板的文件夹名称
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            #'environment':'jinja2.Environment' # 默认
            'environment':'utils.jinja2_env.jinja2_environment', #自定义的jinja2环境配置
        },
    },
]
'''
===========================================================
注意，即使使用jinja2引擎也要保留Django自带的默认引擎，否则会报错
===========================================================
'''
```

####  3. 配置MySQL数据库

##### 3.1 建立数据库机制

* 新建shopping_mysql数据库:

```mysql
 create databases shopping_mysql charset=utf8
```

* 新建用户：

```mysql
create user shopping identified by '123456' 
```

* 授权shopping用户可以访问shopping_mysql数据库：

```mysql
grant all on shopping_mall.* to 'shopping'@'%'
```

* 授权结束后刷新特权

```mysql
flush privileges
```

##### 3.2 配置数据库

```python
ATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # 数据库引擎
        'HOST': '127.0.0.1', # 数据库主机
        'PORT': 3306, # 数据库端口
        'USER': 'shopping', # 数据库用户名
        'PASSWORD': '123456', # 数据库用户密码
        'NAME': 'shopping_mall' # 数据库名字
    },
}
'''
如果出现admin......错误
安装mysqlclient扩展包
'''
```

注：

（1）djngo中自带用户认证系统，直接迁移文件即可，不用生成迁移

但如果在数据库的表中想要添加一些手机号码等字段时要在model中添加，不能手动在数据库添加

在model中添加后要进行  迁移文件

（2）常见错误：SystemCheckError:System check  identified some issues;【系统检查确认了一些问题】

​		意思是系统会认为Django内部指定的是auth.model下的user，即默认的是自带的系统模型类，但是我们要让系统使用自己设计的，所以此时要配置。方法如下：直接在setting.py 最后一行写：AUTH_USER_MODEL = 'users.user'

#### 4. 配置redis数据库

```python
pip install django-redis

ACHES = {
    "default": { # 默认
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/0",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    
    "session": { # session
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    
    # 配置验证码的缓存存储
    "verify": { # 验证码
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/2",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
}
# SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "session"

'''
先导入redis库
默认存入0号库  // 总共有16 个库
'''
```

#### 5、配置工作日志

```python
# 日志代码为固定的
OGGING = {
    'version': 1,
    'disable_existing_loggers': False,  # 是否禁用已经存在的日志器
    'formatters': {  # 日志信息显示的格式
        'verbose': {   #日志的显示格式
            'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
        },
    },
    'filters': {  # 对日志进行过滤
        'require_debug_true': {  # django在debug模式下才输出日志
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {  # 日志处理方法
        'console': {  # 向终端中输出日志
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {  # 向文件中输出日志
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs/shopping.log'),  # 日志文件的位置
            'maxBytes': 300 * 1024 * 1024,
            'backupCount': 10,
            'formatter': 'verbose'
        },
    },
    'loggers': {  # 日志器
        'django': {  # 定义了一个名为django的日志器
            'handlers': ['console', 'file'],  # 可以同时向终端与文件中输出日志
            'propagate': True,  # 是否继续传递日志信息
            'level': 'INFO',  # 日志器接收的最低日志级别
        },
    }
}

```

### 四、注册页面

#### 4.1 用户注册前端逻辑

目的：实现用户的交互和页面局部刷新的效果

1. 用户注册页面绑定vue数据
   * 准备div盒子
   * 绑定数据：变量，事件，错误提示等
2. js文件实现用户交互
   - 导入Vue.js库和请求的库
   - 准备register.js文件
   - 用户交互事件实现

#### 4.2 用户注册后端逻辑

1. 接收数据
   - 用户注册数据是从表单发送过来的，所以使用request.POST来提取	
2. 对数据进行二次校验
   - 前端校验过之后，后端也要校验。方法是一致的。
3. 存入数据库
   - 这里使用的是Django认证系统用户模型类提供的create_user()方法创建的新用户
   - 其中
4. 响应注册结果

* 注册成功，重定向到首页

​        （1）创建首页广告应用 contents模块

​		（2） 配置首页广告路由：绑定命名空间

​		（3）定义首页广告视图

​		（4）响应注册结果，即重定向到首页

​		（5）查看结果

```python
分析功能
'''
一、分析功能前端？后端？
	前端：失去焦点发起异步请求验证用户名是否重复，并且显示对应的提示
	后端：验证用户名是否重复
二、分析功能的大体逻辑
	后端：
		1.接收获取数据
		2.根据接收到的数据验证数据是否唯一
		3.根据结果返回相应的内容
三、根据大概逻辑分析细化
	后端：
		1.接收获取数据(从方法中的参数获取)
		2.根据接收到的数据验证数据是否唯一
		3.根据结果返回相应的内容
	前端：
		1. 绑定一个鼠标失去焦点的事件，调用方法
		2. 定义一个方法，发起一个Ajax异步请求--get方式
		3. 根据请求返回的结果判断用户名的有效性
		4. 根据判断结果提示不同的信息。
四、确认请求方式/参数/响应。。。
	1.请求方式：get
	2.参数：url固定的位置捕获参数 'xxx/<str:xxx>'
'''
```

#### 4.5 状态保持

（1）两种方式：

- 注册后即登录的状态保持

- 注册后不登陆的状态保持

（2）login方法：

* 用户登录的本质是将用户唯一标识信息保存到浏览器cookie中和服务器session中
* Django提供了一个用户认证系统
* login()位置位于django.contrib.auth.__init__.py ⽂件中。
* session数据保存在Redis1号库中。

（3）login（）方法登入用户

```python
# 保存注册数据
try:
 user = User.objects.create_user(username=username, password=password, mobile=mobile)
except DatabaseError:
 return render(request, 'register.html', {'register_errmsg': '注册失败'})
# 登⼊⽤户，实现状态保持
login(request, user)
# 响应注册结果
return redirect(reverse('contents:index'))
```

（4）查看状态保持结果

​	在浏览器中感叹号查看

#### 4.6 用户名重复注册

1. 用户名重复注册前端逻辑

```python
class UsernameCountView(View):
 """判断⽤户名是否重复注册"""
 def get(self, request, username):
 """
 :param request: 请求对象
 :param username: ⽤户名
 :return: JSON
 """
 count = User.objects.filter(username=username).count()
 return http.JsonResponse({'code': 200, 'msg': 'OK', 'count': count})
```

2. 用户名重负注册后端逻辑

```python
if (this.error_name == false) {
 let url = '/usernames/' + this.username + '/count/';
 axios.get(url,{
 responseType: 'json'
 })
 .then(response => {
 if (response.data.count == 1) {
 this.error_name_message = '⽤户名已存在';
 this.error_name = true;
 } else {
 this.error_name = false;
 }
 })
 .catch(error => {
 console.log(error.response);
 })
}
```

#### 4.7 手机号重复注册

##### 1. 手机号重复前端逻辑

```python
if (this.error_name == false) {
 let url = '/usernames/' + this.username + '/count/';
 axios.get(url,{
 responseType: 'json'
 })
 .then(response => {
 if (response.data.count == 1) {
 this.error_name_message = '⽤户名已存在';
 this.error_name = true;
 } else {
 this.error_name = false;
 }
 })
 .catch(error => {
 console.log(error.response);
 })
}
```

##### 2. 手机号重复后端逻辑

```python
if (this.error_name == false) {
 let url = '/usernames/' + this.username + '/count/';
 axios.get(url,{
 responseType: 'json'
 })
 .then(response => {
 if (response.data.count == 1) {
 this.error_name_message = '⽤户名已存在';
 this.error_name = true;
 } else {
 this.error_name = false;
 }
 })
 .catch(error => {
 console.log(error.response);
 })
}
```

#### 4.8 图形验证码

* ##### 概要：

 	1. 将图形验证码的文字保存在Redis数据库，为短信验证码做准备
 	2. UUID用于唯一区分该图形验证码术语哪个用户，也可以使用其他唯一标识符信息实现

* 详细步骤

##### 1、接口设计

* 请求方式
* 请求参数
* 响应结果

##### 2、图形验证码接口定义

* 接口视图
* 总路由
* 子路由

##### 3、图形验证码后端逻辑

* 准备captcha包：用于后端生成图形验证码，若出现错误则是因为没有导入python处理图片的库

```python
pip install Pillow
from PIL import Image
from PIL import ImageFilter
from PIL.ImageDraw Iimport Draw
from PIL.ImageFont import tryetype
```

* 准备Redis数据库

```python
"verify_code": { # 验证码
 "BACKEND": "django_redis.cache.RedisCache",
 "LOCATION": "redis://127.0.0.1:6379/2",
 "OPTIONS": {
 "CLIENT_CLASS": "django_redis.client.DefaultClient",
 }
 },
```

##### 4、前端逻辑实现

4.1 vue实现图形验证码展示

```python
# register.js
mounted(){
 // ⽣成图形验证码
 this.generate_image_code();
},
methods: {
 // ⽣成图形验证码
 generate_image_code(){
 // ⽣成UUID。generateUUID() : 封装在common.js⽂件中，需要提前引⼊
 this.uuid = generateUUID();
 // 拼接图形验证码请求地址
 this.image_code_url = "/image_codes/" + this.uuid + "/";
 },
 ......
}
    

# register.html
<li>
 <label>图形验证码:</label>
 <input type="text" name="image_code" id="pic_code" class="msg_input">
 <img :src="image_code_url" @click="generate_image_code" alt="图形验证码"
class="pic_code">
 <span class="error_tip">请填写图形验证码</span>
</li>
```

4.2 vue实现图形验证码校验

```python
# register.html
<li>
 <label>图形验证码:</label>
 <input type="text" name="image_code" id="pic_code" class="msg_input">
 <img :src="image_code_url" @click="generate_image_code" alt="图形验证码"
class="pic_code">
 <span class="error_tip">请填写图形验证码</span>
</li>

# register.js
check_image_code(){
 if(!this.image_code) {
 this.error_image_code_message = '请填写图⽚验证码';
 this.error_image_code = true;
 } else {
 this.error_image_code = false;
 }
},
```

#### 4.9 短信验证码

4.9.1  总体逻辑

1. 概要：
   - 保存验证码是为注册做准备
   - 为了避免用户使用验证码恶意测试，后端获取到验证码后立即删除
   - 借用第三方【云通讯】发送短信验证码

2. 云通讯注册，了解熟悉步骤。
3. 云通讯测试

4.9.2 详细逻辑

1. 前端逻辑

* Vue绑定短信验证码界面

```python
# register
<li>
 <label>短信验证码:</label>
 <input type="text" v-model="sms_code" @blur="check_sms_code" name="sms_code"
id="msg_code" class="msg_input">
 <a @click="send_sms_code" class="get_msg_code">[[ sms_code_tip ]]</a>
 <span class="error_tip" v-show="error_sms_code">[[ error_sms_code_message ]]</span>
</li>

# register.html
<li>
 <label>短信验证码:</label>
 <input type="text" v-model="sms_code" @blur="check_sms_code" name="sms_code"
id="msg_code" class="msg_input">
 <a @click="send_sms_code" class="get_msg_code">[[ sms_code_tip ]]</a>
 <span class="error_tip" v-show="error_sms_code">[[ error_sms_code_message ]]</span>
</li>
```

* axios请求短信验证码

  ```python
  '''
  1. 发送短信验证码事件
  2. 发送短信验证码效果【即得到结果】
  '''
  send_sms_code(){
   // 避免重复点击
   if (this.sending_flag == true) {
   return;
   }
   this.sending_flag = true;
   // 校验参数
   this.check_mobile();
   this.check_image_code();
   if (this.error_mobile == true || this.error_image_code == true) {
   this.sending_flag = false;
   return;
   }
   // 请求短信验证码
   let url = '/sms_codes/' + this.mobile + '/?image_code=' + this.image_code+'&uuid='+
  this.uuid;
   axios.get(url, {
   responseType: 'json'
   })
   .then(response => {
   if (response.data.code == '0') {
   // 倒计时60秒
   var num = 60;
   var t = setInterval(() => {
   if (num == 1) {
   clearInterval(t);
  this.sms_code_tip = '获取短信验证码';
   this.sending_flag = false;
   } else {
   num -= 1;
   // 展示倒计时信息
  this.sms_code_tip = num + '秒';
   }
   }, 1000, 60)
   } else {
   if (response.data.code == '4001') {
   this.error_image_code_message = response.data.errmsg;
   this.error_image_code = true;
   } else { // 4002
   this.error_sms_code_message = response.data.errmsg;
   this.error_sms_code = true;
   }
   this.generate_image_code();
   this.sending_flag = false;
   }
   })
   .catch(error => {
   console.log(error.response);
   this.sending_flag = false;
   })
  },
  ```

2. 后端逻辑

   （1）接口设计：

   - 请求方式
   - 请求参数：路径参数和查询字符串
   - 响应结果：JSON

   （2) 接口定义

   ```python
   class SMSCodeView(View):
    """短信验证码"""
    def get(self, reqeust, mobile):
    """
    :param reqeust: 请求对象
    :param mobile: ⼿机号
    :return: JSON
    """
    pass
   ```

   （3）逻辑实现

   ```python
   class SMSCodeView(View):
    """短信验证码"""
    def get(self, reqeust, mobile):
    """
    :param reqeust: 请求对象
    :param mobile: ⼿机号
    :return: JSON
    """
    # 接收参数
    image_code_client = reqeust.GET.get('image_code')
    uuid = reqeust.GET.get('uuid')
    # 校验参数
    if not all([image_code_client, uuid]):
    return http.JsonResponse({'code': RETCODE.NECESSARYPARAMERR, 'errmsg': '缺少必传
   参数'})
        # 创建连接到redis的对象
    	redis_conn = get_redis_connection('verify_code')
   	 # 提取图形验证码
   	 image_code_server = redis_conn.get('img_%s' % uuid)
    	 if image_code_server is None:
    		# 图形验证码过期或者不存在
    		return http.JsonResponse({'code': RETCODE.IMAGECODEERR, 'errmsg': '图形验证码失
   效'})
    		# 删除图形验证码，避免恶意测试图形验证码
    		try:
    			redis_conn.delete('img_%s' % uuid)
    		except Exception as e:
    			logger.error(e)
   		 # 对⽐图形验证码
    		image_code_server = image_code_server.decode() # bytes转字符串
    		if image_code_client.lower() != image_code_server.lower(): # 转⼩写后⽐较
   		 return http.JsonResponse({'code': RETCODE.IMAGECODEERR, 'errmsg': '输⼊图形验证码
   有误'})
   		 # ⽣成短信验证码：⽣成6位数验证码
            sms_code = '%06d' % random.randint(0, 999999)
    		 logger.info(sms_code)
    		 # 保存短信验证码
    		redis_conn.setex('sms_%s' % mobile, 								constants.SMS_CODE_REDIS_EXPIRES, sms_code)
    		# 发送短信验证码
    		sdk = SmsSDK(accId, accToken, appId)
    		tid = '1'
    		mobile = mobile
    		datas = (sms_code, '5')
    		resp = sdk.sendMessage(tid, mobile, datas)
    		print(resp)
    		# 响应结果
    		return http.JsonResponse({'code': RETCODE.OK, 'errmsg': '发送短信成功'})                      
   ```

4.9.3 必不可少的注意点：

1. 避免频繁发送短信验证码

```python
'''
1. 提取，校验sebd_flag
2. 重新写入send_flag
3. 界面渲染频繁发送短信信息提示
'''
# 提取，校验sebd_flag
send_flag = redis_conn.get('send_flag_%s' % mobile)
if send_flag:
 return http.JsonResponse({'code': RETCODE.THROTTLINGERR, 'errmsg': '发送短信过于频繁'})

# 重新写入send_flag
# 保存短信验证码
redis_conn.setex('sms_%s' % mobile, constants.SMS_CODE_REDIS_EXPIRES, sms_code)
# 重新写⼊send_flag
redis_conn.setex('send_flag_%s' % mobile, constants.SEND_SMS_CODE_INTERVAL, 1)

# .界⾯渲染频繁发送短信提示信息
if (response.data.code == '4001') {
 this.error_image_code_message = response.data.errmsg;
 this.error_image_code = true;
} else { // 4002
 this.error_sms_code_message = response.data.errmsg;
 this.error_sms_code = true;
}
```

4.9.4 pipeline操作Redis数据库

（1）Redis的 C - S 架构：

* 基于客户端-服务端模型以及请求/响应协议的TCP服务。
*  客户端向服务端发送⼀个查询请求，并监听Socket返回。
* 通常是以阻塞模式，等待服务端响应。
*  服务端处理命令，并将结果返回给客户端。

（2）存在的问题： 

* 如果Redis服务端需要同时处理多个请求，加上⽹络延迟，那么服务端利⽤率不⾼，效率降低。

（3）解决的办法：

* 管道pipeline

```
pipeline简介：
管道pipeline
* 可以⼀次性发送多条命令并在执⾏完后⼀次性将结果返回。
* pipeline通过减少客户端与Redis的通信次数来实现降低往返延时时间。
实现的原理
* 实现的原理是队列。
* Client可以将三个命令放到⼀个tcp报⽂⼀起发送。
* Server则可以将三条命令的处理结果放到⼀个tcp报⽂返回。
* 队列是先进先出，这样就保证数据的顺序性。
```

2. pipeline操作步骤Redis数据库

（1）实现步骤

* 创建Redis管道
* 将Redis请求添加队列
* 执行请求

（2）代码实现

```python
# 创建Redis管道
pl = redis_conn.pipeline()
# 将Redis请求添加到队列
pl.setex('sms_%s' % mobile, constants.SMS_CODE_REDIS_EXPIRES, sms_code)
pl.setex('send_flag_%s' % mobile, constants.SEND_SMS_CODE_INTERVAL, 1)
# 执⾏请求
pl.execute()
```

### 五、异步方案

#### 5.1 生产者消费者设计模式

问题：

* 现行代码是在上而下同步执行
* 发送短信是耗时操作，如果短信被阻塞，用户响应将会延迟
* 响应延迟会造成用户界面的倒计时延时

解决办法：

* 异步发送短信
* 发送短信和响应分开执行，将 ‘’发送短信‘’ 从业务中 ‘’ 解耦‘’ 出来

#### 5.2 逻辑实现

* 了解celery
* 创建celery实例并加载配置
* 定义发送短信任务
* 启动celery服务
* 调用发送短信任务

### 六、账号登陆

#### 6.1 接口设计

* 请求方式
* 请求参数
* 响应结果

#### 6.2 用户名登陆接口定义

#### 6.3 用户名登陆后端逻辑

#### 6.4 实现多账号登录

```python
# 根据username字段登录
'''
1. 先验证是手机号和用户名
2. 异常处理
3. 找到用户对象后验证密码是否正确,验证活跃状态
4.配置自定义的用户认证 AUTHENTICATION_BACKENDS = ["apps.users.utils.UserMobileBackend"]
方法：1. 继承
	 2. 封装；有点：# 封装的思想：降低耦合度，高内聚低耦合，提高重用性
	 3. Q对象实现或的关系 导入：from django.db.models import Q
'''
```

登录与非登录的页面

方法一：Ajax渲染

方法二：vue获取cookie中用户名，需要将username保存在cookie中。

------

### 补充知识点

（1）HTTP发起请求，客户端向浏览器发起请求的方式：

* 在路径的特殊位置进行捕获
* 以查询字符串的方式
* 表单，JSON
* 在头部header传递

























