# AWS-Services-Prep
All new repo to prepare for AWS Interview and Exams

# AWS AppConfig – Interview Questions

A focused set of AWS AppConfig interview questions, modeled similarly to the SQS & SNS Q&A style, covering fundamentals, architecture, integration, and advanced topics.

---

## 1. Basics And Core Concepts

**Q1. What is AWS AppConfig and why is it used?**  
**A.** AWS AppConfig is a capability of AWS Systems Manager that lets you create, manage, and deploy application configuration in a controlled, validated, and auditable way.  
It is used to:
- Decouple configuration from code and deployments
- Safely roll out configuration changes (e.g., feature flags, tuning parameters, endpoints)
- Validate configuration before rollout
- Monitor for issues and automatically roll back if needed

---

**Q2. How is AppConfig different from storing configuration in SSM Parameter Store or AWS Secrets Manager directly?**  
**A.**
- **SSM Parameter Store / Secrets Manager**: Primarily key–value stores for parameters and secrets.
- **AppConfig**:
  - Adds **deployment orchestration** (strategies, rollout %, bake time)
  - Provides **built‑in validation** (JSON schema, Lambda validators)
  - Integrates with **CloudWatch alarms** for automatic rollback
  - Focuses on **runtime config changes** that can be rolled out safely across environments  
In practice you often **store data in Parameter Store or S3**, and **use AppConfig to deploy and control it**.

---

**Q3. What are the main building blocks of AppConfig?**  
**A.**
1. **Application** – Logical group of configurations (e.g., `payments-service`, `web-frontend`).
2. **Environment** – Where config is consumed (e.g., `dev`, `staging`, `prod`).
3. **Configuration Profile** – Points to a configuration **source** (SSM Parameter Store, SSM document, S3 object, or feature flags) and defines validators.
4. **Deployment Strategy** – How to roll out (percentage, step interval, bake time).
5. **Deployment** – An execution of a config rollout to an environment using a strategy.

---

**Q4. What are common configuration sources supported by AppConfig?**  
**A.**
- **AWS Systems Manager (SSM) Parameter Store** – JSON or text parameters
- **SSM Documents**
- **Amazon S3** – Files such as JSON, YAML, or text
- **Feature flags** – Native feature flag configuration type in AppConfig
- **Hosted configuration** – Configuration stored directly in AppConfig

---

## 2. Deployment Strategies And Safety

**Q5. What is a deployment strategy in AppConfig?**  
**A.** A deployment strategy defines **how** a new configuration version is rolled out to an environment. Key aspects:
- **Growth type** (e.g., linear, exponential)
- **Initial and final deployment percentage**
- **Step interval** (time between increments)
- **Bake time** (time to watch metrics before considering deployment successful)

---

**Q6. How does AppConfig reduce the risk of bad configuration changes?**  
**A.**
- Uses **validators** (JSON schema or Lambda) to catch invalid config before deployment.
- Rolls out changes **gradually** using deployment strategies instead of “big‑bang”.
- Integrates with **CloudWatch alarms**: if an alarm triggers (e.g., error rate spike), AppConfig can **automatically roll back**.
- Allows **manual rollback** to a previous config version.

---

**Q7. What is bake time in an AppConfig deployment and why is it important?**  
**A.**  
- **Bake time** is the period after the configuration has been fully deployed during which AppConfig continues to monitor CloudWatch alarms.
- If alarms trigger during bake time, AppConfig can **roll back** the deployment.
- It ensures you don’t mark a deployment as healthy too quickly and miss delayed issues.

---

**Q8. How do you roll back an AppConfig deployment?**  
**A.**
1. AppConfig can **automatically roll back** when a configured CloudWatch alarm goes into `ALARM` state during deployment or bake time.
2. You can also **manually start a new deployment** using a **previous configuration version** (e.g., from the console or CLI), effectively rolling back to a known good config.

---

## 3. Validation And Governance

