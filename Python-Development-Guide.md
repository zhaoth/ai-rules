# Python 3.11+ 现代开发规范

你是 Python、FastAPI、Django、Flask、SQLAlchemy、Pydantic、Poetry、Docker 和现代 Python 生态系统的专家，深度理解这些技术的最佳实践和性能优化技巧。

## 代码风格和结构

- 编写简洁、可维护、技术准确的 Python 代码，遵循 PEP 8 规范
- 使用函数式和面向对象编程模式，根据场景选择合适的范式
- 优先使用类型提示和数据类，遵循 DRY 原则，避免代码重复
- 使用描述性变量名和函数名（如 is_valid、has_permission、get_user_by_id）
- 系统化组织模块：每个文件只包含相关功能，如模型定义、业务逻辑、工具函数和类型定义

## 命名约定

- 包和模块使用小写加下划线（如 user_management、api_client）
- 类名使用 PascalCase（如 UserService、DatabaseConnection）
- 函数和变量使用 snake_case（如 get_user_data、is_authenticated）
- 常量使用大写加下划线（如 MAX_RETRY_COUNT、DEFAULT_TIMEOUT）

## 类型提示使用

- 所有公共函数和方法必须包含类型提示
- 使用 `typing` 模块的现代语法（Python 3.9+ 内置类型）
- 优先使用 Pydantic 模型进行数据验证和序列化
- 使用 Protocol 定义接口，提高代码的可测试性

## 语法和格式

- 使用 f-string 进行字符串格式化
- 优先使用列表推导式和生成器表达式
- 使用上下文管理器处理资源管理
- 遵循 Black 代码格式化标准

## 框架和库选择

- Web 框架：FastAPI（API 开发）、Django（全栈应用）、Flask（轻量级应用）
- 数据库 ORM：SQLAlchemy 2.0+、Django ORM、Tortoise ORM
- 数据验证：Pydantic v2、Marshmallow
- 异步编程：asyncio、aiohttp、asyncpg
- 测试框架：pytest、unittest、hypothesis

## 性能优化

- 使用适当的数据结构（set、dict、deque）
- 实现数据库查询优化和连接池
- 使用缓存策略（Redis、Memcached）
- 异步编程处理 I/O 密集型任务
- 使用 profiling 工具识别性能瓶颈

## 核心约定

- 使用 Poetry 进行依赖管理和虚拟环境
- 实施完整的错误处理和日志记录
- 编写全面的单元测试和集成测试
- 使用 Docker 进行容器化部署

## Python 最佳实践

### 项目结构
```
my_project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── main.py
│       ├── models/
│       │   ├── __init__.py
│       │   └── user.py
│       ├── services/
│       │   ├── __init__.py
│       │   └── user_service.py
│       ├── api/
│       │   ├── __init__.py
│       │   └── routes/
│       └── utils/
├── tests/
├── docs/
├── pyproject.toml
├── README.md
└── .env.example
```

### 类型提示和数据模型
```python
from typing import Optional, List, Dict, Any
from datetime import datetime
from pydantic import BaseModel, Field, validator
from enum import Enum

class UserRole(str, Enum):
    ADMIN = "admin"
    USER = "user"
    MODERATOR = "moderator"

class UserBase(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: str = Field(..., regex=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    full_name: Optional[str] = None
    is_active: bool = True
    role: UserRole = UserRole.USER

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('密码必须包含大写字母')
        if not any(c.isdigit() for c in v):
            raise ValueError('密码必须包含数字')
        return v

class UserResponse(UserBase):
    id: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True
```

### 服务层设计
```python
from abc import ABC, abstractmethod
from typing import Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from .models import User
from .schemas import UserCreate, UserUpdate

class UserServiceProtocol(ABC):
    @abstractmethod
    async def create_user(self, user_data: UserCreate) -> User:
        pass
    
    @abstractmethod
    async def get_user_by_id(self, user_id: int) -> Optional[User]:
        pass

class UserService(UserServiceProtocol):
    def __init__(self, db_session: AsyncSession):
        self.db = db_session
    
    async def create_user(self, user_data: UserCreate) -> User:
        # 检查用户是否已存在
        existing_user = await self._get_user_by_email(user_data.email)
        if existing_user:
            raise ValueError("用户邮箱已存在")
        
        # 创建新用户
        db_user = User(**user_data.dict(exclude={'password'}))
        db_user.password_hash = self._hash_password(user_data.password)
        
        self.db.add(db_user)
        await self.db.commit()
        await self.db.refresh(db_user)
        
        return db_user
    
    async def get_user_by_id(self, user_id: int) -> Optional[User]:
        result = await self.db.get(User, user_id)
        return result
    
    async def _get_user_by_email(self, email: str) -> Optional[User]:
        from sqlalchemy import select
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
    
    def _hash_password(self, password: str) -> str:
        import bcrypt
        return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()
```

### 异步编程模式
```python
import asyncio
import aiohttp
from typing import List, Dict, Any
from contextlib import asynccontextmanager

class AsyncAPIClient:
    def __init__(self, base_url: str, timeout: int = 30):
        self.base_url = base_url
        self.timeout = aiohttp.ClientTimeout(total=timeout)
        self._session: Optional[aiohttp.ClientSession] = None
    
    @asynccontextmanager
    async def get_session(self):
        if self._session is None:
            self._session = aiohttp.ClientSession(
                timeout=self.timeout,
                headers={'User-Agent': 'MyApp/1.0'}
            )
        try:
            yield self._session
        finally:
            pass  # 保持会话开启以复用连接
    
    async def get(self, endpoint: str, params: Dict[str, Any] = None) -> Dict[str, Any]:
        async with self.get_session() as session:
            async with session.get(
                f"{self.base_url}/{endpoint}",
                params=params
            ) as response:
                response.raise_for_status()
                return await response.json()
    
    async def post(self, endpoint: str, data: Dict[str, Any]) -> Dict[str, Any]:
        async with self.get_session() as session:
            async with session.post(
                f"{self.base_url}/{endpoint}",
                json=data
            ) as response:
                response.raise_for_status()
                return await response.json()
    
    async def close(self):
        if self._session:
            await self._session.close()
            self._session = None

# 批量处理示例
async def process_users_batch(user_ids: List[int], batch_size: int = 10) -> List[Dict[str, Any]]:
    client = AsyncAPIClient("https://api.example.com")
    results = []
    
    try:
        # 分批处理，避免过多并发请求
        for i in range(0, len(user_ids), batch_size):
            batch = user_ids[i:i + batch_size]
            tasks = [
                client.get(f"users/{user_id}")
                for user_id in batch
            ]
            batch_results = await asyncio.gather(*tasks, return_exceptions=True)
            
            # 处理结果和异常
            for result in batch_results:
                if isinstance(result, Exception):
                    print(f"处理用户时出错: {result}")
                else:
                    results.append(result)
    
    finally:
        await client.close()
    
    return results
```

