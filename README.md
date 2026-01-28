# Security Maturity by Design

> A Risk-Driven AppSec Demonstration

A task management API demonstrating application security as a maturity spectrum rather than a binary state. This project operates in three intentional security modes — **MVP (Startup)**, **Production**, and **High Assurance** — each representing a realistic stage in a product's lifecycle.

## 🎯 Core Philosophy

This project demonstrates that:
- Not every risk is fixed immediately
- Not every service deserves the same controls
- Security decisions are constrained by velocity, cost, and usability

Instead of treating security as "secure vs insecure," this application makes security trade-offs **explicit, observable, and testable**.

## 📋 What This Application Does

**Domain:** Task Management API

A simple but realistic application for managing tasks within organizations. It includes:
- User authentication and authorization
- Multi-tenant organization support
- Task creation, assignment, and management
- Role-based access control
- Private/public task visibility

**Why this domain?**
- Natural multi-tenancy (users belong to organizations)
- Clear authorization boundaries
- Realistic business logic
- Common attack surfaces (IDOR, privilege escalation, data leaks)

## 🔐 Security Modes

### 🔴 MVP / Startup Mode
**Goal:** Ship fast with minimal friction

**Active Controls:**
- Basic bearer token authentication
- HTTPS enforced

**Known Vulnerabilities:**
- ❌ Broken Object Level Authorization (IDOR)
- ❌ No rate limiting
- ❌ Verbose error messages exposing internals
- ❌ Long-lived tokens (never expire)
- ❌ Mass assignment vulnerabilities

**Purpose:** Demonstrates how early-stage products expose common risks for velocity.

---

### 🟡 Production Mode
**Goal:** Reduce real-world attack paths without harming developer velocity

**Active Controls:**
- ✅ JWT-based authentication (1hr expiry)
- ✅ Object-level authorization
- ✅ Rate limiting (100 req/min)
- ✅ Input validation and sanitization
- ✅ Security headers (CSP, HSTS, X-Frame-Options)
- ✅ Generic error messages
- ✅ Basic access logging

**Mitigated Threats:**
- ✅ IDOR/BOLA attacks blocked
- ✅ Brute force attacks prevented
- ✅ Mass assignment vulnerabilities fixed
- ✅ XSS via reflected input blocked

**Purpose:** Represents how most real SaaS applications operate.

---

### 🟢 High Assurance Mode
**Goal:** Maximize assurance under stricter threat models

**Active Controls:**
- ✅ JWT with 15min expiry + IP binding
- ✅ Multi-layer authorization (org + role + resource)
- ✅ Adaptive rate limiting (30 req/min baseline)
- ✅ Strict input validation with type checking
- ✅ Comprehensive audit logging
- ✅ Security headers + strict CSP
- ✅ Deny-by-default authorization

**Additional Protections:**
- ✅ Token theft/reuse prevention
- ✅ Privilege escalation detection
- ✅ Data exfiltration monitoring
- ✅ Insider threat detection

**Purpose:** Shows how regulated or high-risk systems operate (healthcare, finance, government).

## 🎪 Key Features

### 1. Self-Documenting Security
The `/security-info` endpoint returns:
- Current security mode
- Active security controls
- Known vulnerabilities (in MVP mode)
- Accepted risks and rationale
- Performance impact
- Recommendations for usage

### 2. Proof, Not Claims
Includes **attack simulation scripts** that demonstrate:
- ✅ Successful exploitation in MVP mode
- ❌ Blocking in Production mode  
- ❌ Blocking + logging + alerting in High Assurance mode

### 3. Risk vs Cost Awareness
Each mode documents:
- Performance impact
- Developer effort required
- Operational complexity
- Primary threat classes mitigated

## 🏗️ Architecture

### Core Entities
```
User
├── id
├── email
├── password_hash
├── role (admin, member, viewer)
└── organization_id

Organization
├── id
├── name
└── plan_tier (free, pro, enterprise)

Task
├── id
├── title
├── description
├── status (open, in_progress, done)
├── created_by (user_id)
├── assigned_to (user_id)
├── organization_id
└── is_private (bool)
```

### API Endpoints

