# 204-Azure-Associate-Developer

This is aggressive but realistic for someone with dev experience. 6–8 hours/day. If you're new to Azure, stretch to 10-15 days.

## Part 1: Azure App Service Web Apps (Most tested in compute domain)

### How Azure App Service Works (Deep Mechanics)

Azure App Service is PaaS (Platform as a Service). You upload code (or a container), and Azure handles everything else: VMs, OS patching, load balancing, SSL, etc.

**Under the hood:**
- Your app runs inside an **App Service Plan** — this is a pool of VMs (shared in lower tiers, dedicated in higher).
- Each web app is isolated in its own sandbox (worker process).
- When you scale out, Azure spins up more worker processes across the plan.
- **Always On** feature keeps the app warm (prevents cold starts in Free/Shared).

### Key Concepts:

**Plans & Tiers:**

| Tier | Scaling | Slots | VNet Integration | Exam Use Case |
|------|---------|-------|------------------|---------------|
| Free/Shared | No | No | No | Dev/test only |
| Basic | Manual scale up | No | No | Small apps |
| Standard | Auto-scale out + up | Yes (5 slots) | Yes | Most exam scenarios |
| Premium | Advanced auto-scale | More slots | Yes + Private Endpoints | Production with high traffic |

**Deployment Slots (HUGE on exam — 30% of App Service questions):**
- Slots are separate instances of your app (prod, staging, dev).
- Deploy to staging → test → swap to production (zero downtime).
- During swap: Azure warms up the target slot first, then swaps routing.
- Settings: **Slot settings** (e.g., connection strings) stay with the slot and don't swap.

**Scaling:**
- **Scale up**: Bigger VM (more CPU/RAM).
- **Scale out**: More instances (rules based on CPU, memory, HTTP queue, custom metrics).

**Diagnostics:** Built-in logging to Blob, Application Insights, or Log Analytics.

### Visual: Architecture Diagram
[Azure App Service Architecture](https://medium.com/@lnnrtsvnssn/planning-the-architecture-of-your-web-application-6e73e94942d5)

### Visual: Deployment Slots (Blue-Green Deployment)
[Blue Green Deployment with Azure App Service Deployment Slots using Azure DevOps | by Sahitya Pai | Capgemini Microsoft Blog | Medium](https://medium.com/capgemini-microsoft-team/blue-green-deployment-with-azure-app-service-deployment-slots-using-azure-devops-f06583386178)

---

## Real Exam Questions Asked (2026 Patterns)

From recent exams (ExamTopics, Reddit, Tutorials Dojo Feb 2026 reports):

**Q1 (Very Common):**
> You have an Azure App Service web app named app1 in the Basic pricing tier. You need to create a deployment slot for testing. What should you do first?
> 
> A) Scale up the App Service plan  
> B) Create a new App Service plan  
> C) Enable Always On  
> D) Add a custom domain
> 
> **Correct: A**  
> **Why?** Slots require Standard tier minimum. Scale up first.

**Q2:**
> Your app needs zero-downtime deployment and traffic routing (10% to new version).
> 
> **Answer:** Use deployment slots + Traffic Routing in the portal (or CLI: `az webapp traffic-routing`).

**Q3 (Code Snippet):**
> Which setting makes a connection string slot-specific?
> 
> **Answer:** `WEBSITE_SLOT_STICKY` or mark as **Deployment slot setting** in Configuration.

---

## Practical Lab: Experiment with App Service (45–60 min)

### 1. Create the App

**Portal:**
- Search "App Services" → Create
- Resource Group: `rg-az204-day1`
- Name: `mywebapp-rl` (unique)
- Publish: Code
- Runtime: .NET 8 (or Node if you prefer)
- Plan: Create new → Standard S1

**CLI alternative:**
```bash
az appservice plan create --name myplan --resource-group rg-az204-day1 --sku S1
az webapp create --resource-group rg-az204-day1 --plan myplan --name mywebapp-rl
```

### 2. Deploy Code (Quick way)

- Go to your web app → **Deployment Center** → **GitHub** → Connect your repo (use a simple ASP.NET or Node sample).
- **Or:** Download sample from Microsoft → zip it → **Advanced Tools (Kudu)** → Drag & drop zip.

### 3. Create & Test Deployment Slot (Core experiment)