## Django 最佳实践

### Django 项目结构
```
my_django_project/
├── manage.py
├── requirements.txt
├── .env.example
├── my_project/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/
│   ├── __init__.py
│   ├── users/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── migrations/
│   └── core/
│       ├── __init__.py
│       ├── models.py
│       ├── permissions.py
│       └── utils.py
├── static/
├── media/
├── templates/
└── tests/
```

### Django 模型设计
```python
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.core.validators import MinLengthValidator
from django.utils.translation import gettext_lazy as _
import uuid

class TimeStampedModel(models.Model):
    """带时间戳的抽象基类"""
    created_at = models.DateTimeField(auto_now_add=True, verbose_name=_("创建时间"))
    updated_at = models.DateTimeField(auto_now=True, verbose_name=_("更新时间"))
    
    class Meta:
        abstract = True

class User(AbstractUser):
    """自定义用户模型"""
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    email = models.EmailField(unique=True, verbose_name=_("邮箱"))
    phone = models.CharField(
        max_length=20, 
        blank=True, 
        null=True, 
        verbose_name=_("手机号")
    )
    avatar = models.ImageField(
        upload_to='avatars/', 
        blank=True, 
        null=True,
        verbose_name=_("头像")
    )
    is_verified = models.BooleanField(default=False, verbose_name=_("已验证"))
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
    
    class Meta:
        verbose_name = _("用户")
        verbose_name_plural = _("用户")
        db_table = 'users'
    
    def __str__(self):
        return self.email

class Profile(TimeStampedModel):
    """用户资料"""
    user = models.OneToOneField(
        User, 
        on_delete=models.CASCADE, 
        related_name='profile'
    )
    bio = models.TextField(
        max_length=500, 
        blank=True, 
        verbose_name=_("个人简介")
    )
    birth_date = models.DateField(
        blank=True, 
        null=True, 
        verbose_name=_("生日")
    )
    location = models.CharField(
        max_length=100, 
        blank=True, 
        verbose_name=_("位置")
    )
    
    class Meta:
        verbose_name = _("用户资料")
        verbose_name_plural = _("用户资料")
    
    def __str__(self):
        return f"{self.user.email} 的资料"
```

### Django REST Framework 视图
```python
from rest_framework import generics, status, permissions
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework.pagination import PageNumberPagination
from django.contrib.auth import get_user_model
from django.db.models import Q
from .serializers import UserSerializer, UserCreateSerializer
from .permissions import IsOwnerOrReadOnly

User = get_user_model()

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100

class UserListCreateView(generics.ListCreateAPIView):
    """用户列表和创建视图"""
    queryset = User.objects.all()
    pagination_class = StandardResultsSetPagination
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_serializer_class(self):
        if self.request.method == 'POST':
            return UserCreateSerializer
        return UserSerializer
    
    def get_queryset(self):
        queryset = User.objects.select_related('profile')
        search = self.request.query_params.get('search')
        if search:
            queryset = queryset.filter(
                Q(username__icontains=search) | 
                Q(email__icontains=search) |
                Q(first_name__icontains=search) |
                Q(last_name__icontains=search)
            )
        return queryset.order_by('-date_joined')

class UserDetailView(generics.RetrieveUpdateDestroyAPIView):
    """用户详情视图"""
    queryset = User.objects.select_related('profile')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated, IsOwnerOrReadOnly]
    lookup_field = 'id'

@api_view(['GET'])
@permission_classes([permissions.IsAuthenticated])
def user_stats(request):
    """用户统计信息"""
    from django.db.models import Count
    
    stats = User.objects.aggregate(
        total_users=Count('id'),
        verified_users=Count('id', filter=Q(is_verified=True)),
        active_users=Count('id', filter=Q(is_active=True))
    )
    
    return Response(stats)
```

### Django 序列化器
```python
from rest_framework import serializers
from django.contrib.auth import get_user_model
from django.contrib.auth.password_validation import validate_password
from .models import Profile

User = get_user_model()

class ProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = Profile
        fields = ['bio', 'birth_date', 'location']

class UserSerializer(serializers.ModelSerializer):
    profile = ProfileSerializer(read_only=True)
    full_name = serializers.SerializerMethodField()
    
    class Meta:
        model = User
        fields = [
            'id', 'username', 'email', 'first_name', 'last_name',
            'full_name', 'avatar', 'is_verified', 'date_joined', 'profile'
        ]
        read_only_fields = ['id', 'date_joined', 'is_verified']
    
    def get_full_name(self, obj):
        return f"{obj.first_name} {obj.last_name}".strip()

class UserCreateSerializer(serializers.ModelSerializer):
    password = serializers.CharField(
        write_only=True,
        validators=[validate_password]
    )
    password_confirm = serializers.CharField(write_only=True)
    
    class Meta:
        model = User
        fields = [
            'username', 'email', 'first_name', 'last_name',
            'password', 'password_confirm'
        ]
    
    def validate(self, attrs):
        if attrs['password'] != attrs['password_confirm']:
            raise serializers.ValidationError("密码不匹配")
        return attrs
    
    def create(self, validated_data):
        validated_data.pop('password_confirm')
        user = User.objects.create_user(**validated_data)
        Profile.objects.create(user=user)
        return user
```

