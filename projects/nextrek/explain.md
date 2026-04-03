# Nextrek System Architecture - Expert Analysis

## 📊 **Architecture Overview**

Nextrek is a sophisticated **multi-tenant SaaS accounting platform** built with Ruby on Rails, serving Taiwan/Chinese-speaking businesses. The architecture demonstrates several enterprise-grade patterns optimized for scalability, security, and domain complexity.

```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                        NEXTREK SYSTEM ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────────────┤
│                          🌐 PRESENTATION LAYER                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐      │
│  │   Web Frontend  │ │   Admin Portal  │ │   Mobile Views  │      │
│  │  (Bootstrap 5)  │ │  (Admin Tools)  │ │  (Responsive)   │      │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘      │
├─────────────────────────────────────────────────────────────────────┤
│                         🏗️ APPLICATION LAYER                       │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐      │
│  │   Controllers   │ │   Presenters    │ │     Forms       │      │
│  │                 │ │   (View Logic)  │ │  (Input Logic)  │      │
│  │ • Standard      │ │                 │ │                 │      │
│  │ • Admin         │ │ • Dashboard     │ │ • Complex Forms │      │
│  │ • Accountings   │ │ • Reports       │ │ • Validations   │      │
│  │ • Me            │ │ • Group Init    │ │ • Multi-step    │      │
│  │ • Settings      │ │                 │ │                 │      │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘      │
├─────────────────────────────────────────────────────────────────────┤
│                         🔧 SERVICE LAYER                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐      │
│  │   Services      │ │     Workers     │ │      Jobs       │      │
│  │                 │ │                 │ │                 │      │
│  │ • Bootstrap     │ │ • Auto Renew    │ │ • Group Init    │      │
│  │ • Duplication   │ │ • Confirmations │ │ • Reconciliation│      │
│  │ • PDF Export    │ │ • Reminders     │ │ • S3 Copy       │      │
│  │ • Invoice       │ │ • Cleanup       │ │ • TapPay        │      │
│  │ • Group Clear   │ │                 │ │                 │      │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘      │
├─────────────────────────────────────────────────────────────────────┤
│                         💼 DOMAIN LAYER                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    MULTI-TENANT CORE                       │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │    │
│  │  │   Account   │ │    Group    │ │    Staff    │          │    │
│  │  │  (Tenant)   │ │ (Company)   │ │ (Employee)  │          │    │
│  │  │             │ │             │ │             │          │    │
│  │  │ • Subdomain │ │ • Industry  │ │ • Roles     │          │    │
│  │  │ • User Mgmt │ │ • Multi-Ind │ │ • Permissions│          │    │
│  │  │ • Groups    │ │ • Config    │ │ • Time Zone │          │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  ACCOUNTING DOMAIN                         │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │    │
│  │  │   Journal   │ │    Entry    │ │   Subject   │          │    │
│  │  │ (Trans.)    │ │ (Dr/Cr)     │ │ (Account)   │          │    │
│  │  │             │ │             │ │             │          │    │
│  │  │ • Double    │ │ • Position  │ │ • Category  │          │    │
│  │  │   Entry     │ │ • Amount    │ │ • Code      │          │    │
│  │  │ • Workflow  │ │ • Tags      │ │ • Balance   │          │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │    │
│  │  │   Account   │ │   Budget    │ │   Voucher   │          │    │
│  │  │ (Gateway)   │ │   (Plan)    │ │ (Receipt)   │          │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  BUSINESS DOMAIN                           │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │    │
│  │  │   Contact   │ │     Deal    │ │     Tag     │          │    │
│  │  │ (Customer)  │ │ (Transaction│ │ (Category)  │          │    │
│  │  │             │ │             │ │             │          │    │
│  │  │ • Person    │ │ • Items     │ │ • Flexible  │          │    │
│  │  │ • Org       │ │ • Members   │ │ • Hierarchy │          │    │
│  │  │ • Activity  │ │ • Performance│ │ • Cross-Ref │          │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │    │
│  └─────────────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────────────┤
│                         💾 DATA LAYER                              │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐      │
│  │   PostgreSQL    │ │      Redis      │ │       S3        │      │
│  │                 │ │                 │ │                 │      │
│  │ • Multi-tenant  │ │ • Cache         │ │ • Attachments   │      │
│  │ • ACID Trans    │ │ • Sessions      │ │ • Backups       │      │
│  │ • Complex Rel   │ │ • Background    │ │ • File Storage  │      │
│  │ • JSON Columns  │ │ • Rate Limit    │ │                 │      │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘      │
└─────────────────────────────────────────────────────────────────────┘
```

