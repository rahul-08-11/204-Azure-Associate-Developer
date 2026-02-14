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