**Q9. What types of validators does AppConfig support?**  
**A.**
1. **JSON Schema Validator** – Validate that configuration conforms to a JSON schema (types, required fields, allowed values).
2. **Lambda Validator** – Invoke a custom Lambda function to perform arbitrary validation logic (e.g., cross‑field rules, external checks).

---

**Q10. Why is validation in AppConfig better than validating inside the application code?**  
**A.**
- Validation happens **before deployment** reaches your fleet, reducing blast radius.
- Centralized validators can enforce **organization‑wide rules**.
- Avoids shipping invalid config to all services and then failing at runtime.
- Decouples config validation logic from application releases.

---

**Q11. How does AppConfig integrate with CloudWatch for monitoring and rollback?**  
**A.**
- When creating a deployment, you can associate one or more **CloudWatch alarms**.
- During rollout and bake time, AppConfig monitors those alarms:
  - If an alarm enters `ALARM` state, AppConfig can **stop and roll back** the deployment.
- Typical alarms track:
  - 5xx error rate
  - Latency p95/p99
  - Throttling or specific business KPIs

---

## 4. Integration Patterns

**Q12. How do applications typically consume configuration from AppConfig?**  
**A.** Common patterns:
- Using the **AWS AppConfig Agent** (sidecar) that polls config and exposes it on localhost.
- Using **AWS SDK / AppConfig data plane APIs** directly from the application.
- Using the **Lambda extension** for AppConfig in AWS Lambda functions.
- Caching configuration in‑memory and refreshing periodically or on demand.

---

**Q13. How would you use AppConfig with a microservices architecture?**  
**A.**
- Create an **Application** per domain or microservice group (e.g., `payments`, `user-profile`).
- Create **Environments** like `dev`, `test`, `staging`, `prod`.
- For each microservice, define **Configuration Profiles** for:
  - Feature flags
  - External endpoints
  - Tunable parameters (timeouts, limits)
- Each microservice pulls only the relevant configuration for its application and environment, using deployment strategies to safely roll out changes.

---

**Q14. How does AppConfig complement feature flagging?**  
**A.**
- AppConfig supports **Feature Flags** as a first‑class configuration type.
- You can:
  - Define feature flags with states, constraints, and targeting rules.
  - Roll out flags gradually (e.g., 5% → 25% → 100% of traffic).
  - Combine with CloudWatch alarms to rollback a flag rollout if issues occur.
- This provides **centralized, governed feature flag management** across services.

---

**Q15. How would you integrate AppConfig with AWS Lambda?**  
**A.**
- Use the **AppConfig Lambda extension** to pull configuration from AppConfig with low latency and caching.
- Lambda initialization phase (or periodically) calls AppConfig to fetch the latest config.
- The function code reads configuration from the extension/local cache instead of hitting AppConfig on every invocation, reducing cold‑start and latency impact.

---

## 5. Security, Access Control, And Compliance

**Q16. How is access control handled for AppConfig?**  
**A.**
- **IAM** policies control:
  - Who can create/update Applications, Environments, Profiles, and Deployments.
  - Which roles or services can **read** configuration (data‑plane APIs).
- You can use **resource‑level permissions** to restrict specific Applications/Environments.
- Combine with **AWS Organizations SCPs** for central governance in multi‑account setups.

---

**Q17. Where is configuration data actually stored and how is it secured?**  
**A.**
- Configuration can be stored in:
  - **SSM Parameter Store** (encrypted with KMS)
  - **S3** (encrypted at rest using SSE‑S3 or SSE‑KMS)
  - **AppConfig hosted configuration** (encrypted)
- Data in transit is secured using **TLS** when accessed via AWS APIs.
- AppConfig manages **versions** of configurations so you can audit and roll back.

---

**Q18. How do you handle secrets with AppConfig?**  
**A.**
- Recommended pattern:
  - Store **secrets** in **AWS Secrets Manager** or **Parameter Store (SecureString)**.
  - In AppConfig, store **references** to those secrets (e.g., parameter names, ARNs), not the secrets themselves.