## 🏢 **Multi-Tenant Architecture**

### Tenant Hierarchy (4-Level Structure)
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                        MULTI-TENANT HIERARCHY                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  👤 USER ──────────────────────────────────────────────────────────┐│
│  │ • Authentication (Devise)                                      ││
│  │ • Global Identity                                               ││
│  │ • Cross-Account Access                                          ││
│  │ • Session Management                                            ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                      │                              │
│                                      ▼                              │
│  🏢 ACCOUNT ───────────────────────────────────────────────────────┐│
│  │ • Subdomain Isolation (account.nextrek.com)                    ││
│  │ • Top-level container                                           ││
│  │ • Industry type & headcount                                     ││
│  │ • Group limit management                                        ││
│  │ • Owner relationship                                            ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                      │                              │
│                                      ▼                              │
│  🏭 GROUP ─────────────────────────────────────────────────────────┐│
│  │ • Tenant-level isolation                                        ││
│  │ • Industry-specific customization (Normal, Hair Salon, etc.)    ││
│  │ • Subscription & billing management                             ││
│  │ • Accounting configuration                                      ││
│  │ • Data segregation boundary                                     ││
│  │ • Custom roles & permissions                                    ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                      │                              │
│                                      ▼                              │
│  👷 STAFF ─────────────────────────────────────────────────────────┐│
│  │ • User within Group                                             ││
│  │ • Role-based permissions                                        ││
│  │ • Account-level access control                                  ││
│  │ • Time zone & reporting preferences                             ││
│  │ • Activity tracking                                             ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Isolation Strategy
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                          DATA ISOLATION                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Row-Level Security (RLS) Implementation:                           │
│                                                                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐  │
│  │    Group A      │    │    Group B      │    │    Group C      │  │
│  │                 │    │                 │    │                 │  │
│  │ • Journals      │    │ • Journals      │    │ • Journals      │  │
│  │ • Entries       │    │ • Entries       │    │ • Entries       │  │
│  │ • Contacts      │    │ • Contacts      │    │ • Contacts      │  │
│  │ • Staff         │    │ • Staff         │    │ • Staff         │  │
│  │ • Reports       │    │ • Reports       │    │ • Reports       │  │
│  │                 │    │                 │    │                 │  │
│  │ group_id: 123   │    │ group_id: 456   │    │ group_id: 789   │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘  │
│                                                                     │
│  Automatic group_id filtering in:                                   │
│  • All ActiveRecord scopes                                          │
│  • Controller-level authorization                                   │
│  • Database foreign key constraints                                 │
│  • Query object patterns                                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 📊 **Double-Entry Accounting Domain**

