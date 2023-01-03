---
layout: post
title: 软工项目心得
date: 2023-01-03
Author: Ursula 
tags: [Django，项目]
comments: true
--- 

<font color='blue'>**软工鸡汤**：千万不要从第一行代码开始写项目，复用好框架和参考项目，让你事半功倍</font>

## Django

Django为什么香？

使用Django写BS架构的项目非常方便，它把数据库中的table封装成Model数据对象，与数据库交互的后端部分，直接写成对Model操作的函数，不再用写繁琐的SQL语句，调用繁琐的接口和HTTP请求

静态的前端页面还是要自己写，但借助Form对象可以很容易的实现进行增删改查的表单和处理表单的逻辑，比用JavaScript写要简单很多

Django学习门槛很低，大家都能上手，很适合用来做小组项目

### Django架构

首先创建一个新项目
>  django-admin startproject +项目名

在在项目中创建APP，Django会为每个APP创建一个文件夹

创建应用
>  python manage.py startapp +app_name

项目目录结构

<img src="https://user-images.githubusercontent.com/73097943/210296321-3bea407a-5879-40d6-b592-ab81700afe39.png" alt="img" style="zoom:40%;" />

- **settings.py** 包含所有的网站设置。这是可以注册所有创建的应用的地方，也是静态文件，数据库配置的地方，等等。
- **urls.py** 定义了网站 url 到 view 的映射。虽然这里可以包含所有的 url，但是更常见的做法是把应用相关的 url 包含在相关应用中。
- **wsgi.py** 帮助 Django 应用和网络服务器间的通讯。

APP文件夹内的目录结构

<img src="https://user-images.githubusercontent.com/73097943/210297550-3a486706-7ca0-4edf-b968-b8ad0ef9681f.png" alt="img" style="zoom:40%;" />

### 后端（数据库）

#### 数据模型 

Django中的Model类就相当于数据表的表头，其属性就是表的字段，字段类型（包括键）也能用Django提供的类表示，对象（实例）就相当于表中的一行记录，主键ID字段不用定义，自动生成自增的ID

在内部类Meta定义别名、提示语等属性 

Model对象有对应删改查的方法，filter过滤器，增就是创建对象

`__str__()`，来为每个物件返回一个人类可读的字符串。

model_name.attribute/foreign_key

Model有比较特殊的ManyToManyField()，和普通的数据库字段不同，一条记录的该字段中可添加多个元素值，除此之外还有密码等专用字段


创建（数据模型）实例

实例可以在model.py中直接创建对象，也可以在./admin网页中添加

./admin网页中数据项列表中显示的值是`__str__()`方法的返回值

`VIEW ON SITE`按钮对应`get_absolute_url()`方法，链接到该对象的URL

Manage类

用来管理Model的类，其方法（filter）可以实现数据查找等功能

### 前端
Django中前后端交互的逻辑结构图：

<img src="https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Home_page/basic-django.png" alt="img" />

template文件夹下存放HTML和CSS写出的静态界面

urls.py文件中编写路由，建立从url（网址）到HTML文件（路径）的映射

views.py文件中编写与前端和后端交互的逻辑（从前端读数据，处理一下给后端）

![image](https://user-images.githubusercontent.com/73097943/210301485-d94b0dbb-172c-4f23-9349-e97be09aa07c.png)


#### 表单

Form 

ModelForm，专门为增删改查模型建立的表单

在view.py中编写表单处理函数：

HTTP request作为默认参数传给view中的函数

if request==GET，定位到HTML文件并渲染页面，可以把modelForm对象传到HTML文件中，用{{form}}引用，Django就会自动帮你渲染出表单

if request==POST，提交表单，调用modelForm对象的is_valid方法检查表单是否符合字段约束，save方法保存，保存前可以手动填写字段，使用initial参数初始化

合法性检验

is_valid是对整个表单

对具体字段

```python
for field in product_add_form:
    if field.errors:
        return HttpResponse(field.errors)
return HttpResponse("输入格式不正确，请重新输入")
```

field.errors展示的报错提示是在模型中定义的error_message字段

错误展示有两种方式

1. HttpRespond，跳转页面，很丑

2. 使用bootstrap组件，在表单HTML中加上

![image](https://user-images.githubusercontent.com/73097943/210301569-f1b1d3b6-eaf4-4722-8b42-8231e9b64269.png)

除了在模型中定义error_message，也可以在view.py的函数中定义message

```python
messages.success(request,"宝贝已在心愿清单中了哟")
```

再在HTML文件中使用

![image](https://user-images.githubusercontent.com/73097943/210301648-119f3781-4271-4742-b3f7-c971725467a3.png)


### 管理界面

Django提供了一个可任意对所有APP中的数据（Model）进行增删改查的管理界面，可借助一些python库自动美化管理页面

注册模型

数据模型需要在admin.py中定义 [ModelAdmin](https://docs.djangoproject.com/en/dev/ref/contrib/admin/#modeladmin-objects) 类并注册，才能在./admin页面上显示

创建超级用户

> python manage.py createsuperuser

运行网站

> python manage.py runserver


Notice：

1. 图片

填在表单中的图片是以文件的形式上传的，保存在media文件夹下，数据库中存的是图片的url，在接收表单是要设置文件参数

```python
ProductImageUpdateForm(data=request.POST, files=request.FILES)
```

在HTML文件中要把表单的编码方式定义成Multipart

```html
<form enctype="multipart/form-data" class="account-form" action="" method="post">
```

2. 实现商品搜索

建立表单让用户输入搜索的字段，在展示商品前用filter(name=request.GET['search_text'])即可

3. 更新Model后别忘了数据库迁移

*migration*文件夹，用来存储“migrations”——当你修改你的数据模型时，这个文件会自动升级你的数据库。

<font color="red>**每次模型改变都要运行以下命令**<\font>

> python manage.py makemigrations
> python manage.py migrate
             
             
## 其他的好用的库

### isort（python）

自动整理import语句

自动按类型（标准库/第三方库/自己的模块/……）划分部分

### MPTT（python）

使用树模型分层管理数据

创建Model的时候可以继承MPTTModel类

eg：

```python
  class Category(MPTTModel):
    """
    Category Table implimented with MPTT.
    """

    name = models.CharField(
        verbose_name=_("类别名"),
        help_text=_("Required and unique"),
        max_length=255,
        unique=True,
    )
    brief = models.CharField(
        verbose_name=_("类别简介"),
        max_length=1000,
        null=True,
    )
    slug = models.SlugField(verbose_name=_("安全URL"), max_length=255, 
        help_text=_("在网页中定位到商品的唯一英文字符"), unique=True)
    parent = TreeForeignKey("self", on_delete=models.CASCADE, null=True, blank=True, related_name="children")
    is_active = models.BooleanField(default=True)

    class MPTTMeta:
        order_insertion_by = ["name"]

    class Meta:
        verbose_name = _("类别")
        verbose_name_plural = _("类别")

    def get_absolute_url(self):
        return reverse("catalogue:category_list", args=[self.slug])

    def __str__(self):
        return self.name


### bootstrap5

这次用到的主要是包内封装的渲染固定class的CSS，以及布局组件

表单可以使用bootstrap的CSS美化，用form.索引具体的字段

