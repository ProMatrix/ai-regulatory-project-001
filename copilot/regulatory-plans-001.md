# Regulatory Plans 001

## Objective

Create a completely fresh Microsoft Foundry implementation in this Visual Studio Code workspace that can answer questions about two PDF documents using a knowledge-base-first design backed by Azure AI Search.

The implementation must:

- keep the current knowledge-base-first design
- use `gpt-4.1-mini`
- use the two PDFs as the initial knowledge content
- use TypeScript for development and support scripts
- use `npm` and Node.js commands instead of PowerShell scripts
- prefer Azure AI Search API key authentication everywhere on the Search side, including the Foundry Search project connection when a Search connection is needed
- provide fully explicit setup instructions for a brand new VS Code workspace
- include local validation with Vitest and a small live validation against the deployed agent

## Confirmed Decisions

1. Architecture stays knowledge-base-first.
2. Model stays `gpt-4.1-mini`.
3. The two PDFs remain the initial knowledge content.
4. This is a completely fresh implementation, not a refactor of the current repo.
5. Use TypeScript source files executed with `tsx` and orchestrated through `npm` scripts.
6. Use the cleanest knowledge-source shape for this scenario.
7. Use API-key-only Search authentication as the policy goal.
8. Provide recommended names for all major resources up front.
9. Write instructions in a fully explicit, click-by-click style for a clean VS Code workspace.
10. This file is the full source of truth for assumptions, naming, prerequisites, setup flow, validation, risks, and rename guidance.

## Recommended Implementation Approach

### Language and Script Model

Use TypeScript for all local support scripts and package the workflow behind `npm` scripts.

Recommended local execution model:

- TypeScript source under `src/`
- TypeScript support scripts under `scripts/`
- `tsx` for running `.ts` files directly during development
- `vitest` for local test coverage
- `npm run ...` as the only documented command surface

This is cleaner than raw Node.js `.js` scripts because it keeps all automation typed, testable, and consistent.

### Cleanest Knowledge Source Shape

For this scenario, the cleanest knowledge-source design is:

1. Put both PDFs into one Azure AI Search index.
2. Preserve each PDF as a distinct source through metadata fields such as document name, page number, and chunk number.
3. Create one Search index knowledge source that wraps that index.
4. Create one knowledge base that references that one knowledge source.
5. Ground one Foundry agent through the knowledge base.

This is preferred over creating one knowledge source per PDF because:

- it keeps the first implementation simpler
- it still preserves document-level citations
- it supports cross-document answers
- it avoids unnecessary routing complexity

Use separate knowledge sources later only if the PDFs need independent lifecycle management, permissions, or ingestion schedules.

### Retrieval Guidance

Your requirement mentions semantic search with a vector query type. In a knowledge-base-first design, the cleanest approach is to configure the underlying Search index to support both semantic ranking and vector-ready retrieval, then let the knowledge base own the retrieval orchestration.

Plan recommendation:

- the Search index must include searchable text fields
- the Search index must include an embedding vector field
- the Search index must include a semantic configuration
- the knowledge base should be treated as the stable retrieval surface

Important constraint:

- in the knowledge-base-first pattern, do not over-couple the plan to a direct agent tool `query_type` setting such as `vector_semantic_hybrid`
- if the knowledge-base runtime chooses the effective retrieval path internally, prefer knowledge-base compatibility over forcing a tool-specific query mode name into the design

## Assumptions

This plan assumes the following unless changed later:

1. Azure region: `westus`
2. The new workspace starts empty except for the two PDF files and the implementation files you create.
3. The PDFs do not require per-user authorization or document-level ACL trimming.
4. Strict citations are required in all substantive grounded answers.
5. Foundry Agent Playground is the primary manual validation surface.
6. The solution is initially single-environment, but naming should scale cleanly to future `dev`, `test`, and `prod` variants.
7. Search authentication should use API keys where Search credentials are needed.
8. A fully clean API-key-only story might still require tradeoff notes if Microsoft Foundry knowledge-base integration prefers or requires managed identity at some connection layer in future product changes.