- In web app → **Deployment slots** → **Add Slot** → Name: `staging`
- Deploy a different version to staging (change the homepage text to "STAGING v2").
- **Test:** Go to `mywebapp-rl-staging.azurewebsites.net`
- **Swap:** Click Swap → Production. Watch zero downtime.
- **Experiment:** Add a slot-specific setting (Configuration → Add connection string → Check "Deployment slot setting").

### 4. Autoscaling Experiment

- Go to **Scale out** → Enable autoscale.
- **Rule:** CPU > 70% → Add 1 instance.
- Use Load testing tool (or just refresh many times) to trigger.

**What to Observe:** After swap, production shows v2. Check logs in **Diagnose and solve problems** → **Availability**.



# Part 2: Azure Functions (Second most tested)

## How Azure Functions Works (Deep)
Functions is event-driven serverless. You write only the code that matters — Azure runs it when an event happens.

### Core Flow:
1. Trigger fires (e.g., HTTP request, new blob).
2. Runtime executes your function.
3. Bindings inject data (no SDK calls needed for common cases).

### Hosting Plans (Exam killer):
| Plan | Scaling | Max Duration | Cold Start | VNet | Cost Model | Exam Scenario |
|------|---------|--------------|------------|------|------------|---------------|
| Consumption | Event-driven | 10 min | Yes | No | Per execution | Sporadic jobs |
| Premium | Pre-warmed | 60 min | No | Yes | Always-on instances | Long-running, low latency |
| App Service | Manual | Unlimited | No | Yes | Fixed | When you already have a plan |

