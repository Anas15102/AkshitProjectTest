# @tech-spec.md  

---  

## 1. Technology Stack  

| Layer | Approved Technology | Version (if known) | Reason for Choice | **Not Allowed** |
|-------|---------------------|--------------------|-------------------|-----------------|
| **Mobile UI** | **Flutter** (stable channel) | ≥ 3.13 | Single code‑base for iOS & Android, fast UI iteration, strong community, good for gamified experiences. | React Native, Kotlin/Swift native, Ionic |
| **State Management** | **Riverpod** (or **Provider** as fallback) | ≥ 2.4 | Compile‑time safety, testability, easy to integrate with async streams from backend. | Bloc (over‑engineered for MVP), Redux |
| **Graphics / Game Engine** | **Flame** (Flutter game engine) | ≥ 1.10 | Lightweight 2‑D engine, integrates with Flutter widgets, ideal for simple attention‑training mini‑games. | Unity, Godot (excessive for MVP) |
| **Authentication** | **AWS Cognito** (User Pools + Identity Pools) | Latest | Handles COPPA‑compliant sign‑up, MFA, social login (optional), integrates with AWS IAM for Lambda access. | Firebase Auth, Auth0 |
| **Backend Compute** | **AWS Lambda** (Python 3.12 runtime) | Latest | Serverless, pay‑per‑use, easy to scale, fits <1 month dev timeline. | EC2, Fargate (over‑kill) |
| **API Gateway** | **Amazon API Gateway** (REST) | Latest | Front‑door for Lambda, request validation, throttling, CORS handling. | AppSync (GraphQL) – adds complexity |
| **Database** | **Amazon DynamoDB** (NoSQL) | Latest | Low‑latency, fully managed, auto‑scales, fits session/metric storage. | RDS (SQL) for core data – not needed for MVP |
| **Analytics / Event Tracking** | **Amazon Pinpoint** (or **AWS Amplify Analytics**) | Latest | Push notifications, in‑app messaging, user‑level metrics. | Third‑party analytics (Mixpanel) – adds extra compliance work |
| **CI/CD** | **GitHub Actions** + **AWS SAM** (Serverless Application Model) | Latest | Automates Lambda deployment, infrastructure as code, integrates with Flutter build pipelines. | Bitbucket Pipelines, CircleCI (acceptable but not primary) |
| **Infrastructure as Code** | **AWS SAM** (YAML) | Latest | Simplifies Lambda, API Gateway, Cognito, DynamoDB provisioning. | Terraform (acceptable but not primary) |
| **Testing** | **flutter_test**, **mockito**, **pytest** (for Python) | Latest | Unit & widget testing for Flutter; unit/integration testing for Lambda. | Cypress (e2e web) – not applicable |
| **Compliance** | **AWS Artifact** (for COPPA/GDPR docs) + **AWS WAF** (basic rules) | N/A | Provides audit artifacts, helps enforce data protection. | Storing personal data in plain S3 without encryption |
| **Payments** | **Apple App Store** & **Google Play Billing** (in‑app subscription) | Latest | Required for iOS/Android subscription compliance, handles $9.99/mo recurring billing. | Direct Stripe integration (not allowed for mobile subscriptions) |
| **Logging / Monitoring** | **Amazon CloudWatch** (Logs & Metrics) | Latest | Centralized logs for Lambda, API Gateway, Cognito events. | External logging services (e.g., Loggly) – optional, not primary |

---

## 2. Architecture Overview  

```
+-------------------+          +-------------------+          +-------------------+
|   Flutter Mobile |  HTTPS   |  Amazon API GW    |  Invoke  |   AWS Lambda      |
|   (iOS/Android)  | <------> |   (REST endpoints)| <------> |   (Python)        |
+-------------------+          +-------------------+          +-------------------+
        |                                 |                         |
        |                                 |                         |
        |                                 v                         v
        |                         +-------------------+   +-------------------+
        |                         |   Amazon Cognito  |   |   DynamoDB Tables |
        |                         +-------------------+   +-------------------+
        |                                 |
        |                                 v
        |                         +-------------------+
        |                         |   Amazon Pinpoint |
        |                         +-------------------+
        |                                 |
        |                                 v
        |                         +-------------------+
        |                         |   CloudWatch Logs |
        |                         +-------------------+

```