## Recommended Names

### Naming Strategy

Optimize names for future environment expansion, not just one-off readability.

Pattern:

- workload: `ai-regulatory`
- environment sequence: `001`
- region suffix when useful: `westus`
- resource type prefixes where Azure naming benefits from them

### Recommended Resource Names

Use these names for the fresh implementation.

1. Resource group: `rg-ai-regulatory-001-westus`
2. Foundry account: `aifnd-ai-regulatory-001`
3. Foundry project: `ai-regulatory-project-001`
4. Agent: `ai-regulatory-agent-001`
5. Azure AI Search service: `srch-ai-regulatory-001-westus-01`
6. Search index: `regulatory-pdf-index-001`
7. Search project connection name, if a direct Search connection is created for administrative or fallback use: `regulatory-search-conn-001`
8. Knowledge source: `regulatory-pdf-ks-001`
9. Knowledge base: `regulatory-pdf-kb-001`
10. MCP connection from Foundry project to knowledge base: `regulatory-pdf-kb-mcp-001`
11. Chat model deployment name: `gpt41mini-regulatory-001`
12. Embedding model deployment name: `text-embedding-3-small-regulatory-001`
13. App Insights name, if provisioned with the project: `appi-ai-regulatory-001`
14. Local workspace folder name recommendation: `ai-regulatory-project-001`

### Why These Names

- They are consistent.
- They expose workload and sequence clearly.
- They avoid using the exact model name as the deployment name, which makes future model replacement easier.
- They preserve room for future environments and additional iterations.

## Rename Guidance Summary

### Easy To Rename Later

- local folder names
- package names
- npm script names
- TypeScript file names
- agent instructions text

### Medium To Rename Later

- Foundry agent name
- Search index name
- knowledge source name
- knowledge base name
- project connection names
- model deployment names

These are usually renameable, but references in scripts, docs, config files, and validation steps must be updated together.

### Hard Or Expensive To Rename Later

- resource group name
- Azure AI Search service name
- Foundry account name
- project-wide environment naming conventions

These should be treated as effectively fixed once created because renaming usually means re-creating resources and updating all references.

## Fresh Workspace Plan

## Phase 1: Create the New Workspace

1. Create a new folder on disk named `ai-regulatory-project-001`.
2. Open Visual Studio Code.
3. Select `File`.
4. Select `Open Folder...`.
5. Open the new `ai-regulatory-project-001` folder.
6. Create these top-level folders:
  `documents`  
   `src`  
   `scripts`  
   `tests`  
   `foundry`  
   `docs`
7. Copy the two PDFs into `documents`.

## Phase 2: Initialize the Node.js Project

1. Open the VS Code terminal.
2. Run `npm init -y`.
3. Run `npm install axios dotenv zod`.
4. Run `npm install -D typescript tsx vitest @types/node`.
5. Run `npx tsc --init`.
6. Update `package.json` to add typed support scripts and test commands.
7. Create a `.env.example` file that lists all required environment variables.

Recommended `npm` script surface:

- `npm run check:env`
- `npm run search:create-index`
- `npm run search:index-pdfs`
- `npm run search:create-kb`
- `npm run foundry:create-agent`
- `npm run validate:local`
- `npm run validate:live`
- `npm test`

## Phase 3: Provision Azure Resources

Because you asked for fully explicit instructions, use the Azure portal and Microsoft Foundry portal for initial provisioning.

### Step 3.1: Create the Resource Group

1. Open the Azure portal.
2. Select `Resource groups`.
3. Select `Create`.
4. Choose the correct subscription.
5. Enter `rg-ai-regulatory-001-westus`.
6. Select region `West US`.
7. Select `Review + create`.
8. Select `Create`.