### Django 设置配置
```python
# settings/base.py
import os
from pathlib import Path
from decouple import config

BASE_DIR = Path(__file__).resolve().parent.parent.parent

# 安全设置
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='').split(',')

# 应用配置
DJANGO_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

THIRD_PARTY_APPS = [
    'rest_framework',
    'corsheaders',
    'django_filters',
    'drf_spectacular',
]

LOCAL_APPS = [
    'apps.users',
    'apps.core',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'my_project.urls'

# 数据库配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST', default='localhost'),
        'PORT': config('DB_PORT', default='5432'),
        'OPTIONS': {
            'charset': 'utf8mb4',
        },
    }
}

# REST Framework 配置
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# 自定义用户模型
AUTH_USER_MODEL = 'users.User'

# 国际化
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_TZ = True

# 静态文件
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

## Flask 最佳实践

### Flask 应用结构
```
my_flask_app/
├── app/
│   ├── __init__.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── users.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── user_service.py
│   ├── utils/
│   │   ├── __init__.py
│   │   └── helpers.py
│   └── extensions.py
├── migrations/
├── tests/
├── config.py
├── requirements.txt
├── .env.example
└── run.py
```

### Flask 应用工厂
```python
# app/__init__.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager
from flask_cors import CORS
from flask_marshmallow import Marshmallow
from config import Config

# 扩展实例
db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
cors = CORS()
ma = Marshmallow()

def create_app(config_class=Config):
    """应用工厂函数"""
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    # 初始化扩展
    db.init_app(app)
    migrate.init_app(app, db)
    jwt.init_app(app)
    cors.init_app(app)
    ma.init_app(app)
    
    # 注册蓝图
    from app.api import bp as api_bp
    app.register_blueprint(api_bp, url_prefix='/api/v1')
    
    # 错误处理
    from app.errors import bp as errors_bp
    app.register_blueprint(errors_bp)
    
    return app

# app/extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager
from flask_cors import CORS
from flask_marshmallow import Marshmallow

db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
cors = CORS()
ma = Marshmallow()
```

### Flask 模型定义
```python
# app/models/user.py
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash
from flask_sqlalchemy import SQLAlchemy
from app.extensions import db

class TimestampMixin:
    """时间戳混入类"""
    created_at = db.Column(db.DateTime, default=datetime.utcnow, nullable=False)
    updated_at = db.Column(
        db.DateTime, 
        default=datetime.utcnow, 
        onupdate=datetime.utcnow,
        nullable=False
    )

class User(TimestampMixin, db.Model):
    """用户模型"""
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False, index=True)
    email = db.Column(db.String(120), unique=True, nullable=False, index=True)
    password_hash = db.Column(db.String(255), nullable=False)
    first_name = db.Column(db.String(50))
    last_name = db.Column(db.String(50))
    is_active = db.Column(db.Boolean, default=True, nullable=False)
    is_verified = db.Column(db.Boolean, default=False, nullable=False)
    
    # 关系
    profile = db.relationship('Profile', backref='user', uselist=False, cascade='all, delete-orphan')
    
    def __init__(self, username, email, password, **kwargs):
        self.username = username
        self.email = email
        self.set_password(password)
        for key, value in kwargs.items():
            setattr(self, key, value)
    
    def set_password(self, password):
        """设置密码哈希"""
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        """验证密码"""
        return check_password_hash(self.password_hash, password)
    
    @property
    def full_name(self):
        """获取全名"""
        return f"{self.first_name or ''} {self.last_name or ''}".strip()
    
    def to_dict(self):
        """转换为字典"""
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'full_name': self.full_name,
            'is_active': self.is_active,
            'is_verified': self.is_verified,
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat()
        }
    
    def __repr__(self):
        return f'<User {self.username}>'

class Profile(TimestampMixin, db.Model):
    """用户资料模型"""
    __tablename__ = 'profiles'
    
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    bio = db.Column(db.Text)
    avatar_url = db.Column(db.String(255))
    birth_date = db.Column(db.Date)
    location = db.Column(db.String(100))
    
    def to_dict(self):
        return {
            'bio': self.bio,
            'avatar_url': self.avatar_url,
            'birth_date': self.birth_date.isoformat() if self.birth_date else None,
            'location': self.location
        }
```

### Flask API 视图
```python
# app/api/users.py
from flask import Blueprint, request, jsonify, current_app
from flask_jwt_extended import jwt_required, get_jwt_identity, create_access_token
from marshmallow import ValidationError
from sqlalchemy.exc import IntegrityError
from app.extensions import db
from app.models.user import User
from app.schemas.user import UserSchema, UserCreateSchema
from app.services.user_service import UserService

bp = Blueprint('users', __name__)
user_schema = UserSchema()
users_schema = UserSchema(many=True)
user_create_schema = UserCreateSchema()

@bp.route('/users', methods=['POST'])
def create_user():
    """创建用户"""
    try:
        # 验证请求数据
        user_data = user_create_schema.load(request.json)
    except ValidationError as err:
        return jsonify({'errors': err.messages}), 400
    
    try:
        # 创建用户
        user_service = UserService()
        user = user_service.create_user(user_data)
        
        return jsonify({
            'message': '用户创建成功',
            'user': user_schema.dump(user)
        }), 201
        
    except ValueError as e:
        return jsonify({'error': str(e)}), 400
    except IntegrityError:
        db.session.rollback()
        return jsonify({'error': '用户名或邮箱已存在'}), 409

@bp.route('/users', methods=['GET'])
@jwt_required()
def get_users():
    """获取用户列表"""
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 20, type=int), 100)
    search = request.args.get('search', '')
    
    query = User.query
    
    if search:
        query = query.filter(
            db.or_(
                User.username.contains(search),
                User.email.contains(search),
                User.first_name.contains(search),
                User.last_name.contains(search)
            )
        )
    
    pagination = query.paginate(
        page=page, 
        per_page=per_page, 
        error_out=False
    )
    
    return jsonify({
        'users': users_schema.dump(pagination.items),
        'pagination': {
            'page': page,
            'pages': pagination.pages,
            'per_page': per_page,
            'total': pagination.total,
            'has_next': pagination.has_next,
            'has_prev': pagination.has_prev
        }
    })