**Authentication:**
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh-token`
- `POST /auth/logout`

**Tasks:**
- `GET /tasks` - List tasks
- `POST /tasks` - Create task
- `GET /tasks/:id` - View task
- `PUT /tasks/:id` - Update task
- `DELETE /tasks/:id` - Delete task
- `POST /tasks/:id/assign` - Assign to user

**Users:**
- `GET /users` - List org users
- `GET /users/:id` - View user profile
- `PUT /users/:id/role` - Change role (admin only)

**Security:**
- `GET /security-info` - Current mode + active controls
- `GET /health` - Status check

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- Git

### Running in MVP Mode
```bash
git clone https://github.com/rishikumarbommakanti-ops/security-maturity-by-design
cd security-maturity-by-design
docker-compose up
```

### Check Security Status
```bash
curl http://localhost:8000/security-info
```

### Try an Attack
```bash
./exploits/idor_exploit.sh
# ✅ Returns all tasks (even from other orgs)
```

### Switch to Production Mode
```bash
docker-compose down
docker-compose --env-file .env.production up
```

### Same Attack Now Fails
```bash
./exploits/idor_exploit.sh
# ❌ Returns 403 Forbidden
```

## 🎯 Attack Simulations

### 1. IDOR Exploitation
**Exploits:** `./exploits/idor_exploit.sh`

**Results:**
- 🔴 MVP Mode: ✅ Exfiltrates all tasks across organizations
- 🟡 Production: ❌ Returns 403 for unauthorized tasks
- 🟢 High Assurance: ❌ Blocks + logs + may trigger alerts

### 2. Rate Limit Bypass
**Exploits:** `./exploits/rate_limit_test.py`

**Results:**
- 🔴 MVP Mode: ~1000 req/sec (unlimited)
- 🟡 Production: Limited to 100 req/min
- 🟢 High Assurance: Limited to 30 req/min + adaptive throttling

### 3. Privilege Escalation
**Exploits:** `./exploits/privilege_escalation.sh`

**Results:**
- 🔴 MVP Mode: ✅ Regular user promotes self to admin
- 🟡 Production: ❌ Returns 403 (admin check enforced)
- 🟢 High Assurance: ❌ Blocks + logs + alerts security team

## 💡 Why This Project Works

**1. Solves a Real Portfolio Problem**
Most AppSec portfolios show either:
- Overly technical CTF writeups (shows hacking, not building)
- Theoretical threat models (shows thinking, not doing)
- Isolated tool demos (shows features, not judgment)

This shows **decision-making under constraint** — what the job actually is.

**2. Immediately Interview-Ready**
When asked "walk me through your security thinking," you can:
- Pull up the app
- Toggle between modes
- Show why each control exists (or doesn't)

**3. Demonstrates Business Awareness**
Shows you understand:
- Security isn't free
- Trade-offs are necessary
- Context matters

## 📚 What This Demonstrates

✅ Threat modeling skills  
✅ Secure design thinking  
✅ Risk-based decision making  
✅ Developer empathy  
✅ AppSec automation awareness  
✅ Security evolution over time  
✅ Communication skills  

## 🎓 Educational Value

This project teaches:
- How to implement role-based vs attribute-based authorization
- When rate limiting matters (and when it doesn't)
- How to balance security telemetry with performance
- Why some vulnerabilities are worth accepting (temporarily)
- How to communicate security decisions to developers

## 🛠️ Tech Stack

- **Backend:** Python (FastAPI/Flask)
- **Database:** PostgreSQL
- **Auth:** JWT (python-jose)
- **Rate Limiting:** Flask-Limiter / slowapi
- **Logging:** Structured JSON logs
- **Deployment:** Docker + Docker Compose

## 📁 Project Structure

```
security-maturity-demo/
├── README.md
├── docker-compose.yml
├── requirements.txt
├── app/
│   ├── main.py
│   ├── config.py
│   ├── models.py
│   ├── auth/
│   │   ├── mvp.py
│   │   ├── production.py
│   │   └── high_assurance.py
│   ├── middleware/
│   │   ├── rate_limiting.py
│   │   ├── logging.py
│   │   └── validation.py
│   └── routes/
│       ├── auth.py
│       ├── tasks.py
│       ├── users.py
│       └── security_info.py
├── exploits/
│   ├── idor_exploit.sh
│   ├── rate_limit_test.py
│   ├── privilege_escalation.sh
│   └── README.md
├── tests/
│   ├── test_mvp_mode.py
│   ├── test_production_mode.py
│   └── test_high_assurance_mode.py
└── docs/
    ├── ARCHITECTURE.md
    ├── SECURITY_DECISIONS.md
    └── MODE_COMPARISON.md
```

## 🎤 One-Line Summary

> "A single application demonstrating how security controls evolve across MVP, Production, and High-Assurance environments, making risk trade-offs explicit and testable."

## 📝 Non-Goals

This project does NOT aim to:
- ❌ Demonstrate every OWASP Top 10 vulnerability
- ❌ Build a CTF-style challenge environment
- ❌ Show penetration testing skills
- ❌ Implement enterprise IAM from scratch

Instead, it focuses on **security decision-making at the application layer**.

## 🤝 Contributing

Contributions welcome! Areas for improvement:
- Additional security modes
- More attack simulations
- Additional features (2FA, SSO, RBAC extensions)
- Documentation improvements

## 📄 License

MIT License - see LICENSE file for details

## 👤 Author

**Rishi Kumar Bommakanti**
- SOC Analyst & AppSec Enthusiast
- Focused on AI + Cybersecurity
- GitHub: [@rishikumarbommakanti-ops](https://github.com/rishikumarbommakanti-ops)

---

**Interview Talking Point:**  
*"I built this to demonstrate that security isn't about being 'secure' or 'insecure' — it's about making informed trade-offs based on risk, resources, and business context. This app proves I can not only identify vulnerabilities, but also understand when and how to fix them."*