### Core Accounting Architecture
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                     DOUBLE-ENTRY ACCOUNTING SYSTEM                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📋 JOURNAL (Transaction)                                           │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ • id: 12345                    • Receipt #: INV-2024-001    │     │
│  │ • Amount: $1,000.00            • Tax Rule: 5% VAT          │     │
│  │ • Kind: increase/decrease      • Declarable: true/false    │     │
│  │ • Deal Date: 2024-01-15        • Audit Date: 2024-01-20    │     │
│  │ • Subject: Sales Revenue       • Account: Bank Account     │     │
│  │ • Contact: Customer ABC        • Creator: Staff John       │     │
│  │ • Tags: [Q1, Sales, Online]    • Attachments: [invoice.pdf]│     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                      │                              │
│                                      ▼                              │
│  📊 ENTRIES (Double-Entry Details)                                  │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ DEBIT ENTRY:                   │ CREDIT ENTRY:              │     │
│  │ ┌─────────────────────────┐     │ ┌─────────────────────────┐ │     │
│  │ │ Account: Bank Account   │     │ │ Account: Sales Revenue  │ │     │
│  │ │ Amount: $1,000.00       │     │ │ Amount: $1,000.00       │ │     │
│  │ │ Subject: Cash           │     │ │ Subject: Revenue        │ │     │
│  │ │ Position: 1             │     │ │ Position: 2             │ │     │
│  │ │ Tags: [Bank, Deposit]   │     │ │ Tags: [Sales, Q1]       │ │     │
│  │ │ Contact: Customer ABC   │     │ │ Contact: Customer ABC   │ │     │
│  │ └─────────────────────────┘     │ └─────────────────────────┘ │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  💰 AUTOMATIC BALANCE CALCULATION                                   │
│  • Real-time account balance updates                                │
│  • Complex exchange transaction handling                            │
│  • Decimal precision configuration                                  │
│  • Multi-currency support foundation                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Financial Reporting Architecture
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                       FINANCIAL REPORTING SYSTEM                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📈 STATEMENT GENERATION                                            │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐        │
│  │ Income Statement│ │ Balance Sheet   │ │   Cash Flow     │        │
│  │                 │ │                 │ │                 │        │
│  │ • Revenue & Cost│ │ • Assets        │ │ • Operating     │        │
│  │ • Operating Exp │ │ • Liabilities   │ │ • Investing     │        │
│  │ • Non-Operating │ │ • Equity        │ │ • Financing     │        │
│  │ • Income Tax    │ │ • Balancing     │ │ • Net Change    │        │
│  │ • Extraordinary │ │ • Real-time     │ │ • Period Comp   │        │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘        │
│                                                                     │
│  🔢 BALANCE GROUP RULES (Statement Classification)                  │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ BALANCE_GROUP_RULES = {                                     │     │
│  │   revenue_and_cost: %w[4 5],                              │     │
│  │   operating_expense: %w[6],                               │     │
│  │   non_operating: %w[7 8],                                 │     │
│  │   income_tax: { primary: %w[9], only: /^90/ },           │     │
│  │   extraordinary: { primary: %w[9], except: /^90/ }       │     │
│  │ }                                                          │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🏷️ SUBJECT CATEGORIZATION                                          │
│  • Chart of Accounts with Taiwan accounting standards               │
│  • Industry-specific account templates                              │
│  • Custom subject categories                                        │
│  • Automatic statement type classification                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 🛠️ **Technology Stack & Architectural Patterns**

### Backend Architecture
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                          BACKEND TECHNOLOGY STACK                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🚀 CORE FRAMEWORK                                                  │
│  • Ruby 3.3.7 (Modern language features)                           │
│  • Rails 7.1.5 (Latest stable with enhanced performance)           │
│  • Puma (Production web server)                                     │
│  • Unicorn (Alternative deployment option)                         │
│                                                                     │
│  🗄️ DATA LAYER                                                      │
│  • PostgreSQL (Primary database with JSON columns)                 │
│  • Redis (Cache, sessions, Sidekiq, rate limiting)                 │
│  • AWS S3 (File storage with CarrierWave)                          │
│  • Active Storage (Rails 7 file attachments)                       │
│                                                                     │
│  ⚡ PERFORMANCE & BACKGROUND                                         │
│  • Sidekiq (Background job processing)                             │
│  • Redis::Objects (Distributed locks)                              │
│  • Action Cable (WebSocket connections)                            │
│  • Fragment caching with Rails.cache                               │
│                                                                     │
│  🔐 SECURITY & AUTH                                                 │
│  • Devise (Authentication)                                         │
│  • CanCanCan (Authorization)                                       │
│  • TOTP/Email OTP for admin (2FA)                                  │
│  • Brakeman (Security scanning)                                    │
│                                                                     │
│  💳 PAYMENT INTEGRATION                                             │
│  • TapPay (Taiwan payment gateway)                                 │
│  • Pay2Go/EZPay (Local payment processors)                         │
│  • Subscription billing automation                                 │
│                                                                     │
│  🧪 TESTING & QUALITY                                               │
│  • RSpec (Testing framework)                                       │
│  • Factory Bot (Test data generation)                              │
│  • Capybara + Playwright (Feature testing)                         │
│  • SimpleCov (Coverage reporting)                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Frontend Architecture
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                         FRONTEND TECHNOLOGY STACK                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🎨 UI FRAMEWORK                                                    │
│  • Bootstrap 5.3.0 (Modern responsive design)                      │
│  • SCSS (Advanced CSS preprocessing)                               │
│  • jQuery (DOM manipulation & AJAX)                                │
│  • Chart.js (Financial charts & visualizations)                    │
│                                                                     │
│  📱 RESPONSIVE DESIGN                                               │
│  • Mobile-first approach                                           │
│  • Progressive Web App capabilities                                │
│  • Optimized for Taiwan business workflows                         │
│                                                                     │
│  ⚡ ASSET PIPELINE                                                   │
│  • Rails Asset Pipeline                                            │
│  • Yarn package management                                         │
│  • ES6+ JavaScript support                                         │
│  • Asset compression & fingerprinting                              │
│                                                                     │
│  🖼️ RICH CONTENT                                                    │
│  • Trix editor (Rich text editing)                                 │
│  • FontAwesome icons                                               │
│  • Image optimization with CarrierWave                             │
│  • PDF generation with WickedPDF                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 🏗️ **Advanced Architectural Patterns**