@bp.route('/users/<int:user_id>', methods=['GET'])
@jwt_required()
def get_user(user_id):
    """获取用户详情"""
    user = User.query.get_or_404(user_id)
    return jsonify(user_schema.dump(user))

@bp.route('/users/me', methods=['GET'])
@jwt_required()
def get_current_user():
    """获取当前用户信息"""
    current_user_id = get_jwt_identity()
    user = User.query.get_or_404(current_user_id)
    return jsonify(user_schema.dump(user))

@bp.route('/users/<int:user_id>', methods=['PUT'])
@jwt_required()
def update_user(user_id):
    """更新用户信息"""
    current_user_id = get_jwt_identity()
    
    # 检查权限
    if current_user_id != user_id:
        return jsonify({'error': '无权限修改其他用户信息'}), 403
    
    user = User.query.get_or_404(user_id)
    
    try:
        user_data = user_schema.load(request.json, partial=True)
        
        for key, value in user_data.items():
            if hasattr(user, key):
                setattr(user, key, value)
        
        db.session.commit()
        
        return jsonify({
            'message': '用户信息更新成功',
            'user': user_schema.dump(user)
        })
        
    except ValidationError as err:
        return jsonify({'errors': err.messages}), 400
```

### Flask 序列化器
```python
# app/schemas/user.py
from marshmallow import Schema, fields, validate, validates, ValidationError
from app.models.user import User

class UserSchema(Schema):
    """用户序列化器"""
    id = fields.Int(dump_only=True)
    username = fields.Str(
        required=True,
        validate=validate.Length(min=3, max=80)
    )
    email = fields.Email(required=True)
    first_name = fields.Str(validate=validate.Length(max=50))
    last_name = fields.Str(validate=validate.Length(max=50))
    full_name = fields.Str(dump_only=True)
    is_active = fields.Bool(dump_only=True)
    is_verified = fields.Bool(dump_only=True)
    created_at = fields.DateTime(dump_only=True)
    updated_at = fields.DateTime(dump_only=True)
    
    @validates('username')
    def validate_username(self, value):
        if User.query.filter_by(username=value).first():
            raise ValidationError('用户名已存在')
    
    @validates('email')
    def validate_email(self, value):
        if User.query.filter_by(email=value).first():
            raise ValidationError('邮箱已存在')

class UserCreateSchema(UserSchema):
    """用户创建序列化器"""
    password = fields.Str(
        required=True,
        validate=validate.Length(min=8),
        load_only=True
    )
    password_confirm = fields.Str(
        required=True,
        load_only=True
    )
    
    @validates('password')
    def validate_password(self, value):
        if not any(c.isupper() for c in value):
            raise ValidationError('密码必须包含大写字母')
        if not any(c.isdigit() for c in value):
            raise ValidationError('密码必须包含数字')
    
    def validate(self, data, **kwargs):
        errors = {}
        if data.get('password') != data.get('password_confirm'):
            errors['password_confirm'] = ['密码确认不匹配']
        
        if errors:
            raise ValidationError(errors)
        
        return data
```

### Flask 配置管理
```python
# config.py
import os
from datetime import timedelta

class Config:
    """基础配置"""
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-secret-key'
    
    # 数据库配置
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'postgresql://user:password@localhost/myapp'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SQLALCHEMY_ENGINE_OPTIONS = {
        'pool_size': 20,
        'pool_recycle': 3600,
        'pool_pre_ping': True
    }
    
    # JWT 配置
    JWT_SECRET_KEY = os.environ.get('JWT_SECRET_KEY') or 'jwt-secret-string'
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=1)
    JWT_REFRESH_TOKEN_EXPIRES = timedelta(days=30)
    
    # 邮件配置
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 587)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    
    # Redis 配置
    REDIS_URL = os.environ.get('REDIS_URL') or 'redis://localhost:6379/0'
    
    # 文件上传
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB
    UPLOAD_FOLDER = os.path.join(os.path.dirname(__file__), 'uploads')

class DevelopmentConfig(Config):
    """开发环境配置"""
    DEBUG = True
    SQLALCHEMY_ECHO = True

class ProductionConfig(Config):
    """生产环境配置"""
    DEBUG = False
    
    # 安全设置
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'

class TestingConfig(Config):
    """测试环境配置"""
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

## FastAPI 最佳实践

### API 路由设计
```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from .database import get_db_session
from .services import UserService
from .schemas import UserCreate, UserResponse
from .auth import verify_token

app = FastAPI(
    title="用户管理 API",
    description="现代化的用户管理系统",
    version="1.0.0"
)

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db_session)
) -> User:
    token = credentials.credentials
    user_id = verify_token(token)
    if not user_id:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="无效的认证令牌"
        )
    
    user_service = UserService(db)
    user = await user_service.get_user_by_id(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="用户不存在"
        )
    
    return user

@app.post("/users/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db_session)
):
    """创建新用户"""
    user_service = UserService(db)
    
    try:
        user = await user_service.create_user(user_data)
        return UserResponse.from_orm(user)
    except ValueError as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(e)
        )

@app.get("/users/me", response_model=UserResponse)
async def get_current_user_info(
    current_user: User = Depends(get_current_user)
):
    """获取当前用户信息"""
    return UserResponse.from_orm(current_user)

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db_session)
):
    """根据ID获取用户信息"""
    user_service = UserService(db)
    user = await user_service.get_user_by_id(user_id)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="用户不存在"
        )
    
    return UserResponse.from_orm(user)
```

### 数据库配置
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from datetime import datetime
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql+asyncpg://user:pass@localhost/db")

engine = create_async_engine(
    DATABASE_URL,
    echo=True,  # 开发环境启用SQL日志
    pool_size=20,
    max_overflow=0,
    pool_pre_ping=True,
    pool_recycle=3600
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, index=True, nullable=False)
    email = Column(String(100), unique=True, index=True, nullable=False)
    full_name = Column(String(100))
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    role = Column(String(20), default="user")
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)