- Application retrieves:
  - Non‑sensitive config from AppConfig
  - Secrets directly from Secrets Manager/Parameter Store using appropriate IAM permissions

---

## 6. Operational And Design Questions

**Q19. When would you prefer AppConfig over environment variables or config files baked into the container image?**  
**A.**
- When you need to:
  - **Change configuration without redeploying code**.
  - **Gradually roll out** config changes with the ability to rollback.
  - **Validate** configuration centrally before rollout.
  - Maintain **history and versioning** of configuration.
- Environment variables or baked config are fine for static values, but not for **frequently changing or risky** config.

---

**Q20. Can AppConfig be used across multiple AWS accounts and regions? How would you design that?**  
**A.**
- AppConfig is **regional**, but you can:
  - Use **separate AppConfig Applications/Environments per account/region**.
  - Or use a **central config account** and allow cross‑account access via IAM roles and resource policies.
- Design considerations:
  - Latency between services and AppConfig region.
  - Blast radius: account/region isolation for critical production config.
  - Organizational governance – who owns and approves config changes.

---

**Q21. How do you version and track configuration changes with AppConfig?**  
**A.**
- Each configuration update creates a new **version**.
- You can:
  - View version history in the AppConfig console.
  - Tag versions with metadata (e.g., `release-2025-12-04`, `canary`, `hotfix`).
  - Use audit trails via **AWS CloudTrail** for who changed what and when.
- Rollback is done by redeploying a **previous version**.

---

**Q22. What are some common design patterns using AppConfig in real systems?**  
**A.**
- **Feature flag management** for gradual rollout.
- **Dynamic routing or endpoint configuration** for blue/green deployments.
- **Rate limit / quota configuration** without redeploying services.
- **Per‑environment tuning** of timeouts, retries, and thresholds.
- **Business rules configuration** (e.g., discount percentages, thresholds for alerts) with validation to avoid invalid values.

---

## 7. Comparison And Trade‑offs

**Q23. How does AppConfig compare to open‑source solutions like LaunchDarkly or ConfigCat?**  
**A.** (High‑level points you can mention)
- **Pros:**
  - Native AWS integration (IAM, CloudWatch, CloudTrail, SSM, Lambda).
  - Pay‑as‑you‑go pricing, no separate SaaS to manage in an AWS‑centric stack.
  - Strong infra‑as‑code support via CloudFormation, CDK, Terraform.
- **Cons:**
  - Might lack some advanced targeting rules and UI polish vs specialized feature‑flag products.
  - Vendor lock‑in to AWS.

---

**Q24. In what scenarios would AppConfig be a bad fit or overkill?**  
**A.**
- Very simple apps with static configuration that rarely changes.
- Systems that already use a centralized, non‑AWS config system that meets all requirements.
- Config that is purely local and not risky (e.g., static UI labels).
- If you just need a **single parameter** and don’t require rollout/validation, SSM Parameter Store alone may be simpler.

---

## 8. Hands‑On / Implementation‑Style Questions

**Q25. Describe how you would set up AppConfig for a new microservice from scratch.**  
**A.** A good answer would include:
1. **Create an AppConfig Application** named after the service (e.g., `orders-service`).
2. **Create Environments**: `dev`, `staging`, `prod`.
3. Decide on **configuration source** (Parameter Store, S3, hosted config).
4. Define **Configuration Profiles**:
   - `feature-flags`
   - `service-params`
5. Add **validators**:
   - JSON schema for structure.
   - Optional Lambda for advanced business rules.
6. Create or reuse **Deployment Strategies**:
   - Fast rollout in `dev`, slower canary in `prod` with bake time.
7. Integrate application with **AppConfig Agent or SDK** to fetch config.
8. Set up **CloudWatch alarms** for key metrics and link them to deployments.
9. Automate via **CDK/CloudFormation** for repeatable infra.

---

You can extend this with more scenario‑based questions (e.g., “You deployed a bad config that broke only one tenant. How do you recover?”) in the same style as needed.
