# Software Development Life Cycle (SDLC)

## What is SDLC?

SDLC (Software Development Life Cycle) is the process followed by software companies to build and deliver high-quality software.

The main goal of SDLC is to **deliver a high-quality product** that meets customer requirements.

---

## Main Phases of SDLC

1. Planning & Requirement Gathering
2. Requirement Analysis / Documentation
3. Designing (HLD & LLD)
4. Building / Development
5. Testing
6. Deployment

---

## Example

Suppose I work at **XYZ Company**, and my team is building an **E-commerce Website**.

The project follows this cycle:

```text
                  6. Deployment
                       ▲
                       │
5. Testing ◄────── 4. Development
     ▲                    │
     │                    │
     └────────────────────┘

1. Planning & Requirements
            │
            ▼
2. Requirement Documentation
            │
            ▼
3. Designing (HLD & LLD)
```

After deployment, customer feedback is collected, and the cycle starts again for new features or improvements.

---

## Understanding Each Phase

### 1. Planning & Requirement Gathering

- Understand customer requirements.
- Discuss project goals.
- Estimate time and resources.
- Decide project scope.

---

### 2. Requirement Documentation

- Document all business requirements.
- Create Software Requirement Specification (SRS).
- Ensure everyone understands the requirements before development starts.

---

### 3. Designing (HLD & LLD)

Design the system architecture before coding.

- **HLD (High-Level Design):** Overall system architecture.
- **LLD (Low-Level Design):** Detailed implementation of individual modules.

---

### 4. Building / Development

**Responsible Team:** Developers

- Write application code.
- Implement required features.
- Push code to Git repositories.
- Fix bugs and improve functionality.

---

### 5. Testing

**Responsible Team:** QA / QE Engineers

- Test the application thoroughly.
- Verify that all features work correctly.
- Detect bugs and report them.
- Ensure the application is ready for deployment.

The application is usually deployed to a **testing/staging environment**, where the QA team validates everything before production.

---

### 6. Deployment

Deploy the tested application to the **Production Environment**, where end users can access it.

This is the final stage before customers start using the application.

---

# Where Does DevOps Fit?

The main responsibility of a DevOps Engineer is to make the **Development → Testing → Deployment** process faster, smoother, and fully automated.

Instead of performing these tasks manually, DevOps engineers automate the workflow using CI/CD pipelines and various DevOps tools.

Their goal is to:

- Reduce manual intervention.
- Automate repetitive tasks.
- Speed up software delivery.
- Ensure reliable deployments.
- Reduce deployment failures.

> In simple words, a DevOps Engineer ensures that software moves from **Developer → Testing → Production** quickly, safely, and automatically.

---

# Automation Phase

The following phases are commonly automated using DevOps practices:

- Development (Build Process)
- Testing
- Deployment

This automation is achieved using tools such as Git, Jenkins, GitHub Actions, Docker, Kubernetes, Terraform, etc.

---

# SDLC Models

Different organizations follow different SDLC models.

## 1. Waterfall Model

- Linear and sequential process.
- Each phase must be completed before moving to the next.
- No going back to previous phases.
- Best suited for projects with fixed requirements.

---

## 2. Iterative Model

- Software is developed in multiple iterations.
- Each iteration adds new features.
- Customer feedback is incorporated after every iteration.
- Easier to improve the product over time.

---

## 3. Agile Model

- Work is divided into small iterations called **Sprints**.
- Frequent releases.
- Continuous customer feedback.
- Most widely used SDLC model in modern software companies.
- Works well with DevOps practices.