async def get_db_session() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

## 项目结构规范

### 标准项目布局

```
my_python_project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── main.py              # 应用入口点
│       ├── config.py            # 配置管理
│       ├── models/              # 数据模型
│       │   ├── __init__.py
│       │   ├── base.py
│       │   └── user.py
│       ├── schemas/             # Pydantic 模式
│       │   ├── __init__.py
│       │   └── user.py
│       ├── services/            # 业务逻辑层
│       │   ├── __init__.py
│       │   ├── base.py
│       │   └── user_service.py
│       ├── api/                 # API 路由
│       │   ├── __init__.py
│       │   ├── dependencies.py
│       │   └── routes/
│       │       ├── __init__.py
│       │       ├── auth.py
│       │       └── users.py
│       ├── core/                # 核心功能
│       │   ├── __init__.py
│       │   ├── database.py
│       │   ├── security.py
│       │   └── exceptions.py
│       └── utils/               # 工具函数
│           ├── __init__.py
│           ├── helpers.py
│           └── validators.py
├── tests/                       # 测试文件
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/                        # 文档
├── scripts/                     # 脚本文件
├── docker/                      # Docker 配置
├── .env.example                 # 环境变量示例
├── .gitignore
├── pyproject.toml              # 项目配置
├── README.md
└── Dockerfile
```

### 配置管理
```python
from pydantic import BaseSettings, Field
from typing import Optional
import os

class Settings(BaseSettings):
    # 应用配置
    app_name: str = "My Python App"
    app_version: str = "1.0.0"
    debug: bool = Field(default=False, env="DEBUG")
    
    # 数据库配置
    database_url: str = Field(..., env="DATABASE_URL")
    database_pool_size: int = Field(default=20, env="DB_POOL_SIZE")
    
    # Redis 配置
    redis_url: str = Field(default="redis://localhost:6379", env="REDIS_URL")
    
    # 安全配置
    secret_key: str = Field(..., env="SECRET_KEY")
    access_token_expire_minutes: int = Field(default=30, env="TOKEN_EXPIRE_MINUTES")
    
    # 外部服务
    email_service_url: Optional[str] = Field(default=None, env="EMAIL_SERVICE_URL")
    
    class Config:
        env_file = ".env"
        case_sensitive = False

# 全局设置实例
settings = Settings()
```

## 测试

### pytest 配置和最佳实践
```python
# conftest.py
import pytest
import asyncio
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.pool import StaticPool
from main import app
from core.database import get_db_session, Base

# 测试数据库配置
TEST_DATABASE_URL = "sqlite+aiosqlite:///:memory:"

@pytest.fixture(scope="session")
def event_loop():
    """创建事件循环"""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def test_engine():
    """创建测试数据库引擎"""
    engine = create_async_engine(
        TEST_DATABASE_URL,
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    yield engine
    await engine.dispose()

@pytest.fixture
async def test_db_session(test_engine):
    """创建测试数据库会话"""
    async with AsyncSession(test_engine) as session:
        yield session
        await session.rollback()

@pytest.fixture
async def client(test_db_session):
    """创建测试客户端"""
    app.dependency_overrides[get_db_session] = lambda: test_db_session
    
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    
    app.dependency_overrides.clear()

# 测试示例
@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    """测试创建用户"""
    user_data = {
        "username": "testuser",
        "email": "test@example.com",
        "password": "TestPass123",
        "full_name": "Test User"
    }
    
    response = await client.post("/users/", json=user_data)
    
    assert response.status_code == 201
    data = response.json()
    assert data["username"] == user_data["username"]
    assert data["email"] == user_data["email"]
    assert "password" not in data  # 确保密码不在响应中

@pytest.mark.asyncio
async def test_get_user_not_found(client: AsyncClient):
    """测试获取不存在的用户"""
    response = await client.get("/users/999")
    assert response.status_code == 404

# 参数化测试
@pytest.mark.parametrize("username,email,password,expected_status", [
    ("validuser", "valid@example.com", "ValidPass123", 201),
    ("", "valid@example.com", "ValidPass123", 422),  # 用户名为空
    ("validuser", "invalid-email", "ValidPass123", 422),  # 无效邮箱
    ("validuser", "valid@example.com", "weak", 422),  # 弱密码
])
@pytest.mark.asyncio
async def test_create_user_validation(client: AsyncClient, username, email, password, expected_status):
    """测试用户创建验证"""
    user_data = {
        "username": username,
        "email": email,
        "password": password
    }
    
    response = await client.post("/users/", json=user_data)
    assert response.status_code == expected_status
```

## 技术栈版本要求

| 技术栈 | 版本要求 | 说明 |
|--------|----------|------|
| **Python** | `^3.11.0` | 使用最新稳定版，支持现代语法特性 |
| **FastAPI** | `^0.104.0` | 高性能异步 Web 框架 |
| **Pydantic** | `^2.5.0` | 数据验证和序列化 |
| **SQLAlchemy** | `^2.0.0` | 现代 ORM，支持异步操作 |

### 核心依赖

```toml
[tool.poetry.dependencies]
python = "^3.11"
fastapi = "^0.104.0"
uvicorn = {extras = ["standard"], version = "^0.24.0"}
pydantic = {extras = ["email"], version = "^2.5.0"}
sqlalchemy = {extras = ["asyncio"], version = "^2.0.0"}
alembic = "^1.13.0"
asyncpg = "^0.29.0"
redis = "^5.0.0"
python-jose = {extras = ["cryptography"], version = "^3.3.0"}
passlib = {extras = ["bcrypt"], version = "^1.7.4"}
python-multipart = "^0.0.6"
```

### 开发工具

```toml
[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
pytest-asyncio = "^0.21.0"
httpx = "^0.25.0"
black = "^23.0.0"
isort = "^5.12.0"
flake8 = "^6.0.0"
mypy = "^1.7.0"
pre-commit = "^3.5.0"
```

### 环境配置

#### pyproject.toml 配置

