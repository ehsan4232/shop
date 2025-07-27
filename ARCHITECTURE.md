# Mall Platform - Architecture & Technology Stack

## System Architecture Overview

### Multi-Tenant SaaS Architecture
The Mall platform follows a **multi-tenant, single-database** architecture optimized for 1000+ concurrent users with room for horizontal scaling.

```
┌─────────────────────────────────────────────────────────────┐
│                     Load Balancer (Nginx)                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────┐
│                 API Gateway                                 │
│            (Django + Django REST Framework)                │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────┐
│                Frontend Layer                               │
│    ┌─────────────────┬─────────────────┬─────────────────┐  │
│    │   Next.js App   │  Admin Panel    │   Store Sites   │  │
│    │   (Landing)     │  (Dashboard)    │   (Multi-theme) │  │
│    └─────────────────┴─────────────────┴─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────┐
│                Database Layer                               │
│    ┌─────────────────┬─────────────────┬─────────────────┐  │
│    │   PostgreSQL    │     Redis       │   File Storage  │  │
│    │   (Primary DB)  │   (Cache/Queue) │   (Media/CDN)   │  │
│    └─────────────────┴─────────────────┴─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────┐
│              External Services                              │
│    ┌─────────────────┬─────────────────┬─────────────────┐  │
│    │   Payment       │   SMS Gateway   │  Social Media   │  │
│    │   Gateways      │   (Kavenegar)   │   APIs          │  │
│    └─────────────────┴─────────────────┴─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Technology Stack

### Backend (shop-back)
- **Framework**: Django 4.2+ with Django REST Framework
- **Database**: PostgreSQL 15+ (Primary) + Redis 7+ (Cache/Sessions/Queue)
- **Authentication**: JWT + OTP (using django-otp)
- **API Documentation**: Django REST Framework + drf-spectacular (OpenAPI)
- **Task Queue**: Celery with Redis broker
- **File Storage**: Django-storages with MinIO/S3-compatible storage
- **Multi-tenancy**: django-tenant-schemas or custom implementation
- **Monitoring**: Django Debug Toolbar (dev) + Sentry (prod)

### Frontend (shop-front)
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **UI Library**: Tailwind CSS + Headless UI
- **Persian Support**: next-intl for i18n
- **State Management**: Zustand or TanStack Query
- **Forms**: React Hook Form + Zod validation
- **Charts**: Chart.js or Recharts
- **Rich Text**: TipTap or similar Persian-friendly editor

### Infrastructure & DevOps
- **Containerization**: Docker + Docker Compose
- **Web Server**: Nginx (reverse proxy, static files)
- **Deployment**: GitHub Actions CI/CD
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)

### Third-Party Integrations
- **Payment Gateways**: ZarinPal, Mellat Bank, Parsian, Saman
- **SMS Service**: Kavenegar, SmsIr
- **Social Media**: Instagram Basic Display API, Telegram Bot API
- **Domain Management**: Custom domain routing with wildcard SSL

## Database Design

### Core Models Structure

#### Tenant Management
```python
class Tenant(models.Model):
    name = models.CharField(max_length=100)
    schema_name = models.CharField(max_length=63, unique=True)
    domain_url = models.CharField(max_length=128, unique=True)
    created_on = models.DateField(auto_now_add=True)
    is_active = models.BooleanField(default=True)

