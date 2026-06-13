# Multi-Tenant Enterprise AI Gateway (SaaS)

Platforma SaaS w formie API, umożliwiająca różnym firmom (tenantom) bezpieczne korzystanie z wbudowanych agentów AI do analizy tekstu. 
Całość jest w pełni zautomatyzowana, wyizolowana per klient i gotowa do wdrożenia na produkcję.

Projekt demonstruje nowoczesne podejście do budowy skalowalnych aplikacji AI w chmurze, 
integrując zaawansowane zarządzanie tożsamością, infrastrukturę jako kod (IaC) oraz pełne observability.

## 🚀 Tech Stack

- **Aplikacja:** Python, FastAPI, LangChain
- **AI / LLM:** AWS Bedrock (Claude 3 / Titan)
- **Tożsamość & Auth:** FusionAuth (Multi-Tenancy, SSO, JWT)
- **Infrastruktura:** AWS (ECS Fargate, ECR, VPC, ALB, DynamoDB)
- **IaC:** Terraform
- **CI/CD:** GitHub Actions (z użyciem AWS OIDC)
- **Observability:** Sentry (Error tracking), AWS CloudWatch (Logi, Metryki)
- **Konteneryzacja:** Docker (Multi-stage builds)

## 🏗️ Architektura Systemu

Poniższy diagram przedstawia przepływ danych i architekturę wdrożeniową na AWS.

```mermaid
graph TD
  subgraph "Klienci / Użytkownicy"
    ClientA[Firma A User]
    ClientB[Firma B User]
  end

  subgraph "Zarządzanie Tożsamością"
    FA[FusionAuth <br/> Multi-Tenant JWT]
  end

  subgraph "AWS Cloud (Terraform)"
    ALB[Application Load Balancer]
    ECS[ECS Fargate <br/> FastAPI + LangChain]
    Bedrock[AWS Bedrock <br/> Modele LLM]
    DDB[(DynamoDB <br/> Dane izolowane per Tenant)]
    CW[CloudWatch <br/> Logi i Metryki]
  end

  subgraph "Observability"
    Sentry[Sentry.io <br/> Śledzenie Błędów]
  end

  subgraph "CI/CD Pipeline"
    GH[GitHub Actions]
    OIDC[AWS IAM OIDC]
    ECR[Amazon ECR]
  end

  %% Przepływ użytkownika
  ClientA -->|1. Logowanie| FA
  ClientB -->|1. Logowanie| FA
  
  ClientA -->|2. Request API + JWT| ALB
  ClientB -->|2. Request API + JWT| ALB

  %% Logika wewnątrz AWS
  ALB --> ECS
  ECS -.->|Walidacja Tokenu| FA
  ECS -->|Prompty LangChain| Bedrock
  ECS -->|Zapis historii (Tenant ID)| DDB
  
  %% Monitoring
  ECS -->|Zgłaszanie wyjątków| Sentry
  ECS -->|Przesyłanie logów| CW

  %% Automatyzacja
  GH -->|Uwierzytelnianie bezkluczowe| OIDC
  GH -->|Push obrazu Docker| ECR
  ECR -->|Deploy nowej wersji| ECS
  GH -.->|Aktualizacja Infrastruktury| Terraform