```toml
[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.poetry]
name = "my-python-project"
version = "0.1.0"
description = "现代 Python 项目模板"
authors = ["Your Name <your.email@example.com>"]
readme = "README.md"
packages = [{include = "my_project", from = "src"}]

[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''

[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["my_project"]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_unreachable = true
strict_equality = true

[tool.pytest.ini_options]
minversion = "6.0"
addopts = "-ra -q --strict-markers --strict-config"
testpaths = ["tests"]
asyncio_mode = "auto"
```

---

## 性能优化技巧

### 数据库优化
```python
from sqlalchemy import select, func
from sqlalchemy.orm import selectinload, joinedload

class OptimizedUserService:
    async def get_users_with_posts(self, limit: int = 10) -> List[User]:
        """优化的用户查询，预加载关联数据"""
        stmt = (
            select(User)
            .options(selectinload(User.posts))  # 预加载文章
            .where(User.is_active == True)
            .limit(limit)
        )
        result = await self.db.execute(stmt)
        return result.scalars().all()
    
    async def get_user_statistics(self) -> Dict[str, int]:
        """使用聚合查询获取统计信息"""
        stmt = select(
            func.count(User.id).label('total_users'),
            func.count(User.id).filter(User.is_active == True).label('active_users'),
            func.count(User.id).filter(User.role == 'admin').label('admin_users')
        )
        result = await self.db.execute(stmt)
        row = result.first()
        return {
            'total_users': row.total_users,
            'active_users': row.active_users,
            'admin_users': row.admin_users
        }
```

### 缓存策略
```python
import redis.asyncio as redis
from functools import wraps
import json
import pickle
from typing import Any, Callable

class CacheService:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
    
    async def get(self, key: str) -> Any:
        """获取缓存值"""
        value = await self.redis.get(key)
        if value:
            return pickle.loads(value)
        return None
    
    async def set(self, key: str, value: Any, expire: int = 3600) -> None:
        """设置缓存值"""
        await self.redis.set(key, pickle.dumps(value), ex=expire)
    
    async def delete(self, key: str) -> None:
        """删除缓存"""
        await self.redis.delete(key)

def cache_result(expire: int = 3600, key_prefix: str = ""):
    """缓存装饰器"""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 生成缓存键
            cache_key = f"{key_prefix}:{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # 尝试从缓存获取
            cached_result = await cache_service.get(cache_key)
            if cached_result is not None:
                return cached_result
            
            # 执行函数并缓存结果
            result = await func(*args, **kwargs)
            await cache_service.set(cache_key, result, expire)
            
            return result
        return wrapper
    return decorator

# 使用示例
@cache_result(expire=1800, key_prefix="user")
async def get_user_profile(user_id: int) -> Dict[str, Any]:
    # 复杂的用户资料查询
    pass
```

### 异步任务处理
```python
import asyncio
from typing import List, Callable, Any
from concurrent.futures import ThreadPoolExecutor
import logging

class TaskProcessor:
    def __init__(self, max_workers: int = 10):
        self.executor = ThreadPoolExecutor(max_workers=max_workers)
        self.logger = logging.getLogger(__name__)
    
    async def process_batch(
        self, 
        items: List[Any], 
        processor: Callable,
        batch_size: int = 10,
        max_concurrent: int = 5
    ) -> List[Any]:
        """批量处理任务"""
        semaphore = asyncio.Semaphore(max_concurrent)
        results = []
        
        async def process_item(item):
            async with semaphore:
                try:
                    if asyncio.iscoroutinefunction(processor):
                        return await processor(item)
                    else:
                        # CPU 密集型任务使用线程池
                        loop = asyncio.get_event_loop()
                        return await loop.run_in_executor(self.executor, processor, item)
                except Exception as e:
                    self.logger.error(f"处理项目失败: {item}, 错误: {e}")
                    return None
        
        # 分批处理
        for i in range(0, len(items), batch_size):
            batch = items[i:i + batch_size]
            batch_tasks = [process_item(item) for item in batch]
            batch_results = await asyncio.gather(*batch_tasks, return_exceptions=True)
            results.extend(batch_results)
        
        return [r for r in results if r is not None]
```

## 项目部署和环境管理

### Docker 容器化部署

#### Dockerfile 示例
```dockerfile
# Python 基础镜像
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 设置环境变量
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .
COPY requirements-dev.txt .

# 安装 Python 依赖
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# 复制应用代码
COPY . .

# 创建非 root 用户
RUN adduser --disabled-password --gecos '' appuser && \
    chown -R appuser:appuser /app
USER appuser

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# 启动命令
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

#### docker-compose.yml 示例
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
      - SECRET_KEY=${SECRET_KEY}
    depends_on:
      - db
      - redis
    volumes:
      - ./media:/app/media
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
      - ./media:/var/www/media
    depends_on:
      - web
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### 环境管理最佳实践

#### 使用 pyenv 管理 Python 版本
```bash
# 安装 pyenv
curl https://pyenv.run | bash

# 安装 Python 版本
pyenv install 3.11.0
pyenv global 3.11.0

# 项目特定版本
echo "3.11.0" > .python-version
```

#### 使用 Poetry 管理依赖
```toml
# pyproject.toml
[tool.poetry]
name = "my-python-app"
version = "0.1.0"
description = "Python 应用示例"
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.11"
fastapi = "^0.104.0"
uvicorn = {extras = ["standard"], version = "^0.24.0"}
sqlalchemy = "^2.0.0"
alembic = "^1.12.0"
pydantic = "^2.4.0"
python-multipart = "^0.0.6"
python-jose = {extras = ["cryptography"], version = "^3.3.0"}
passlib = {extras = ["bcrypt"], version = "^1.7.4"}
python-decouple = "^3.8"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
pytest-asyncio = "^0.21.0"
pytest-cov = "^4.1.0"
black = "^23.9.0"
isort = "^5.12.0"
flake8 = "^6.1.0"
mypy = "^1.6.0"
pre-commit = "^3.5.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'