### Domain-Driven Design Implementation
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                      DOMAIN-DRIVEN DESIGN PATTERNS                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🏛️ SERVICE OBJECTS (Business Logic Extraction)                    │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/services/                                               │     │
│  │ ├── bootstrap_service.rb      # Group initialization         │     │
│  │ ├── group_duplicate_service.rb # Business duplication       │     │
│  │ ├── auto_renew_service.rb      # Subscription management    │     │
│  │ ├── pdf_export_service.rb      # Report generation         │     │
│  │ └── subscription_date_service.rb # Complex date logic      │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  📋 FORM OBJECTS (Complex Input Handling)                          │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/forms/                                                  │     │
│  │ ├── basic_form.rb               # Base form abstraction     │     │
│  │ ├── staff_form.rb               # Staff management         │     │
│  │ ├── role_form.rb                # Permission management    │     │
│  │ └── accountings/budget_input_form.rb # Accounting forms    │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🎭 PRESENTERS (View Logic Separation)                             │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/presenters/                                             │     │
│  │ ├── application_presenter.rb    # Base presenter pattern    │     │
│  │ ├── dashboard_sidebar_presenter.rb # Navigation logic      │     │
│  │ └── group_initialization_presenter.rb # Setup workflow     │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🔍 QUERY OBJECTS (Database Query Encapsulation)                   │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/queries/                                                │     │
│  │ ├── base_query.rb               # Query abstraction         │     │
│  │ ├── journal_query.rb            # Complex journal searches │     │
│  │ ├── entry_query.rb              # Entry filtering          │     │
│  │ └── tag_analyse_query.rb        # Tag analysis             │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🎨 DECORATORS (Object Enhancement with Draper)                    │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/decorators/                                             │     │
│  │ ├── base_decorator.rb           # Decoration base           │     │
│  │ ├── group_decorator.rb          # Group view enhancements  │     │
│  │ ├── staff_decorator.rb          # Staff presentation       │     │
│  │ └── contact_decorator.rb        # Contact formatting       │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Industry-Specific Domain Architecture
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                    INDUSTRY-SPECIFIC DOMAINS                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🏢 NORMAL BUSINESS DOMAIN                                          │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/models/domain/normal/                                   │     │
│  │ ├── ability.rb              # Standard permissions          │     │
│  │ ├── configuration.rb        # General business config       │     │
│  │ ├── service/bootstrap.rb    # Standard initialization      │     │
│  │ └── presenter/group_dashboard.rb # Standard dashboard      │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  💇 HAIR SALON DOMAIN                                               │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ app/models/domain/hairsalon/                                │     │
│  │ ├── ability.rb              # Salon-specific permissions   │     │
│  │ ├── configuration.rb        # Salon business rules         │     │
│  │ ├── service/performance_calculation.rb # Stylist metrics   │     │
│  │ └── presenter/staff_dashboard.rb # Stylist-focused UI      │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🔧 DOMAIN FACTORY PATTERN                                          │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ def domain_class(*args)                                     │     │
│  │   klass = "#{args.unshift('Domain', industry.camelize)     │     │
│  │           .join('::')}".constantize                        │     │
│  │   block_given? ? yield(klass) : klass                      │     │
│  │ end                                                         │     │
│  │                                                             │     │
│  │ # Usage: group.domain_class('Service', 'Bootstrap')        │     │
│  │ # => Domain::Hairsalon::Service::Bootstrap                 │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 🚀 **Expert Performance & Scalability Insights**