* **Mobile App** – Built with Flutter, uses Riverpod for state, Flame for game rendering. Handles COPPA‑compliant onboarding (age verification, parental consent UI) and authenticates via Cognito SDK.  
* **API Layer** – Amazon API Gateway exposes a small set of REST endpoints (`/session`, `/metrics`, `/profile`, `/notifications`). Request validation (JSON schema) enforced at the gateway.  
* **Business Logic** – AWS Lambda functions (Python) process game session data, compute adaptive difficulty, store results in DynamoDB, and trigger push notifications via Pinpoint.  
* **Data Store** – DynamoDB tables: `Users`, `ChildrenProfiles`, `GameSessions`, `Metrics`. All tables encrypted at rest (AWS KMS).  
* **Authentication** – Cognito User Pools store parent accounts; each parent can create multiple child profiles (stored in DynamoDB).  
* **Payments** – In‑app subscription handled by Apple/Google stores; receipt validation performed in a dedicated Lambda (`validate_receipt`).  
* **Analytics & Notifications** – Pinpoint sends scheduled reminders, progress alerts, and promotional messages (opt‑in required).  
* **Observability** – CloudWatch collects Lambda logs, API latency metrics, and custom metrics (e.g., daily active users).  

---

## 3. Key Technical Decisions  

| Decision | Rationale |
|----------|-----------|
| **Flutter over React Native** | Single Dart codebase, faster UI iteration, built‑in support for high‑performance 2‑D graphics via Flame, aligns with <1 month MVP timeline. |
| **Serverless (Lambda + API GW) instead of traditional servers** | No provisioning, auto‑scaling, lower ops overhead, cost‑effective for low‑to‑moderate traffic, matches developer’s Python skillset. |
| **DynamoDB (NoSQL) for core data** | Flexible schema for evolving game metrics, low latency reads/writes, eliminates need for complex migrations during rapid MVP changes. |
| **Cognito for authentication** | Provides out‑of‑the‑box COPPA‑friendly user pools, MFA, and easy federation with Apple/Google sign‑in if needed later. |
| **In‑app subscription via App Store / Play Store** | Mandatory for iOS/Android, simplifies PCI compliance, handles recurring billing automatically. |
| **Riverpod for state management** | Compile‑time safety, less boilerplate than Bloc, easier to test, suitable for small to medium apps. |
| **Flame game engine** | Lightweight, integrates with Flutter UI, ideal for simple attention‑training mini‑games without the overhead of Unity. |
| **AWS SAM for IaC** | Tight integration with Lambda & API GW, reduces deployment friction, single YAML file for all resources. |
| **COPPA compliance via parental consent flow + data minimization** | Legal requirement for children <13 in US; ensures data collection is limited to necessary metrics, stored encrypted, and never shared with third parties. |
| **Push notifications via Pinpoint** | Native integration with both iOS and Android, supports segmentation (e.g., reminder schedules), and is serverless. |
| **No external SQL database** | MVP does not need relational joins; DynamoDB suffices and reduces operational complexity. |
| **No third‑party analytics SDKs** | Avoids extra privacy concerns; rely on Pinpoint and internal metrics stored in DynamoDB. |

---

## 4. Project Structure  

```
/project-root
│
├─ /mobile                     # Flutter app
│   ├─ lib/
│   │   ├─ main.dart
│   │   ├─ src/
│   │   │   ├─ app.dart                # MaterialApp & Riverpod ProviderScope
│   │   │   ├─ routes.dart
│   │   │   ├─ features/
│   │   │   │   ├─ onboarding/
│   │   │   │   │   ├─ view/
│   │   │   │   │   └─ controller/
│   │   │   │   ├─ child_game/
│   │   │   │   │   ├─ view/
│   │   │   │   │   ├─ controller/
│   │   │   │   │   └─ models/
│   │   │   │   ├─ parent_dashboard/
│   │   │   │   │   └─ ...
│   │   │   ├─ services/
│   │   │   │   ├─ api_service.dart      # Wrapper around API GW calls
│   │   │   │   ├─ auth_service.dart     # Cognito SDK
│   │   │   │   └─ notification_service.dart
│   │   │   ├─ utils/
│   │   │   └─ widgets/
│   │   └─ test/
│   │       └─ unit/ & widget/
│   ├─ ios/
│   ├─ android/
│   └─ pubspec.yaml
│
├─ /backend                    # Serverless backend (Python)
│   ├─ src/
│   │   ├─ handlers/
│   │   │   ├─ session_handler.py
│   │   │   ├─ metrics_handler.py
│   │   │   ├─ profile_handler.py
│   │   │   └─ receipt_validation.py
│   │   ├─ models/
│   │   │   └─ dynamo_models.py
│   │   ├─ utils/
│   │   │   ├─ cognito_helper.py
│   │   │   └─ pinpoint_helper.py
│   │   └─ requirements.txt
│   ├─ tests/
│   │   └─ unit/
│   └─ template.yaml           # SAM template (infrastructure)
│
├─ /infra                      # IaC (if separate from SAM)
│   └─ sam/
│       └─ template.yaml
│
├─ .github/
│   └─ workflows/
│       └─ ci-cd.yml          # GitHub Actions for Flutter + SAM deploy
│
├─ README.md
└─ .gitignore
```

*All folder names are lower‑snake_case for consistency.*

---

## 5. Dependencies  

### Mobile (Flutter)  



---
*Generated by CollabLearn on 2026-01-19*