class Store(models.Model):
    tenant = models.OneToOneField(Tenant, on_delete=models.CASCADE)
    owner = models.ForeignKey('User', on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    logo = models.ImageField(upload_to='store_logos/')
    theme = models.CharField(max_length=50, default='default')
    layout = models.CharField(max_length=50, default='grid')
    domain = models.CharField(max_length=255, unique=True, null=True, blank=True)
    is_subdomain = models.BooleanField(default=True)
```

#### Product Hierarchy System
```python
class ProductCategory(MPTTModel):
    name = models.CharField(max_length=100)
    name_fa = models.CharField(max_length=100)  # Persian name
    parent = TreeForeignKey('self', on_delete=models.CASCADE, null=True, blank=True)
    is_categorizer = models.BooleanField(default=False)  # For subclass generation
    store = models.ForeignKey('Store', on_delete=models.CASCADE)
    
    class MPTTMeta:
        order_insertion_by = ['name']

class ProductAttribute(models.Model):
    ATTRIBUTE_TYPES = [
        ('color', 'Color'),
        ('description', 'Description'),
        ('size', 'Size'),
        ('custom', 'Custom'),
    ]
    
    category = models.ForeignKey(ProductCategory, on_delete=models.CASCADE)
    name = models.CharField(max_length=50)
    name_fa = models.CharField(max_length=50)
    attribute_type = models.CharField(max_length=20, choices=ATTRIBUTE_TYPES)
    is_required = models.BooleanField(default=False)
    is_categorizer = models.BooleanField(default=False)

class Product(models.Model):
    store = models.ForeignKey('Store', on_delete=models.CASCADE)
    category = models.ForeignKey(ProductCategory, on_delete=models.CASCADE)
    name = models.CharField(max_length=200)
    name_fa = models.CharField(max_length=200)
    base_price = models.DecimalField(max_digits=10, decimal_places=2)
    images = models.JSONField(default=list)  # Store image URLs
    videos = models.JSONField(default=list)  # Store video URLs
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    view_count = models.PositiveIntegerField(default=0)

class ProductInstance(models.Model):
    product = models.ForeignKey(Product, on_delete=models.CASCADE, related_name='instances')
    price = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    stock_quantity = models.PositiveIntegerField(default=0)
    sku = models.CharField(max_length=100, unique=True)
    is_active = models.BooleanField(default=True)

class ProductInstanceAttribute(models.Model):
    instance = models.ForeignKey(ProductInstance, on_delete=models.CASCADE)
    attribute = models.ForeignKey(ProductAttribute, on_delete=models.CASCADE)
    value = models.CharField(max_length=255)
    color_hex = models.CharField(max_length=7, null=True, blank=True)  # For color attributes
```

#### User & Authentication
```python
class User(AbstractUser):
    phone_number = models.CharField(max_length=15, unique=True)
    is_store_owner = models.BooleanField(default=False)
    is_customer = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    USERNAME_FIELD = 'phone_number'

class OTPVerification(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    phone_number = models.CharField(max_length=15)
    otp_code = models.CharField(max_length=6)
    is_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    expires_at = models.DateTimeField()
```

## API Architecture

### RESTful API Design
```
/api/v1/
├── auth/
│   ├── send-otp/
│   ├── verify-otp/
│   └── refresh-token/
├── stores/
│   ├── {store_id}/
│   ├── products/
│   ├── categories/
│   └── analytics/
├── products/
│   ├── {product_id}/
│   ├── instances/
│   └── attributes/
├── orders/
├── customers/
└── social-media/
    ├── telegram/
    └── instagram/
```

### Multi-tenant Routing
- **Subdomain routing**: `{store}.mall.ir`
- **Custom domain support**: `store.com`
- **API tenant detection**: Via subdomain or custom header

## Performance Optimization (1000+ Users)

### Caching Strategy
- **Redis Cache**: API responses, product catalogs, user sessions
- **Database Query Optimization**: Select_related, prefetch_related, database indexes
- **CDN**: Static assets and media files
- **Page Caching**: Full page cache for product listings

### Database Optimization
- **Connection Pooling**: PgBouncer for PostgreSQL
- **Read Replicas**: Separate read/write operations
- **Database Partitioning**: By tenant or date for large tables
- **Indexing Strategy**: Composite indexes for common queries

### Horizontal Scaling
- **Load Balancing**: Multiple Django application servers
- **Microservices**: Separate services for high-load operations
- **Queue Processing**: Celery workers for background tasks
- **Database Sharding**: Tenant-based sharding for extreme scale

## Security Considerations

### Authentication & Authorization
- **JWT Tokens**: Short-lived access tokens + refresh tokens
- **OTP Verification**: SMS-based two-factor authentication
- **Rate Limiting**: API endpoint protection
- **CORS Configuration**: Proper cross-origin settings

### Data Protection
- **Tenant Isolation**: Complete data separation between stores
- **SQL Injection Prevention**: ORM usage, parameterized queries
- **XSS Protection**: Input sanitization, CSP headers
- **File Upload Security**: Type validation, virus scanning

## Deployment Strategy

### Development Environment
```yaml
# docker-compose.yml
version: '3.8'
services:
  backend:
    build: ./shop-back
    ports:
      - "8000:8000"
    environment:
      - DEBUG=True
      - DATABASE_URL=postgresql://user:pass@db:5432/mall_db
    depends_on:
      - db
      - redis

  frontend:
    build: ./shop-front
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mall_db
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass

  redis:
    image: redis:7-alpine
```

### Production Deployment
- **Container Orchestration**: Docker Swarm or Kubernetes
- **Reverse Proxy**: Nginx with SSL termination
- **Database**: Managed PostgreSQL with automated backups
- **Monitoring**: Health checks, performance metrics
- **CI/CD**: Automated testing and deployment pipeline

## Development Phases

### Phase 1: Core Backend (Weeks 1-4)
- Django project setup with multi-tenancy
- User authentication system with OTP
- Basic product model implementation
- Store management system
- Admin panel customization

### Phase 2: Product System (Weeks 5-8)
- Advanced product hierarchy implementation
- Product instance management
- Attribute system with categorizers
- Media management (images/videos)
- Social media integration APIs

### Phase 3: Frontend Development (Weeks 9-12)
- Next.js application setup
- Admin dashboard development
- Store website themes
- Product management interfaces
- Persian language optimization

### Phase 4: E-commerce Features (Weeks 13-16)
- Shopping cart and checkout
- Order management system
- Payment gateway integration
- Customer account system
- SMS notification system

### Phase 5: Analytics & Optimization (Weeks 17-20)
- Dashboard analytics implementation
- Performance optimization
- SEO optimization
- Mobile responsiveness
- Testing and deployment

This architecture provides a solid foundation for the Mall platform, designed to handle 1000+ concurrent users while maintaining scalability for future growth. The technology choices prioritize performance, maintainability, and the specific requirements of the Iranian market.