[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "--cov=app --cov-report=html --cov-report=term-missing"
```

#### 环境变量管理
```python
# config.py
import os
from typing import Optional
from pydantic import BaseSettings, validator

class Settings(BaseSettings):
    """应用配置"""
    
    # 基础配置
    app_name: str = "My Python App"
    debug: bool = False
    secret_key: str
    
    # 数据库配置
    database_url: str
    database_echo: bool = False
    
    # Redis 配置
    redis_url: str = "redis://localhost:6379/0"
    
    # JWT 配置
    jwt_secret_key: str
    jwt_algorithm: str = "HS256"
    jwt_expire_minutes: int = 30
    
    # 邮件配置
    mail_server: Optional[str] = None
    mail_port: int = 587
    mail_username: Optional[str] = None
    mail_password: Optional[str] = None
    mail_use_tls: bool = True
    
    # 文件上传
    upload_max_size: int = 10 * 1024 * 1024  # 10MB
    upload_allowed_extensions: list = ['.jpg', '.jpeg', '.png', '.gif', '.pdf']
    
    # 日志配置
    log_level: str = "INFO"
    log_file: Optional[str] = None
    
    @validator('database_url')
    def validate_database_url(cls, v):
        if not v:
            raise ValueError('DATABASE_URL is required')
        return v
    
    @validator('secret_key')
    def validate_secret_key(cls, v):
        if not v or len(v) < 32:
            raise ValueError('SECRET_KEY must be at least 32 characters')
        return v
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        case_sensitive = False

# 创建全局设置实例
settings = Settings()
```

#### .env 文件示例
```bash
# .env
# 基础配置
SECRET_KEY=your-super-secret-key-here-at-least-32-characters
DEBUG=false
APP_NAME=My Python App

# 数据库配置
DATABASE_URL=postgresql://user:password@localhost:5432/myapp
DATABASE_ECHO=false

# Redis 配置
REDIS_URL=redis://localhost:6379/0

# JWT 配置
JWT_SECRET_KEY=your-jwt-secret-key
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=30

# 邮件配置
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_USE_TLS=true

# 日志配置
LOG_LEVEL=INFO
LOG_FILE=app.log
```

### CI/CD 配置

#### GitHub Actions 示例
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install Poetry
      uses: snok/install-poetry@v1
      with:
        version: latest
        virtualenvs-create: true
        virtualenvs-in-project: true
    
    - name: Load cached venv
      id: cached-poetry-dependencies
      uses: actions/cache@v3
      with:
        path: .venv
        key: venv-${{ runner.os }}-${{ hashFiles('**/poetry.lock') }}
    
    - name: Install dependencies
      if: steps.cached-poetry-dependencies.outputs.cache-hit != 'true'
      run: poetry install --no-interaction --no-root
    
    - name: Install project
      run: poetry install --no-interaction
    
    - name: Run linting
      run: |
        poetry run black --check .
        poetry run isort --check-only .
        poetry run flake8 .
        poetry run mypy .
    
    - name: Run tests
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
        REDIS_URL: redis://localhost:6379/0
        SECRET_KEY: test-secret-key-for-ci-cd-pipeline
        JWT_SECRET_KEY: test-jwt-secret-key
      run: |
        poetry run pytest --cov=app --cov-report=xml
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
        fail_ci_if_error: true

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # 添加部署脚本
```

### 监控和日志

#### 日志配置
```python
# logging_config.py
import logging
import logging.config
from typing import Dict, Any

def setup_logging(log_level: str = "INFO", log_file: str = None) -> None:
    """配置日志系统"""
    
    config: Dict[str, Any] = {
        "version": 1,
        "disable_existing_loggers": False,
        "formatters": {
            "default": {
                "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s",
                "datefmt": "%Y-%m-%d %H:%M:%S",
            },
            "detailed": {
                "format": "%(asctime)s - %(name)s - %(levelname)s - %(module)s - %(funcName)s - %(lineno)d - %(message)s",
                "datefmt": "%Y-%m-%d %H:%M:%S",
            },
        },
        "handlers": {
            "console": {
                "class": "logging.StreamHandler",
                "level": log_level,
                "formatter": "default",
                "stream": "ext://sys.stdout",
            },
        },
        "loggers": {
            "": {  # root logger
                "level": log_level,
                "handlers": ["console"],
                "propagate": False,
            },
            "uvicorn": {
                "level": "INFO",
                "handlers": ["console"],
                "propagate": False,
            },
            "sqlalchemy.engine": {
                "level": "WARNING",
                "handlers": ["console"],
                "propagate": False,
            },
        },
    }
    
    # 添加文件处理器
    if log_file:
        config["handlers"]["file"] = {
            "class": "logging.handlers.RotatingFileHandler",
            "level": log_level,
            "formatter": "detailed",
            "filename": log_file,
            "maxBytes": 10485760,  # 10MB
            "backupCount": 5,
        }
        
        # 为所有日志器添加文件处理器
        for logger_config in config["loggers"].values():
            logger_config["handlers"].append("file")
    
    logging.config.dictConfig(config)

# 使用示例
logger = logging.getLogger(__name__)

def log_request(request_id: str, method: str, path: str, user_id: int = None):
    """记录请求日志"""
    logger.info(
        f"Request {request_id}: {method} {path}",
        extra={
            "request_id": request_id,
            "method": method,
            "path": path,
            "user_id": user_id,
        }
    )

def log_error(error: Exception, context: dict = None):
    """记录错误日志"""
    logger.error(
        f"Error occurred: {str(error)}",
        extra={"error_type": type(error).__name__, "context": context or {}},
        exc_info=True
    )
```

#### 健康检查端点
```python
# health.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from sqlalchemy import text
import redis
from app.database import get_db
from app.config import settings

router = APIRouter()

@router.get("/health")
async def health_check():
    """基础健康检查"""
    return {"status": "healthy", "service": "my-python-app"}

@router.get("/health/detailed")
async def detailed_health_check(db: Session = Depends(get_db)):
    """详细健康检查"""
    health_status = {
        "status": "healthy",
        "checks": {
            "database": "unknown",
            "redis": "unknown",
        }
    }
    
    # 检查数据库连接
    try:
        db.execute(text("SELECT 1"))
        health_status["checks"]["database"] = "healthy"
    except Exception as e:
        health_status["checks"]["database"] = f"unhealthy: {str(e)}"
        health_status["status"] = "unhealthy"
    
    # 检查 Redis 连接
    try:
        r = redis.from_url(settings.redis_url)
        r.ping()
        health_status["checks"]["redis"] = "healthy"
    except Exception as e:
        health_status["checks"]["redis"] = f"unhealthy: {str(e)}"
        health_status["status"] = "unhealthy"
    
    if health_status["status"] == "unhealthy":
        raise HTTPException(status_code=503, detail=health_status)
    
    return health_status
```

### 性能优化

#### 数据库优化
```python
# database_optimization.py
from sqlalchemy import create_engine, event
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import QueuePool
import time
import logging

logger = logging.getLogger(__name__)

# 数据库连接池配置
engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,          # 连接池大小
    max_overflow=30,       # 最大溢出连接数
    pool_pre_ping=True,    # 连接前检查
    pool_recycle=3600,     # 连接回收时间（秒）
    echo=False,            # 生产环境关闭 SQL 日志
)

# 监听数据库查询性能
@event.listens_for(engine, "before_cursor_execute")
def receive_before_cursor_execute(conn, cursor, statement, parameters, context, executemany):
    context._query_start_time = time.time()

@event.listens_for(engine, "after_cursor_execute")
def receive_after_cursor_execute(conn, cursor, statement, parameters, context, executemany):
    total = time.time() - context._query_start_time
    if total > 0.5:  # 记录慢查询（超过500ms）
        logger.warning(f"Slow query: {total:.2f}s - {statement[:100]}...")

# 数据库查询优化示例
from sqlalchemy.orm import selectinload, joinedload

def get_users_with_profiles_optimized(db: Session, limit: int = 100):
    """优化的用户查询 - 使用预加载避免 N+1 问题"""
    return db.query(User)\
        .options(selectinload(User.profile))\
        .limit(limit)\
        .all()

def get_user_posts_optimized(db: Session, user_id: int):
    """优化的用户文章查询 - 使用连接加载"""
    return db.query(User)\
        .options(joinedload(User.posts))\
        .filter(User.id == user_id)\
        .first()
```

#### 缓存策略
```python
# cache.py
import redis
import json
import pickle
from typing import Any, Optional, Union
from functools import wraps
import hashlib

class CacheManager:
    """缓存管理器"""
    
    def __init__(self, redis_url: str):
        self.redis_client = redis.from_url(redis_url)
    
    def get(self, key: str) -> Optional[Any]:
        """获取缓存"""
        try:
            data = self.redis_client.get(key)
            if data:
                return pickle.loads(data)
        except Exception as e:
            logger.error(f"Cache get error: {e}")
        return None
    
    def set(self, key: str, value: Any, expire: int = 3600) -> bool:
        """设置缓存"""
        try:
            data = pickle.dumps(value)
            return self.redis_client.setex(key, expire, data)
        except Exception as e:
            logger.error(f"Cache set error: {e}")
            return False
    
    def delete(self, key: str) -> bool:
        """删除缓存"""
        try:
            return bool(self.redis_client.delete(key))
        except Exception as e:
            logger.error(f"Cache delete error: {e}")
            return False
    
    def clear_pattern(self, pattern: str) -> int:
        """清除匹配模式的缓存"""
        try:
            keys = self.redis_client.keys(pattern)
            if keys:
                return self.redis_client.delete(*keys)
        except Exception as e:
            logger.error(f"Cache clear pattern error: {e}")
        return 0

# 缓存装饰器
def cache_result(expire: int = 3600, key_prefix: str = ""):
    """缓存函数结果的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # 生成缓存键
            cache_key = f"{key_prefix}:{func.__name__}:{_generate_cache_key(args, kwargs)}"
            
            # 尝试从缓存获取
            cached_result = cache_manager.get(cache_key)
            if cached_result is not None:
                return cached_result
            
            # 执行函数并缓存结果
            result = func(*args, **kwargs)
            cache_manager.set(cache_key, result, expire)
            
            return result
        return wrapper
    return decorator

def _generate_cache_key(args: tuple, kwargs: dict) -> str:
    """生成缓存键"""
    key_data = str(args) + str(sorted(kwargs.items()))
    return hashlib.md5(key_data.encode()).hexdigest()

# 使用示例
cache_manager = CacheManager(settings.redis_url)

@cache_result(expire=1800, key_prefix="user")
def get_user_profile(user_id: int):
    """获取用户资料（带缓存）"""
    # 数据库查询逻辑
    pass
```

## AI 开发指导

### Python 代码生成规则

#### 🤖 AI 代码生成提示词模板

```
请基于以下规范生成 Python 代码：

**技术要求：**
- 使用 Python 3.11+ 语法特性
- 完整的类型提示和 Pydantic 模型
- 遵循 PEP 8 代码规范
- 异步编程支持（如适用）

**代码规范：**
- 模块名称：{module_name}
- 功能描述：{功能说明}
- 输入参数：{参数列表}
- 返回类型：{返回类型}
- 异常处理：{异常类型}

**质量要求：**
- 包含完整的文档字符串
- 实现适当的错误处理
- 添加必要的日志记录
- 编写对应的单元测试

**文件结构：**
- main.py（主要实现）
- schemas.py（数据模型）
- tests/test_main.py（单元测试）
```

#### 📋 代码生成检查清单

- [ ] 使用类型提示和 Pydantic 模型
- [ ] 完整的错误处理和日志记录
- [ ] 遵循 PEP 8 代码规范
- [ ] 包含文档字符串和注释
- [ ] 实现相应的单元测试
- [ ] 使用现代 Python 语法特性
- [ ] 适当的异步编程模式
- [ ] 安全的数据处理和验证

---

这个 Python 开发规范文档涵盖了现代 Python 开发的各个方面，从基础的代码风格到高级的性能优化技巧。它为 AI 助手提供了全面的指导，确保生成的 Python 代码符合最佳实践和行业标准。