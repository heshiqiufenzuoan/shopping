一、项目准备
 1、导入虚拟环境：pip install virtualenv
 2、在导入的虚拟环境中创建虚拟环境：mkvirtualenv  -p python shopping_vir
 3、查看是否创建成功虚拟环境：workon
 4、使用虚拟环境: workon shopping_virtual

二、创建商城功能模块
* 创建商城项目：django-admin startproject shopping
* 创建APP：python manage.py startapp  libs[存放第三方包]/log[]/static[静态资源：图片等]/templates[HTML文件等模块]/utils[公共类]

三、模块介绍：
此压缩包包含8个子文件，以下是各个问价夹对应的模块功能介绍
* 1、apps:
     *里面包含8个子文件：
          * area：收货地址；
          carts：购物车；
          contents：广告页面，轮播图；
          goods：商品；
          orders：订单界面
          payment：支付界面
          user：用户登陆界面
          verifications：图形验证码界面
 2、celery_tasks：
       里面包含两个文件：
          emails：邮箱信息；
          sms：短信验证码；
 3、libs：验证码包
 4、logs：记录日志；
 5、shopping：setting.py、urls.py等；
 6、static：css，图片等；
 7、templates：渲染页面；
 8、utils：公共方法，统一管理工具；
 
