# Mall Platform - Product Description
## مال - فروشگاه‌ساز

### Platform Overview
Mall (فروشگاه‌ساز مال) is a comprehensive e-commerce platform designed specifically for the Iranian market. The platform enables store owners to create and manage their online stores with full Persian language support.

### Core Features

#### Platform Capabilities
- Store website builder for shop owners
- Persian language interface and content
- Store management dashboard for owners
- Product management system
- Multi-store architecture

#### Visual Identity
- Unique logo design featuring red, blue, and white colors
- Modern, fancy homepage design
- Long-form landing page with feature presentations
- Complementary images and short videos
- Dual call-to-action buttons (top and middle/bottom)
- Pop-up request forms
- Sliders and interactive elements
- Online chat integration
- Store owner login section

#### Administration System
- Django admin panel for platform management
- Store creation and management
- User account management
- Product catalog management
- Custom product creation for store owners

#### Product Architecture

##### Product Class Structure
- **Root Class**: Product (with price attribute and media lists)
- **Tree Structure**: Unlimited depth levels
- **Categorization System**: Level 1 children can be marked as "categorize by this"
- **Subclass Generation**: Marked attributes become subclasses
- **Instance Creation**: Only from leaf nodes

##### Product Attributes
- **Predefined Types**: Color, Description
- **Color Fields**: Visual color squares
- **Description Fields**: Text boxes
- **Custom Attributes**: User-defined attributes per level
- **Empty Values**: Allowed for non-price attributes

##### Product Instances
- Created from leaf products only
- Multiple instances with varying attributes
- Bulk creation with "create another" checkbox
- Stock management and warnings
- Shared and unique attribute display

##### Social Media Integration
- "Get from social media" button for descriptions and media
- Telegram and Instagram post/story import
- Last 5 posts extraction
- Separate image, video, and text selection
- Direct content integration

#### Store Website Features

##### Product Display
- Recent products lists
- Category organization (main classes and subclasses)
- Most viewed products
- Recommended products
- Complete product and category menus
- Search functionality (names and categories)
- Multi-level filtering system
- Sorting options (recent, most viewed, most purchased, price)

##### Customization
- Multiple layout options
- Various themes
- Real-time theme/layout switching
- Independent domain support
- Optional subdomain hosting

##### E-commerce Functionality
- Iranian logistics provider integration
- Payment gateway integration
- Customer account management
- Order tracking
- Shopping cart management
- Checkout process
- SMS promotion campaigns

#### Authentication & Security
- OTP-based login system
- Secure user authentication
- Multi-level access control

#### Analytics & Reporting
- Sales dashboards
- Website analytics
- Interaction tracking
- Chart visualizations
- Performance metrics

### Technical Requirements
- Support for 1000+ concurrent users
- Persian language optimization
- Mobile-responsive design
- High performance and scalability
- Integration with Iranian payment systems
- Social media API integration
- Real-time analytics
- Multi-tenant architecture