### Step 3.2: Create the Azure AI Search Service

1. In the Azure portal, search for `Azure AI Search`.
2. Select `Create`.
3. Choose the same subscription.
4. Choose the resource group `rg-ai-regulatory-001-westus`.
5. Enter the service name `srch-ai-regulatory-001-westus-01`.
6. Select region `West US`.
7. Choose a tier that supports semantic ranking and the knowledge-base features you need.
8. Select `Review + create`.
9. Select `Create`.
10. After deployment, open the Search resource.
11. Open `Keys`.
12. Copy an admin key for local indexing and configuration.
13. Open the Search overview page and copy the service endpoint.
14. Confirm semantic search is available or enable the appropriate semantic option supported by the service tier.

### Step 3.3: Create the Foundry Account and Project

1. Open Microsoft Foundry.
2. Create or open a Foundry account named `aifnd-ai-regulatory-001` if the experience asks for an account-level resource.
3. Create a project named `ai-regulatory-project-001`.
4. Place it in `rg-ai-regulatory-001-westus`.
5. Keep the region aligned with the Search service if the product allows that alignment directly.
6. Copy the Foundry project endpoint after creation.

### Step 3.4: Create Model Deployments

1. In the Foundry project, open model deployments.
2. Deploy `gpt-4.1-mini`.
3. Use deployment name `gpt41mini-regulatory-001`.
4. If local indexing needs embeddings, deploy `text-embedding-3-small`.
5. Use deployment name `text-embedding-3-small-regulatory-001`.

## Phase 4: Create the Search Index and Ingestion Pipeline

Implement the following in TypeScript support scripts.

### Script Responsibilities

1. `scripts/check-env.ts`
  validate required environment variables
2. `scripts/create-search-index.ts`
  create or update the Search index  
   include searchable text fields  
   include citation metadata fields  
   include a vector field  
   include a semantic configuration
3. `scripts/index-pdfs.ts`
  read the PDFs  
   extract text  
   chunk text  
   generate embeddings  
   upload documents to Azure AI Search
4. `scripts/create-knowledge-assets.ts`
  create the Search index knowledge source  
   create the knowledge base
5. `scripts/create-foundry-agent.ts`
  create or update the Foundry agent definition  
   bind it to the knowledge-base MCP connection if supported by the selected Foundry flow
6. `scripts/validate-live.ts`
  run one or more live prompts against the deployed agent

### Index Schema Requirements

At minimum, include fields like:

- `id`
- `title`
- `content`
- `source_file`
- `source_url`
- `page_number`
- `chunk_number`
- `content_vector`

This preserves document identity while still using one index.

## Phase 5: Create the Knowledge Source and Knowledge Base

### Recommended Shape

1. Create one Search index knowledge source named `regulatory-pdf-ks-001`.
2. Point it at `regulatory-pdf-index-001`.
3. Return citation-supporting metadata fields from the index.
4. Create one knowledge base named `regulatory-pdf-kb-001`.
5. Reference `regulatory-pdf-ks-001` from that knowledge base.
6. Add retrieval instructions that allow cross-document synthesis only when grounded evidence supports it.
7. Add answer instructions that require explicit citations and refusal of unsupported claims.

### Important Design Note

Because you selected API-key-only Search, document this tradeoff explicitly in the implementation:

- Search indexing and Search knowledge-asset creation should use Search API keys
- if the Foundry-side knowledge-base connection flow later requires managed identity for MCP access, keep the implementation plan knowledge-base-first but note that this is the first place where product behavior may conflict with the API-key-only preference

Do not hide that tradeoff in the plan.

## Phase 6: Create the Foundry Agent

1. Create a prompt agent named `ai-regulatory-agent-001`.
2. Use deployment `gpt41mini-regulatory-001`.
3. Ground it through the knowledge base rather than through direct Search tool wiring.
4. Use instructions that enforce:
  grounded answers only  
   Markdown output  
   explicit citations  
   refusal when support is insufficient  
   careful synthesis across the two documents only when supported