### Caching Strategy
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                         CACHING ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🏎️ MULTI-LAYER CACHING                                            │
│                                                                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐        │
│  │   Browser       │ │     Rails       │ │     Redis       │        │
│  │   Cache         │ │     Cache       │ │     Cache       │        │
│  │                 │ │                 │ │                 │        │
│  │ • Static Assets │ │ • Fragment      │ │ • Session Data  │        │
│  │ • ETags         │ │ • Query Cache   │ │ • Background    │        │
│  │ • HTTP Headers  │ │ • Action Cache  │ │ • Permissions   │        │
│  │ • Service       │ │ • View          │ │ • Account Lists │        │
│  │   Worker        │ │   Partials      │ │ • UI Layout     │        │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘        │
│                                                                     │
│  🔑 CACHE KEY STRATEGIES                                            │
│  • "role:#{id}:permitted_accounts_cache:#{updated_at.to_i}"        │
│  • "group_#{id}_ui_layout_cache_#{accounting_configuration.ui...}"  │
│  • "staff#{id}_accounting_accounts"                                 │
│                                                                     │
│  ♻️ CACHE INVALIDATION                                              │
│  • Touch-based dependency invalidation                             │
│  • Event-driven cache clearing                                     │
│  • Smart partial cache expiration                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Security Architecture
```ascii
┌─────────────────────────────────────────────────────────────────────┐
│                        SECURITY ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🔐 AUTHENTICATION LAYERS                                           │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ USER AUTHENTICATION:                                        │     │
│  │ • Devise-based with session management                      │     │
│  │ • Confirmable email verification                            │     │
│  │ • Password complexity requirements                          │     │
│  │ • Session token invalidation                                │     │
│  │                                                             │     │
│  │ ADMIN AUTHENTICATION:                                       │     │
│  │ • TOTP (Time-based One-Time Password)                      │     │
│  │ • Email OTP as fallback                                    │     │
│  │ • Admin role hierarchies                                   │     │
│  │ • Audit trail for admin actions                            │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🛡️ AUTHORIZATION FRAMEWORK                                         │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │ CANCANCAN ABILITIES:                                        │     │
│  │ • Role-based permission system                              │     │
│  │ • Resource-level authorization                              │     │
│  │ • Dynamic permission calculation                            │     │
│  │ • Account-level access control                              │     │
│  │                                                             │     │
│  │ PERMISSION POLICIES:                                        │     │
│  │ • Version-controlled permission schemas                     │     │
│  │ • Capability-based role definitions                         │     │
│  │ • Industry-specific permission sets                         │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                     │
│  🔒 DATA PROTECTION                                                 │
│  • Subdomain-based tenant isolation                                │
│  • Database foreign key constraints                                │
│  • Input validation at multiple layers                             │
│  • SQL injection prevention                                        │
│  • XSS protection with Rails security features                     │
│  • CSRF token validation                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 💡 **Expert Architectural Assessment**

### Strengths
- **Sophisticated Multi-tenancy**: 4-level hierarchy with proper isolation
- **Domain-Driven Design**: Industry-specific customization with clean abstractions
- **Accounting Expertise**: Proper double-entry implementation with Taiwan standards
- **Modern Rails Patterns**: Service objects, form objects, presenters, decorators
- **Scalable Architecture**: Proper caching, background processing, queue management
- **Security-First Design**: Multi-layer authentication, comprehensive authorization

### Performance Optimizations
- Redis-based caching with intelligent invalidation strategies
- Background job processing with Sidekiq for heavy operations
- Database query optimization with custom query objects
- Asset pipeline optimization with fingerprinting
- Fragment caching for complex UI components

### Scalability Considerations
- Horizontal scaling capability through subdomain isolation
- Database sharding potential through group-based partitioning
- CDN-ready asset architecture
- Microservice extraction paths (payments, notifications, reporting)
- API-first design potential for mobile apps

This architecture demonstrates **enterprise-grade Ruby on Rails development** with sophisticated domain modeling, proper separation of concerns, and scalable multi-tenant SaaS patterns optimized for the accounting industry in Taiwan.