### Triggers & Bindings (Memorize these):
* **Triggers:** HTTP, Timer (`0 0 * * * *` = hourly), Blob, Queue, CosmosDB (change feed), Service Bus, Event Grid.
* **Bindings:** Input (read), Output (write) — e.g., blob output binding writes to storage without code.
* **Durable Functions:** For workflows (orchestrator + activities). State is stored in Azure Storage.

 [Triggers & Bindings](https://notes.kodekloud.com/docs/AZ-204-Developing-Solutions-for-Microsoft-Azure/Developing-Azure-Functions/Triggers-and-Bindings/page)

[How Functions Runtime Works](https://learn.microsoft.com/en-us/azure/azure-functions/functions-custom-handlers)

## Real Exam Questions
**Q4:** Your function processes 2 GB files and takes 25 minutes. Which plan?  
**Answer:** Premium (Consumption max 10 min).

**Q5 (Code):**
```csharp
[FunctionName("ProcessBlob")]
public static void Run(
    [BlobTrigger("input/{name}")] string myBlob,  // Trigger
    [Blob("output/{name}")] out string outputBlob) // Output binding
```

## Practical Lab: Functions (60 min)

### 1. Create Function App
Portal: Search "Function Apps" → Create → Consumption plan → .NET 8.

### 2. HTTP Trigger Experiment
* In VS Code: Install Azure Functions extension → Create new project → HTTP trigger.
* Deploy → Test in portal (copy function URL).

### 3. Blob Trigger + Binding
* Create new function: Blob trigger.
* Upload a file to the linked storage container.
* Experiment: Change binding to output to a Queue — see message appear.

### 4. Timer Trigger
* CRON: `0 */5 * * * *` (every 5 min).
* Run and check logs.

**Experiment:** Switch to Premium plan (scale out) → notice no cold start.

# Azure Container Service 

## Azure Container Registry (ACR)

Azure Container Registry is a fully managed, private registry service in Azure for storing and managing Docker container images and other OCI (Open Container Initiative) artifacts. It's essentially Azure's equivalent to Docker Hub but with enterprise-grade features, security, and integration with other Azure services. ACR allows you to build, store, and distribute container images securely within your Azure environment, making it ideal for CI/CD pipelines in development workflows.

#### Key Components and How It Works
1. **Registry Structure**:
   - ACR is organized into repositories, similar to Docker Hub. Each repository can hold multiple images, tagged with versions (e.g., `myapp:v1`).
   - Supports OCI artifacts beyond just Docker images, such as Helm charts or other container-related files.

2. **Tiers and Pricing**:
   - Available in Basic, Standard, and Premium tiers.
     - **Basic**: For small-scale testing, limited storage and throughput.
     - **Standard**: Adds replication and higher performance.
     - **Premium**: Includes geo-replication (automatic syncing across regions for high availability), content trust (signed images), and vulnerability scanning via Microsoft Defender for Containers.
   - Billing is based on storage used and data transfer.

3. **Building Images with ACR Tasks**:
   - ACR Tasks is a built-in feature for building container images directly in the cloud, without needing a local Docker daemon.
   - You can trigger tasks manually, on a schedule, or via webhooks (e.g., from GitHub commits).
   - Supports multi-step tasks defined in YAML (e.g., build, test, push).
   - Example workflow: Commit code to Git → ACR Task builds image → Push to registry.

4. **Security and Access Control**:
   - Integrates with Azure Active Directory (AAD) for authentication.
   - Use managed identities or service principals for secure access from services like Azure DevOps or AKS (Azure Kubernetes Service).
   - Features like private endpoints and VNet integration prevent public exposure.
   - Content trust ensures images are signed and verified before deployment.
   - Built-in vulnerability scanning (in Premium) integrates with Microsoft Defender to detect issues in images.

5. **Integration with Other Azure Services**:
   - Seamless with Azure Kubernetes Service (AKS), Azure Web Apps for Containers, and Azure DevOps for CI/CD.
   - Use with Azure Container Instances (ACI) or Azure Container Apps (ACA) for pulling images directly.
   - Geo-replication ensures low-latency pulls from global locations.

6. **Use Cases for AZ-204 Exam**:
   - In the exam, expect scenarios on creating ACR, pushing/pulling images via Azure CLI/PowerShell (e.g., `az acr create`, `docker push`), setting up tasks for automated builds, and securing registries.
   - Common question: How to integrate ACR with AKS for private image pulls using managed identities.

#### Hands-On Example (via Azure CLI)
To create an ACR and push an image:
```
az acr create --resource-group myGroup --name myACR --sku Basic
docker login myACR.azurecr.io --username <username> --password <password>
docker tag myimage:latest myACR.azurecr.io/myimage:latest
docker push myACR.azurecr.io/myimage:latest
```

## Azure Container Instances (ACI)

Azure Container Instances is a serverless compute service that allows you to run containers on-demand without provisioning or managing underlying infrastructure like VMs or orchestrators (e.g., no Kubernetes needed). It's designed for quick, isolated container execution, making it suitable for bursty workloads, testing, or simple apps that don't require complex scaling.

#### Key Components and How It Works
1. **Container Groups**:
   - The core unit in ACI is a "container group," which is like a pod in Kubernetes. It can include one or more containers that share the same lifecycle, network, and storage.
   - Containers in a group can communicate via localhost and share volumes.

2. **Deployment and Runtime**:
   - Deploy via Azure Portal, CLI, ARM templates, or SDKs.
   - Specify CPU, memory, image from ACR/Docker Hub, ports, environment variables, and volumes.
   - ACI runs on Azure's hypervisor-isolated infrastructure for security (each group is isolated).
   - No persistent state by default; use Azure Files or other storage for persistence.

3. **Networking and Exposure**:
   - Public IP assignment for internet-facing containers.
   - Supports VNet integration for private networking (e.g., connect to Azure SQL without public exposure).
   - Load balancing via Azure Load Balancer or Application Gateway (manual setup required).

4. **Scaling and Management**:
   - No built-in auto-scaling; it's for on-demand runs. Scale manually by creating more instances.
   - Restart policies: Always, OnFailure, Never.
   - Monitoring via Azure Monitor (logs, metrics like CPU usage).

5. **Limitations**:
   - Not for long-running, stateful apps (better for stateless or short-lived tasks).
   - Max 60 GB storage per group, limited GPU support.

6. **Integration with Other Azure Services**:
   - Pull images from ACR.
   - Use with Logic Apps or Azure Functions for event-driven triggers.
   - Can be orchestrated lightly with Azure Container Apps or AKS for more complex scenarios.

7. **Use Cases for AZ-204 Exam**:
   - Exam focuses on deploying ACI for quick container runs, e.g., batch jobs or dev/testing.
   - Questions on YAML deployment specs, securing with managed identities, and integrating storage.
   - Scenario: Run a containerized script to process data without managing VMs.

#### Hands-On Example (via Azure CLI)
Deploy a simple container:
```
az container create --resource-group myGroup --name myACI --image mcr.microsoft.com/hello-world --dns-name-label myaci --ports 80
```

## Azure Container Apps (ACA)

Azure Container Apps is a serverless container platform built on top of Kubernetes (using Azure Kubernetes Service under the hood but abstracted away). It's designed for microservices and event-driven apps, offering automatic scaling, revisions, and integrations without requiring Kubernetes expertise. ACA is the "modern favorite" for serverless containers in Azure, evolving from ACI with more advanced features.

#### Key Components and How It Works
1. **Environments and Apps**:
   - **Container App Environment**: A secure boundary for apps, providing networking (VNet support), logging, and Dapr integration.
   - **Container Apps**: Individual apps deployed as containers. Each app can have multiple revisions.

2. **Scaling with KEDA**:
   - Uses Kubernetes Event-Driven Autoscaling (KEDA) for zero-to-many scaling.
   - Scalers: HTTP traffic, CPU/memory, queues (e.g., Azure Service Bus, Kafka), custom metrics.
   - Min replicas can be 0 for cost savings; scales up on triggers.

3. **Revisions and Deployments**:
   - Supports revisions like deployment slots in App Service—deploy new versions without downtime.
   - Traffic splitting: Route percentages to different revisions (e.g., 90% to stable, 10% to new).
   - Rollback to previous revisions easily.

4. **Dapr Integration**:
   - Distributed Application Runtime (Dapr) is built-in for microservices.
   - Sidecar pattern: Each app gets a Dapr sidecar for service discovery, pub/sub, state management, and secrets.
   - Enables building resilient apps without coding plumbing.

5. **Networking and Security**:
   - Internal/external endpoints, mTLS for secure communication.
   - Integrates with Azure API Management, Application Gateway.
   - Secrets management via Azure Key Vault.

6. **Monitoring and Logging**:
   - Built-in with Azure Monitor and Log Analytics.
   - Observability via OpenTelemetry.

7. **Limitations**:
   - Abstracts Kubernetes, so no direct access to pods/nodes.
   - Best for microservices; for full control, use AKS.

8. **Integration with Other Azure Services**:
   - Pull from ACR.
   - Event-driven with Azure Event Grid, Service Bus.
   - Complements Azure Functions for hybrid serverless setups.

9. **Use Cases for AZ-204 Exam**:
   - Exam emphasizes ACA for modern apps: Configuring scaling rules, using Dapr for microservices, managing revisions.
   - Scenarios: Build a scalable API with HTTP scaling, or event-driven worker apps.
   - Key: Understand differences from ACI (ACA is more scalable/orchestrated) and AKS (ACA is serverless).

#### Hands-On Example (via Azure CLI)
Create an environment and app:
```
az containerapp env create --name myEnv --resource-group myGroup --location eastus
az containerapp create --name myApp --resource-group myGroup --environment myEnv --image myACR.azurecr.io/myimage:latest --target-port 80 --ingress external
```

### Comparison Table

| Feature                  | ACR (Registry)                          | ACI (Instances)                        | ACA (Apps)                              |
|--------------------------|-----------------------------------------|----------------------------------------|-----------------------------------------|
| **Primary Purpose**     | Store and manage container images      | Run single/multiple containers quickly | Serverless microservices with scaling  |
| **Orchestration**       | None (just storage)                    | None (simple groups)                   | Built-in (Kubernetes-based, abstracted)|
| **Scaling**             | N/A                                    | Manual                                 | Auto with KEDA (HTTP, queues, etc.)    |
| **Revisions/Slots**     | Image tags/versions                    | None                                   | Built-in revisions with traffic split  |
| **Integrations**        | CI/CD, AKS, ACI, ACA                   | Basic (storage, networking)            | Dapr, events, API Mgmt                 |
| **Use Case**            | Image repository in pipelines          | Quick tests/batch jobs                 | Production microservices               |
| **Exam Relevance**      | Building/pushing images, security      | On-demand container runs               | Modern serverless, scaling, Dapr       |

For AZ-204 prep, focus on hands-on labs via Azure Portal/CLI, and review official docs for updates. Practice scenarios like integrating these services in a CI/CD pipeline. If you have specific scenarios or questions, let me know!