## Phase 7: Local Validation and Tests

### Vitest Coverage

Add simple local tests for:

1. environment validation logic
2. chunking behavior
3. document metadata preservation
4. naming/config generation
5. request payload generation for Search index creation
6. request payload generation for knowledge source and knowledge base creation

Keep these tests fast and local so they do not depend on Azure.

## Phase 8: Live Validation

Run at least one live validation script and one manual Playground validation pass.

### Required Live Checks

1. Ask a single-document question.
2. Ask a cross-document synthesis question.
3. Ask a citation-pressure question.
4. Ask an unsupported-answer question.

### Example Live Questions

1. Find one topic discussed in only one of the two PDFs and cite the source.
2. Summarize overlapping themes across both PDFs and cite each point.
3. What evidence in the documents supports the main regulatory claims? Cite every major claim.
4. What do the documents not establish about the topic? Do not speculate.

## Prerequisites

Before starting, make sure the fresh workspace has:

1. Node.js LTS installed.
2. `npm` available.
3. Visual Studio Code installed.
4. Azure subscription access.
5. Microsoft Foundry access.
6. Azure AI Search service creation rights.
7. The two PDF files available locally.
8. Azure CLI installed if you want optional CLI verification commands.

## Environment Variables

Document these in `.env.example`.

- `AZURE_SEARCH_ENDPOINT`
- `AZURE_SEARCH_ADMIN_KEY`
- `AZURE_FOUNDRY_PROJECT_ENDPOINT`
- `AZURE_OPENAI_CHAT_DEPLOYMENT`
- `AZURE_OPENAI_EMBEDDING_DEPLOYMENT`
- `AZURE_SEARCH_INDEX_NAME`
- `AZURE_KNOWLEDGE_SOURCE_NAME`
- `AZURE_KNOWLEDGE_BASE_NAME`
- `AZURE_FOUNDRY_AGENT_NAME`

## Known Risks and Tradeoffs

1. The knowledge-base-first architecture is the correct long-term design for extensibility, but API-key-only Search may not remain the cleanest supported integration path as Foundry evolves.
2. Search service names must be globally unique. Be prepared to add a suffix if the preferred name is unavailable.
3. `gpt-4.1-mini` and embedding deployments can hit quota or rate limits during testing and indexing.
4. Product behavior may differ between a direct Search tool connection and the newer knowledge-base MCP pattern.
5. If the product enforces managed identity on the Foundry-to-knowledge-base connection path, that will be a documented exception to the API-key-only preference rather than an implementation mistake.
6. If the knowledge-base runtime abstracts retrieval internally, the underlying index can still be vector-ready and semantic-enabled even if the exposed agent configuration does not use the exact tool-style `query_type` field name.

## Explicit Rename Guidance

If future rename requests happen, use this order of caution.

### Safest To Rename

1. local TypeScript script names
2. npm command names
3. docs and markdown files
4. instruction text

### Rename With Coordinated Updates

1. agent name
2. deployment names
3. index name
4. knowledge source name
5. knowledge base name
6. connection names

When renaming these, update:

- `.env.example`
- local config files
- TypeScript constants
- tests
- validation scripts
- documentation

### Prefer Not To Rename After Creation

1. resource group
2. Search service
3. Foundry account

Treat those as stable identifiers.

## Final Recommendation

Proceed with a fresh TypeScript and `npm`-driven implementation using:

- project `ai-regulatory-project-001`
- agent `ai-regulatory-agent-001`
- model `gpt-4.1-mini`
- deployment name `gpt41mini-regulatory-001`
- one shared Search index with strong source metadata
- one Search index knowledge source
- one knowledge base
- strict citations
- Vitest plus one live validation script

This is the cleanest path that satisfies your confirmed requirements while keeping the system extensible for more knowledge sources later.