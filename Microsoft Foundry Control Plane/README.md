# Lab 1 — Microsoft Foundry Control Plane

Hands-on labs that let AI Engineers and Solution Architects **experience the Foundry Control Plane** on a realistic enterprise scenario. All labs use **Jupyter notebooks** and authenticate with `DefaultAzureCredential` (via `az login`).

> **Docs**: [What is Microsoft Foundry Control Plane?](https://learn.microsoft.com/en-us/azure/foundry/control-plane/overview) · [Control Plane docs source](https://github.com/MicrosoftDocs/azure-ai-docs/tree/main/articles/foundry/control-plane)

---

## The Zava scenario

**Zava** is a fictitious online home & garden retailer running several AI agents on Microsoft Foundry:

| Agent | Purpose |
|-------|---------|
| `zava-support-bot` | Customer support — returns, orders, shipping |
| `zava-product-advisor` | Product recommendations from the Zava catalog |
| `zava-loyalty-agent` | Rewards points & tier inquiries |
| `zava-order-tracker` | Tool-calling agent that queries the order DB |

Zava's platform team needs **fleet-wide visibility, safety guardrails, evaluations, tracing, red-teaming, and quota/cost control** — exactly the promise of the Foundry Control Plane.

Sample Zava data lives in [data/](data/):
- [`zava_products.jsonl`](data/zava_products.jsonl) — product catalog
- [`zava_return_policy.md`](data/zava_return_policy.md) — grounding doc for the support bot
- [`zava_eval_dataset.jsonl`](data/zava_eval_dataset.jsonl) — query / ground-truth pairs for evaluations

---

## Lab index

| # | Notebook | Control Plane pane | What you'll do |
|---|----------|--------------------|----------------|
| 00 | [00-setup.ipynb](00-setup.ipynb) | — | Connect to your Foundry project with `DefaultAzureCredential`, verify environment |
| 01 | [01-fleet-inventory.ipynb](01-fleet-inventory.ipynb) | **Assets** | Discover Zava's agents, models, and tools across projects |
| 02 | [02-content-safety-guardrails.ipynb](02-content-safety-guardrails.ipynb) | **Compliance** | Apply Content Safety filters + create a guardrail policy for Zava |
| 03 | [03-evaluations.ipynb](03-evaluations.ipynb) | **Assets → Evaluation** | Run agent, quality, and risk-safety evaluators on `zava-support-bot` |
| 04 | [04-tracing-and-monitoring.ipynb](04-tracing-and-monitoring.ipynb) | **Assets → Monitoring** | Enable OpenTelemetry tracing + set up continuous evaluation |
| 05 | [05-red-teaming.ipynb](05-red-teaming.ipynb) | **Compliance / Security** | Use the AI Red Teaming Agent to probe Zava for jailbreak & harm risks |
| 06 | [06-quotas-and-cost.ipynb](06-quotas-and-cost.ipynb) | **Quota / Overview** | Inspect model quota, token usage, and cost signals |

---

## Prerequisites

Before running the notebooks, make sure you have:

1. **Python 3.10 or later**, Git, and VS Code with the Python and Jupyter extensions (or JupyterLab).
2. **Azure CLI** installed and authenticated to the subscription that contains the lab resources:
   ```powershell
   az login
   az account set --subscription "<your-subscription-id>"
   az account show --output table
   ```
3. **An Azure subscription and resource group** containing a Microsoft Foundry resource and project.
4. **A deployed chat model** in the project. The labs use its deployment name, not the model catalog name. A small model such as `gpt-4o-mini` is suitable for most exercises.
5. **A judge model deployment for lab 03**. Set `FOUNDRY_EVALUATION_MODEL_NAME`; when omitted, the notebook expects a deployment named `gpt-4.1`.
6. **Application Insights attached to the Foundry project for lab 04**. The project connection is preferred; `APPLICATIONINSIGHTS_CONNECTION_STRING` can be used as a fallback.
7. **Sufficient quota and budget** for model inference, evaluations, and red teaming. Labs 03 and 05 can make many model calls; start with the sample sizes provided.

AI Gateway is optional and is not required for notebooks 00–06. See the [AI Gateway - APIM](../AI%20Gateway%20-%20APIM/) labs for that separate setup.

### Local setup

Run these commands from the repository root:

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name ai-governance-labs --display-name "AI Governance Labs"
```

In VS Code, select the **AI Governance Labs** kernel for each notebook.

### Environment configuration

Copy the template, then populate the local file. `.env` is ignored by Git and must never be committed.

```powershell
Copy-Item .env.example .env
```

Required for notebooks 00–06:

| Variable | Value |
|----------|-------|
| `FOUNDRY_PROJECT_ENDPOINT` | Full project endpoint, ending in `/api/projects/<project-name>` |
| `FOUNDRY_MODEL_NAME` | Candidate chat model deployment name |
| `AZURE_SUBSCRIPTION_ID` | Subscription containing the Foundry resources |
| `AZURE_RESOURCE_GROUP` | Resource group containing the Foundry resources |

Optional or lab-specific:

| Variable | Used by | Value |
|----------|---------|-------|
| `FOUNDRY_EVALUATION_MODEL_NAME` | 03 | Judge model deployment; defaults to `gpt-4.1` |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` | 04 | Fallback when the project cannot return its attached Application Insights connection |

---

## Azure access

Grant your user (or the service principal you're using) at least:

| Role | Scope | Why |
|------|-------|-----|
| `Azure AI User` | Foundry project/resource | Invoke models and use project data-plane operations |
| `Azure AI Project Manager` | Foundry project | Create agents and manage project assets used by labs 01–05 |
| `Cognitive Services Contributor` | Foundry resource | Create and update the Content Safety blocklist in lab 02 |
| `Reader` | Subscription or resource group | Inventory Foundry resources, deployments, and quota in labs 01 and 06 |
| `Cost Management Reader` | Subscription | Query subscription cost data in lab 06 |
| `Log Analytics Reader` | Application Insights workspace | Inspect traces produced by lab 04 |

Role assignment can take several minutes to propagate. Organization-specific custom roles are also valid when they grant the same actions with narrower scope.

More detail: [Foundry Control Plane RBAC](https://learn.microsoft.com/en-us/azure/foundry/control-plane/govern-agent-infrastructure-entra-admin).

---

## Running the labs

From the repo root, with the venv activated:

```powershell
jupyter lab
```

Open notebooks in order. Each notebook is self-contained — you can rerun a single lab without rerunning the previous ones (but 00 does the setup checks).

Important execution notes:

- Run each notebook from top to bottom and stop on the first error. Restart its kernel before rerunning tracing or authentication setup.
- Notebook 01 creates sample prompt-agent versions when they do not already exist.
- Notebook 02 creates or updates a Content Safety blocklist.
- Notebook 03 runs paid model and safety evaluations and writes local result files.
- Notebook 04 sends model calls and telemetry to Application Insights; exported traces can take a few minutes to appear.
- Notebook 05 runs 60 attacks with the supplied settings and can take several minutes.
- Notebook 06 reads quota and cost data. Azure Cost Management data may lag by 8–24 hours.
- Generated evaluation and red-team result files can contain model prompts and responses. Review them before sharing or committing them.

## Where to look in the Foundry portal

Many Control Plane panes are portal-only today. Notebooks will point you at the exact portal path — for example:

> **Foundry portal → `Operate` (top-right) → `Compliance`**

for guardrail policy management. Look for the **Operate** button in the upper-right of the Foundry workspace at [ai.azure.com](https://ai.azure.com).
