---
layout: post
title: "Python Web开发-Django入门详解"
date: 2019-03-10 
description: "Django基础、模型、视图、模板、管理后台、URL配置、表单处理"
tag: Python

---

## Django框架的重要性

Django是一个功能强大的Python Web框架，采用"电池包含"的设计哲学，提供了完整的Web开发解决方案。掌握Django对于构建复杂的企业级Web应用至关重要，它提供了ORM、模板系统、管理后台、安全性等丰富的功能。

## 项目创建和结构

### 1. 项目初始化

```bash
# 创建Django项目
django-admin startproject myproject
cd myproject

# 创建应用
python manage.py startapp myapp

# 数据库迁移
python manage.py makemigrations
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser

# 启动开发服务器
python manage.py runserver
```

### 2. 项目结构说明

```python
def django_structure_demo():
    """Django项目结构演示"""
    print("=== Django项目结构 ===")
    print("myproject/")
    print("├── manage.py              # Django管理脚本")
    print("├── myproject/")
    print("│   ├── __init__.py")
    print("│   ├── settings.py       # 项目配置")
    print("│   ├── urls.py          # URL路由配置")
    print("│   ├── wsgi.py          # WSGI配置")
    print("│   └── asgi.py          # ASGI配置")
    print("└── myapp/")
    print("    ├── __init__.py")
    print("    ├── admin.py         # 管理后台配置")
    print("    ├── apps.py          # 应用配置")
    print("    ├── models.py        # 数据模型")
    print("    ├── views.py         # 视图函数")
    print("    ├── urls.py          # URL配置")
    print("    ├── tests.py         # 测试文件")
    print("    └── migrations/      # 数据库迁移文件")

django_structure_demo()
```

## 模型（Models）设计

### 1. 基础模型定义

```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse

class Category(models.Model):
    """文章分类模型"""
    name = models.CharField(max_length=100, verbose_name="分类名称")
    description = models.TextField(blank=True, verbose_name="分类描述")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    
    class Meta:
        verbose_name = "分类"
        verbose_name_plural = "分类"
        ordering = ['name']
    
    def __str__(self):
        return self.name

class Tag(models.Model):
    """标签模型"""
    name = models.CharField(max_length=50, unique=True, verbose_name="标签名称")
    color = models.CharField(max_length=7, default="#007bff", verbose_name="标签颜色")
    
    def __str__(self):
        return self.name
class Article(models.Model):
    """文章模型"""
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('published', '已发布'),
        ('archived', '已归档'),
    ]
    
    title = models.CharField(max_length=200, verbose_name="标题")
    slug = models.SlugField(max_length=200, unique=True, verbose_name="URL别名")
    content = models.TextField(verbose_name="内容")
    excerpt = models.TextField(max_length=500, blank=True, verbose_name="摘要")
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="作者")
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, verbose_name="分类")
    tags = models.ManyToManyField(Tag, blank=True, verbose_name="标签")
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft', verbose_name="状态")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新时间")
    published_at = models.DateTimeField(null=True, blank=True, verbose_name="发布时间")
    view_count = models.PositiveIntegerField(default=0, verbose_name="浏览次数")
    
    class Meta:
        verbose_name = "文章"
        verbose_name_plural = "文章"
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        """获取文章详情页URL"""
        return reverse('article_detail', kwargs={'slug': self.slug})
    
    def save(self, *args, **kwargs):
        """保存时自动设置发布时间"""
        if self.status == 'published' and not self.published_at:
            from django.utils import timezone
            self.published_at = timezone.now()
        super().save(*args, **kwargs)
```

### 2. 模型关系和方法

```python
class Comment(models.Model):
    """评论模型"""
    article = models.ForeignKey(Article, on_delete=models.CASCADE, related_name='comments', verbose_name="文章")
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="评论者")
    content = models.TextField(verbose_name="评论内容")
    parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, verbose_name="父评论")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="评论时间")
    is_approved = models.BooleanField(default=False, verbose_name="是否审核通过")
    
    class Meta:
        verbose_name = "评论"
        verbose_name_plural = "评论"
        ordering = ['created_at']
    
    def __str__(self):
        return f"{self.author.username} 对 {self.article.title} 的评论"
    
    def get_replies(self):
        """获取回复"""
        return Comment.objects.filter(parent=self)

def model_demo():
    """模型演示"""
    print("=== Django模型设计演示 ===")
    print("模型特性:")
    print("- 自动数据库表生成")
    print("- 字段类型丰富")
    print("- 关系映射")
    print("- 元数据配置")
    print("- 自定义方法")
    print("- 信号机制")

model_demo()
```

## 视图（Views）设计

### 1. 函数式视图

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.http import JsonResponse, HttpResponse
from django.core.paginator import Paginator
from django.db.models import Q
from .models import Article, Category, Tag, Comment
from .forms import ArticleForm, CommentForm

