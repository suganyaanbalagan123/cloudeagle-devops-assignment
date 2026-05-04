# CloudEagle DevOps Assignment

## Part 1 – CI/CD Design

### Branching Strategy

* `main` → Production
* `staging` → Staging environment
* `develop` → QA environment
* `feature/*` → Feature development

**Approach:**

* Developers work on feature branches and raise PRs to `develop`
* After QA validation, code is promoted to `staging`
* Production deployments happen only via `main`

**Production Safety:**

* `main` is a protected branch
* Mandatory PR approvals
* Jenkins includes a **manual approval step** before production deployment to prevent accidental releases

---

### Jenkins Pipeline Design

**Pipeline Stages:**

1. Checkout source code
2. Build application using Maven
3. Run unit tests
4. Build Docker image
5. Push image to container registry
6. Deploy to target environment
7. Perform basic health check

---

### PR vs Merge Behavior

**On Pull Request:**

* Build and unit tests are executed
* Static validation (optional: lint/sonar)
* No deployment is triggered

**On Merge:**

* Docker image is built and tagged
* Image is pushed to registry
* Deployment is triggered based on branch:

  * `develop` → QA
  * `staging` → Staging
  * `main` → Production (with approval)

---

### Rollback Strategy

* Each build produces a **versioned Docker image**
* Previous stable versions are retained in the registry
* In case of failure:

  * Redeploy the last stable image
  * Or use rollout rollback (in Kubernetes)

This ensures quick recovery with minimal downtime.

---

### Configuration Management

* Environment-specific configurations are handled using **Spring profiles**:

  * `application-qa.yml`
  * `application-staging.yml`
  * `application-prod.yml`

* Runtime configuration is injected using environment variables or ConfigMaps (in GKE)

This keeps code environment-independent.

---

### Secrets Management

* Sensitive data such as:

  * MongoDB credentials
  * API keys

  are stored in **GCP Secret Manager**

* Access is controlled via IAM roles

* Secrets are injected at runtime (not stored in code or repository)

---

### Deployment Strategy

**Rolling Deployment is used**

**Reason:**

* Ensures zero or minimal downtime
* Gradual replacement of old instances with new ones
* Supports health checks and automatic rollback

---

## Part 2 – Infrastructure Design

### Compute Choice – GKE (Google Kubernetes Engine)

**Reason for choosing GKE:**

* Built-in auto-scaling (HPA)
* Self-healing (failed pods restart automatically)
* Supports rolling deployments
* Suitable for multi-environment workloads

Compared to alternatives:

* Cloud Run → simpler but less control
* Compute Engine → more operational overhead

---

### Database – MongoDB Atlas

* Managed MongoDB service
* Provides:

  * Automatic backups
  * High availability
  * Easy scaling

Reduces operational overhead compared to self-hosted databases.

---

### Networking

* Application is deployed inside a **private VPC**
* Only the **Load Balancer** is publicly accessible
* Traffic flow:

User → Load Balancer (HTTPS) → GKE Ingress → Application Pods

* Internal communication happens via private networking

---

### Security

Security is implemented at multiple levels:

* HTTPS enforced via Load Balancer (SSL/TLS)
* Private VPC to isolate internal services
* Secrets managed via GCP Secret Manager
* IAM roles enforce least privilege access
* MongoDB Atlas secured with IP whitelisting
* Production deployments require manual approval

---

### Monitoring & Logging

* **GCP Cloud Logging** for application logs

* **Cloud Monitoring** for metrics:

  * CPU usage
  * Memory usage
  * Pod health

* Alerts configured for failures and anomalies

---

### Cost Optimization

* Use of auto-scaling to match demand
* Right-sized node configurations
* Avoid over-provisioning resources
* Option to use preemptible nodes for non-critical workloads

---

## Summary

This design focuses on:

* Safe and controlled deployments
* Scalability using Kubernetes
* Strong security practices
* Cost-efficient infrastructure

## Architecture Diagram

![Architecture](architecture-diagram.png)

The approach ensures the application can be deployed reliably while maintaining high availability and operational efficiency.