def article_list(request):
    """文章列表视图"""
    # 获取查询参数
    search = request.GET.get('search', '')
    category_id = request.GET.get('category', '')
    
    # 构建查询
    articles = Article.objects.filter(status='published')
    
    if search:
        articles = articles.filter(
            Q(title__icontains=search) | 
            Q(content__icontains=search)
        )
    
    if category_id:
        articles = articles.filter(category_id=category_id)
    
    # 分页
    paginator = Paginator(articles, 10)  # 每页10篇文章
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    # 获取分类和标签用于筛选
    categories = Category.objects.all()
    tags = Tag.objects.all()
    
    context = {
        'page_obj': page_obj,
        'articles': page_obj.object_list,
        'categories': categories,
        'tags': tags,
        'search': search,
        'selected_category': category_id,
    }
    
    return render(request, 'articles/list.html', context)
```

### 2. 类视图

```python
from django.views.generic import ListView, DetailView, CreateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from django.contrib import messages

class ArticleListView(ListView):
    """文章列表类视图"""
    model = Article
    template_name = 'articles/list.html'
    context_object_name = 'articles'
    paginate_by = 10
    
    def get_queryset(self):
        """获取查询集"""
        queryset = Article.objects.filter(status='published')
        
        # 搜索
        search = self.request.GET.get('search')
        if search:
            queryset = queryset.filter(
                Q(title__icontains=search) | 
                Q(content__icontains=search)
            )
        
        return queryset.select_related('author', 'category').prefetch_related('tags')
    
    def get_context_data(self, **kwargs):
        """添加上下文数据"""
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        context['tags'] = Tag.objects.all()
        context['search'] = self.request.GET.get('search', '')
        return context
class ArticleDetailView(DetailView):
    """文章详情类视图"""
    model = Article
    template_name = 'articles/detail.html'
    context_object_name = 'article'
    
    def get_queryset(self):
        return Article.objects.filter(status='published')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        article = context['article']
        
        # 增加浏览次数
        article.view_count += 1
        article.save(update_fields=['view_count'])
        
        # 相关文章
        context['related_articles'] = Article.objects.filter(
            category=article.category,
            status='published'
        ).exclude(id=article.id)[:5]
        
        return context
```

## 管理后台配置

### 1. 基础管理配置

```python
from django.contrib import admin
from .models import Article, Category, Tag, Comment

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    """分类管理"""
    list_display = ['name', 'created_at']
    search_fields = ['name']
    list_filter = ['created_at']
@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    """标签管理"""
    list_display = ['name', 'color']
    search_fields = ['name']
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    """文章管理"""
    list_display = ['title', 'author', 'category', 'status', 'created_at', 'view_count']
    list_filter = ['status', 'category', 'created_at', 'author']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    filter_horizontal = ['tags']
    date_hierarchy = 'created_at'
    
    fieldsets = (
        ('基本信息', {
            'fields': ('title', 'slug', 'author', 'category')
        }),
        ('内容', {
            'fields': ('content', 'excerpt', 'tags')
        }),
        ('发布设置', {
            'fields': ('status', 'published_at')
        }),
        ('统计信息', {
            'fields': ('view_count',),
            'classes': ('collapse',)
        }),
    )
    
    def get_queryset(self, request):
        """自定义查询集"""
        return super().get_queryset(request).select_related('author', 'category')
@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    """评论管理"""
    list_display = ['author', 'article', 'created_at', 'is_approved']
    list_filter = ['is_approved', 'created_at']
    search_fields = ['content', 'author__username']
    actions = ['approve_comments', 'disapprove_comments']
    
    def approve_comments(self, request, queryset):
        """批量审核评论"""
        queryset.update(is_approved=True)
        self.message_user(request, f'已审核 {queryset.count()} 条评论')
    approve_comments.short_description = "审核选中的评论"
    
    def disapprove_comments(self, request, queryset):
        """批量取消审核"""
        queryset.update(is_approved=False)
        self.message_user(request, f'已取消审核 {queryset.count()} 条评论')
    disapprove_comments.short_description = "取消审核选中的评论"
```

## 总结

Django入门的关键要点：

1. **项目结构**：清晰的文件组织和模块化设计
2. **模型设计**：ORM映射、关系定义、字段类型
3. **视图处理**：函数视图和类视图的使用场景
4. **模板系统**：模板继承、标签、过滤器
5. **URL配置**：路由管理和参数传递
6. **管理后台**：快速数据管理界面
7. **表单处理**：数据验证和用户交互

掌握这些Django基础概念，可以快速构建功能完整的Web应用，为深入学习Django高级特性打下坚实基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-Django入门详解](http://zhouzhiyang.cn/2019/03/Python_Web_Django_Intro/) 

