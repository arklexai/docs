# Docs vs code audit

> **Status (2026-10-08): applied.** All page changes below are in the working tree. Decisions:
>
> - No links to the platform website. The docs.json Sign in button and the signup link are removed.
> - "Arklex Platform" is the official name.
> - Metric Alignment is hidden. The page moved to `drafts/metric-alignment.mdx` with its old content; fix it per section 4 before re-publishing.
> - SSO/SAML is removed from the docs.
> - Providers are Anthropic, OpenAI, and Gemini.
>
> Still open:
>
> - YouTube support (decision 6): left out of knowledge.mdx.
> - Every screenshot and Supademo embed needs a retake. The scenario generation embed was removed.
> - The product bugs in section 6. In addition, the Billing plan card in the app still lists "SSO / SAML" (`FE pages/SettingsPage/BillingTab/constants.js:49`), and the model catalog on `main` has no Anthropic models (`PY arkdock_python/llms/catalog.py`).

Date: 2026-10-08. Compared every page in this repo against `origin/main` of:

| Repo | Commit | Notes |
| --- | --- | --- |
| frontend (`apps/arkdock-web`) | `ea82647d` | Staging runs `prerelease-arkdock-web-0.1` (v0.1.41), which is main minus #1234 and #1251 |
| backend (arkdock-api) | `fde1c2c5` | Staging is main minus #1131 and #1143 |
| chatbots (`arkdock-python`) | `7de158e2` | Staging is main minus #986 |

**Release facts.** No `release-arkdock-*` branch or tag exists yet. `https://arklex.ai/platform/` returns 503 and `arkdock.arklex.ai` is NXDOMAIN. Staging (`stg.arklex.ai/platform/`) is live.

Prod Helm sets `FEATURE_MEM_ALIGN=false` (Metric Alignment hidden) and `FEATURE_BILLING=true`. Staging has both on.

**Main-only (not on staging or prod):** MCP tools as a scenario source (frontend #1234, backend #1131, chatbots #986) and the MCP error-message rewrite. Don't document these yet.

**Path conventions.** Code paths are relative to each repo:

- `FE` = `frontend/apps/arkdock-web/src`
- `BE` = `backend`
- `PY` = `chatbots/arkdock-python`

Tags: **ADD** (in the product, not in the docs), **UPDATE** (docs are wrong), **DELETE** (docs describe something that no longer exists), **VERIFY** (needs a product or ops answer).

---

## 1. Decisions needed before editing

1. **Production URL.**
   - `docs.json` "Sign in" points to `arkdock.arklex.ai/platform/login`, which is NXDOMAIN.
   - `quickstart.mdx` links to `arklex.ai/signup`, which is 404. The app route is `/platform/signup`.
   - Prod `arklex.ai/platform/` returns 503 today.
   - Which host is canonical?
2. **Product name.**
   - The UI says "Arklex Platform" everywhere: `<title>`, sidebar, onboarding, and login copy.
   - The docs mix "Arklex", "Arklex Platform" and "Arkdock" (support.mdx, index.mdx, README).
   - Recommend "Arklex Platform" on first mention and "Arklex" after.
3. **Metric Alignment** is off in prod. Keep the page with an availability Note (the AGENTS.md rule), or hide it from the nav until it ships?
4. **SSO / SAML** is listed as an Enterprise extra (settings.mdx and the BillingTab plan card). The only SAML in code is the internal `/admin` console login. Is it sold contractually?
5. **LLM providers in prod.** The model pickers only show providers with credentials:
   - OpenAI: GPT-6 Sol (default), GPT-6 Luna, GPT-5.6 Terra/Sol/Luna.
   - Google: Gemini 3.x and 2.5 models.
   - No Anthropic.

   Confirm which providers prod has, or keep the docs model-agnostic.
6. **YouTube links.** The UI says "Supports individual web pages and YouTube videos". The crawler only renders visible page text in headless Chrome and fetches no transcript. Keep the claim?

---

## 2. Site-wide

- **UPDATE: style.** There are 163 em and en dashes across 13 pages, which the AGENTS.md rules forbid. Counts: scenarios 22, evaluations 19, metrics 17, agents 16, annotations 15, quickstart 14, settings 14, knowledge 11, metric-alignment 11, simulations 11, faq 7, overview 5, index 1.
- **UPDATE: "admin" should be "Owner"** at annotations.mdx:6, faq.mdx:58 and quickstart.mdx:17. Only the `owner` and `member` roles exist (`BE base/auth/arkdock_user.go:53`).
- **UPDATE: "persona" used for "scenario"** at quickstart.mdx:78,85,91, scenarios.mdx:69,99 and README.md:22. faq.mdx:30 and overview.mdx:59 should say "goal and user profile".
- **UPDATE: Title Case headings.** overview.mdx "Why Arklex Platform?" and "Getting Started"; support.mdx "Get Help" and "Connect With Us".
- **UPDATE: the sidebar changed.** It now reads Scenarios, Simulations, Evaluations | Agents, Knowledge, Metrics, MCP | API Reference, Settings (`FE components/PlatformLayout/Sidebar.jsx:47-84`). Every screenshot shows the old sidebar.
- **UPDATE: "seven built-in metrics"** in overview.mdx:67, faq.mdx:42 and metrics.mdx:6,12,102. See Metrics below.
- **DELETE: leftover files.**
  - `snippets/snippet-intro.mdx`: template filler, unused.
  - `CONTRIBUTING.md`: unedited Mintlify template that links a missing `development.mdx`.
  - `index.mdx`: not in the nav and branded "Arkdock". Replace it with a `docs.json` redirect.
- **UPDATE: README.md.** The title and link use `arkdock.arklex.ai`. The structure table omits faq, metric-alignment and support, and calls overview "Product overview and FAQ".
- **UPDATE: overview.mdx:23-25.** The "See Examples" card links to /agents, which has no examples. Retitle it "Connect an agent" or remove it.
- **UPDATE: LinkedIn mismatch.** support.mdx uses `/company/arklex`; the docs.json footer uses `/company/arklexai`. The two auditors got conflicting HTTP results, so check both by hand and use one.
- **UPDATE: support.mdx.** It says "Arkdock". The GitHub card says "Star the repository" but links to the org. Add a tip to include the UI and API versions shown in the account menu.

---

## 3. New pages to add

### `mcp.mdx`: MCP servers (sidebar **MCP**, shipped, no flag)
Source: `FE pages/McpPage/index.jsx`, `ToolTester.jsx`; `BE base/arkdock/mcp_server.go`, `mcp_tool_approval.go`.

- **Add a server.** **Add Server** opens **Add MCP Server** with these fields:
  - **Display name**.
  - **Transport**: fixed to Streamable HTTP. stdio is self-hosted only and is rejected today.
  - **Server URL**: must be public; private and loopback addresses are blocked.
  - **Auth header**: one name/value pair, defaulting to `Authorization`.

  Then **Test connection** and **Add and Connect**. Names must be unique per organization. There is no edit; to change the URL or credential, delete and re-add.
- **Server table.**
  - Columns: Server, Health (**Reachable** / **Error** / **Not checked**), Tools (enabled/total), Last checked, Enabled, and Actions (**Test**, Remove).
  - Summary tiles: **Reachable servers** and **Enabled tools**.
- **Details sheet.**
  - Tabs **Tools (n)** and **About**, plus **Re-run tools/list**.
  - Per tool: Enabled switch; safety label **Read-only** / **Write-capable** / **Unknown safety**; approval switch **Judge may call** (main renames it).
  - Only Read-only tools can be approved. Approval is revoked automatically when a tool's schema changes.
- **Test a tool.** **Test this tool**, then the arguments form or **Edit as JSON**, then **Run tool**.
- **Roles.** Owners only can run or approve tools. Any member can add, remove, enable and test servers.
- **Limits.** 10 s probe timeout, 30 s tool-call timeout, 500 tools cached.
- **Where it's used today.** Custom metrics can call approved tools (see Metrics). Evaluations then show **Evidence used for this score** and offer **Use saved / Fetch new tool results** on re-evaluate.
- **Do not document yet:**
  - Scenario grounding via MCP (main-only).
  - Simulated users calling MCP tools. The page subtitle claims it, but `PY service/tasks/simulation.py` never reads MCP servers.

### `api.mdx`: API reference and API keys (shipped)
Source: `FE pages/ApiDocsPage`, `pages/SettingsPage/ApiKeysTab.jsx`; `BE models/publicapi/v1/openapi.yaml`, `service/api/arkdock-api/main.go:563-631`, `base/arkdock/apikey.go`.

- **The in-app page.** **API Reference** in the sidebar renders the live OpenAPI spec at `{app origin}/api/v1/openapi.yaml`, with **Test Request**.
- **API keys.** Under **Settings > API keys**, Owners only:
  - **Create key** opens **Create an API key** (Name, max 100 characters).
  - The key is shown once, in **Copy your API key**.
  - The list shows Name, Key prefix, Created, and Last used.
  - The trash icon revokes the key. There's no expiry and no scopes.
  - Keys can't reach billing, members, or key management.
  - A key stops working if its creator leaves the organization or is removed.
  - Auth header: `Authorization: Bearer ark_...`.
- **Resources (29 operations).**
  - Connections (= Agents): CRUD, rotate-credentials, and test.
  - Simulations, including inline scenarios and `trace_capture`.
  - Evaluations, including inline transcripts. Agent perspective only.
  - Scenario groups and scenarios.
  - Lookups for metrics and models.
- **Conventions.**
  - Error envelope `{"error":{"code","message"}}`.
  - `Idempotency-Key` (24 h).
  - 402 `QUOTA_EXCEEDED` and 429 `RATE_LIMITED` / `CONCURRENCY_LIMIT_REACHED` with `Retry-After`.
  - Default limit of 500 requests per 60 s per key; prod values weren't verified.
- **Base URL.** It is the app origin. Don't cite `api.arklex.ai`, which is NXDOMAIN.

### `tool-tracing.mdx`: Capture tool calls (shipped)
Source: `FE pages/AgentDetailPage/TraceSetupCard.jsx`, `pages/SimulationsPage/NewSimulationSheet.jsx:777-845`, `pages/SimulationDetailPage/ToolCallList.jsx`; `PY tests/fixtures/otel/README.md`.

- **Setup.**
  - The agent's **Configuration** tab has a **Tool call tracing** card with the OTLP/HTTP endpoint `{origin}/api/arkdock/otlp/v1/traces`, authenticated with an organization API key.
  - Your agent must continue the W3C `traceparent`, or copy the baggage entry `arklex.capture.id` onto spans.
  - Traces from non-simulation traffic are discarded.
  - Tested formats: Google ADK, OpenInference (LangChain/LangGraph, OpenAI Agents SDK), LangSmith OTel, and OTel GenAI conventions.
- **Per run.** Turn on **Capture tool calls** in New Simulation. **Advanced** has **Trace wait time (seconds)**: 10-300, default 60.
- **Results.**
  - The simulation **Tool calls** column shows **Collecting tool calls**, **Tool calls captured**, **Partially captured**, or **No tool calls captured**.
  - Transcripts list "N tool calls" with arguments, result or error, and duration.
- **Related features.**
  - The **Tool call behavior** built-in metric.
  - The **Tool calls** assertion type (Must call / Must never call).
  - Evaluations wait up to 5 minutes for trace collection.
- **Limits.** 4 MiB per request, 16 MiB decompressed, 2,000 spans per request, and 600 requests and 64 MiB per minute per organization.

Suggested nav: a new **Developers** group with `api` and `tool-tracing`, and `mcp` under Core Features after `metrics`.

---

## 4. Per page

### overview.mdx
- **UPDATE :55.** Agents connect via Chat Completions, A2A **or Custom HTTP**.
- **UPDATE :67.** Replace "Seven built-in metrics" with "Built-in metrics". Optionally add "plus three for scoring the simulated user".
- **UPDATE :71.** The tab is **Annotation Calibration**. It shows per-metric agreement, Need Attention and Coverage. It does not "surface common disagreements" or link to disputed turns (`FE pages/EvaluationDetailPage/CalibrationTab.jsx`; `topDisagreement` is fetched but never rendered).
- **ADD.** Mention assertions: scenario checks decide Pass or Fail, and metrics add scores. This is now the core evaluation model.

### quickstart.mdx
- **UPDATE :9-13 (onboarding).** New users first see the required **Welcome to Arklex Platform!** profile dialog (Company, Occupation, optional referral, **Continue**). An optional **Start walkthrough** tour follows (6 steps), then the **Onboarding Checklist 0/5**.
  - Steps are checked by these actions: creating an agent, *exiting a generation session* (import doesn't count), creating a simulation, creating an evaluation, and creating a custom metric.
  - Progress is stored per browser in localStorage.
  - Source: `FE components/OnboardingDialog`, `WelcomeInterstitial`, `tours/checklistItems.js`.
- **UPDATE :17.** Fix the signup URL (decision 1). Change "ask an admin" to "ask an Owner of your organization to invite you". Invites expire after 7 days.
- **UPDATE :18.** Add Custom HTTP. Add that the endpoint must be publicly reachable.
- **UPDATE Step 1.**
  - The API type tabs are **Chat Completions** / **Agent to Agent (A2A)** / **Custom HTTP**.
  - For A2A, enter the base URL that serves `/.well-known/agent-card.json`.
  - For A2A, delete the pre-filled body rows, or the save fails.
  - There's no "green **Connected** status". The success signal is **Agent responded successfully** from Test Connection, after which the agent's detail page opens.
- **UPDATE Step 2 (rewrite).**
  - Knowledge is optional.
  - There's no **Generate Scenarios** button. The flow is **Create group**, then **Add Scenarios**, then **Build with Assistant**.
  - Then: describe the agent, confirm goals and counts, choose knowledge, describe customers, click **Generate N**, review drafts, and click **Save N selected**.
  - Success line: scenarios appear in the group's **Scenarios** tab.
- **DELETE :89-93.** Remove the duplicate "You'll know it worked" line, the extra `---`, and the `PASTE_SUPADEMO_EMBED` comment.
- **UPDATE Step 3.**
  - Conversations per scenario: default 1, **max 10**. The run total is capped at 2,000 (not 50).
  - Add **Max concurrent conversations** (default 10, max 50).
  - After **Run Simulation** the sheet closes and the run appears in the list; it doesn't open the detail page.
  - **Completed** can still include failed conversations.
- **UPDATE Step 4.**
  - Fields, in order: **Evaluate** (Agent), **Evaluation Name**, **Simulation** (one), **Metrics**, **Model**.
  - "Choose a judge" with GPT-4o, Claude 3.5 Sonnet and Gemini 2.0 Flash is wrong: the field is **Model**, and none of those models are offered.
  - "Goal Completion is always included" is wrong. Helpfulness, Coherence and Relevance are preselected.
  - Results: the "four sections" now lead with **Conversations passed**. There are no "suggested fixes". Conversations show **Result** and **Assertions**. Reasoning is an info icon on qualitative labels under the **metrics** link.
- **VERIFY.** All four Supademo embeds predate Custom HTTP, the assistant flow, the new simulation fields and assertions. Re-record them.

### faq.mdx
- **UPDATE :13-14.** Keep "No code changes", but add that the endpoint must be public and that tool capture needs OTel export.
- **UPDATE :17-19.** Add Custom HTTP.
- **UPDATE :26.** An evaluation scores **one** simulation, using the judge and the scenarios' assertions (`BE controllers/arkdock/evaluation.go:81`, a single `simulation_id`).
- **UPDATE :30.** Change "persona and goal" to "the scenario's goal and user profile".
- **UPDATE :42, :46.** Drop "seven". Custom metrics return 1-5, 0-1, or a label you define.
- **UPDATE :50.** Use the tab names **Annotations** and **Annotation Calibration**, and drop "surfaces common disagreements".
- **UPDATE :58.** Change "Admins can toggle" to "Owners automatically see every reviewer's scores and the majority. Members see only their own until a row is resolved." There is no toggle.
- **ADD :61-63.** Only header values are encrypted. Keep secrets out of Parameters, Messages and Custom HTTP templates.

### agents.mdx
Source: `FE pages/AgentsPage/AddAgentForm.jsx`, `pages/AgentDetailPage/ConfigurationTab.jsx`, `TestConnectionResult.jsx`; `BE base/arkdock/agent.go`.

- **ADD: Custom HTTP API type.**
  - **Request Body Template** must contain `{{message}}`. `{{chat_id}}` and `{{timestamp}}` are available through the **Insert variable** chips.
  - **Import template** has an "OpenAI (chat completions)" preset.
  - **Response Path** is required (for example `choices.0.message.content`).
  - **Additional Fields** has Path / Label / Format columns.
  - A placeholder must be the whole string value.
  - Only the current message is sent each turn, so stateful agents key on `{{chat_id}}`. Responses must not stream.
- **UPDATE :32 "Integration method".** The field is **Method**: **API Endpoint**, or **Arklex Agent (coming soon)**, which is disabled.
- **UPDATE :43 localhost.** Private, loopback and link-local endpoints are rejected (`agent.go:645`; prod doesn't override this). Suggest a public tunnel.
- **ADD: A2A endpoint.** Enter the base URL; the agent card is read from `/.well-known/agent-card.json`. A body is rejected for A2A ("body is not supported for a2a agents"), but the form pre-fills `model`. Tell users to delete those rows.
- **UPDATE :67 response contract.**
  - Each turn sends the configured messages plus the full conversation.
  - The reply can be in OpenAI `choices[0].message.content`, Anthropic `content[]` or Gemini `candidates[0]` format.
  - Defaults: `model`=`gpt-5.4` and the system message "You are a helpful assistant."
- **UPDATE :65.** The system row can't be deleted while its role is `system`, but the role dropdown can change it.
- **UPDATE :69-70.** The button is **Import JSON**; it replaces Parameters and Messages.
- **UPDATE :56-58.** The label is **Values stored encrypted** next to **Headers**, and there's an eye toggle. AES-256-GCM is correct.
- **UPDATE :77 Test Connection.**
  - Success shows "Agent responded successfully". The latency line never renders because the API returns no latency.
  - Failure shows the status hint and a **What the server replied** block.
  - Custom HTTP also shows **Reply the agent would give** and the response body.
  - The Chat Completions probe sends one user message, `test`. Any 2xx counts as success.
- **UPDATE :77.** There's no confirmation toast. **Connect Agent** opens the detail page on **Simulations**.
- **ADD :16 list page.** Header "Agent Manager", a card ⋮ menu with **Delete**, 9 agents per page, and the empty state "No agents yet". There is no search.
- **UPDATE :87-94 Simulations tab.**
  - Columns: **Simulation Name**, **Scenario Groups**, **Model**, **Last Run**, **Status**.
  - Add **Cancelled**, search, the filter (**Created by me**, Status), **New Simulation**, and row actions.
- **UPDATE :101 Evaluations tab.**
  - Columns: **Evaluation Name** (with an Agent or Simulated user badge), **Simulations Evaluated**, **Created By**, **Date**, **Status**. There's no Errors column.
  - Running rows can't be opened.
  - **New Evaluation** here lists only this agent's simulations.
- **UPDATE :110-111.** **Save** and **Discard** appear only after an edit, next to "You have unsaved changes".
- **UPDATE :112.** **Test Connection** uses the current form values, including unsaved edits. If you change the endpoint, re-enter the header values.
- **UPDATE :113.** Delete is a trash icon in the detail page header, or **Delete** in the card ⋮ menu. It's blocked while simulations use the agent. Delete their evaluations, then the simulations, then the agent.
- **UPDATE :115 and FAQ :136.** "Leave blank to keep" is wrong, and following it saves an empty value. Saved values load masked (`****`): leave the mask unchanged to keep the value, replace it to update. Renaming a key requires re-entering its value (`agent.go:427-444`).
- **ADD: Tool call tracing card** on the Configuration tab. Link it to `tool-tracing.mdx`.

### scenarios.mdx (largest rewrite)
Source: `FE pages/ScenariosPage/AddScenariosPicker.jsx`, `ScenarioSession/*`, `ImportScenariosWizard.jsx`, `pages/ScenarioGroupPage/components/GroupDetailView.jsx`, `pages/ScenarioDetailPage/*`; `PY scenario_builder/config.py`, `service/session_*.py`.

- **DELETE :26-81.** Remove the four-mode table, the **Create a Scenario** and **Generate at Scale** wizards, the "up to 20", "10 / 100 knowledge items" and "knowledge required" rules, and the Demographics/Business/Psychographics tabs. **Add Scenarios** now opens "How do you want to add scenarios?" with three choices: **Build with Assistant**, **From Conversation History** and **Import a File**.
- **ADD: Build with Assistant.** This is a chat session ("Build scenarios") with **Plan** and **Scenarios (N)** tabs. The steps:
  1. Describe the agent ("Describe what you need, or paste a link"; a pasted link is read and attached).
  2. Confirm goals and per-goal counts on question cards (**Send answers**).
  3. Choose knowledge: **My whole knowledge base**, **Choose documents**, **Pasted records** (min 40 characters) or **Nothing**. With nothing attached, a warning offers **Choose knowledge**, and the button becomes "Generate N anyway".
  4. Describe customers: **Describe your customer segments** (or **Suggest some for me**), or **None**. Then add attributes under "Anything to vary between them?"
  5. Click **Generate N**. There are **at most 100 per run**; click again for more, or use **Change the split**. **Stop** cancels the run.
  6. Review drafts (they arrive selected):
     - Card actions: Edit, Regenerate, Discard.
     - Grounding labels: Grounded, No supporting knowledge found, Generated without grounding, A source could not be read.
     - Check findings include Duplicate, Contradiction, Outcome given away and others.
     - **Re-ground them** appears for drafts without supporting knowledge.
     - Then **Save N selected**. A flagged selection needs a second save.
- **ADD: Plan tab.** Use it to edit goals and counts (**+ Add goal**, "N/100 scenarios"), **Customer segments (optional)** and **Persona attributes (optional)**, then **Save plan**.
- **ADD: chat requests.** You can ask in chat to revise specific drafts, or to review the group's saved scenarios.
- **ADD: model.** The model picker is in the header and is locked after the first message. Use **Start over** to switch; saved scenarios are kept.
- **ADD: leave and come back.** Sessions save automatically and keep running in the background. Resume one from the group's **Drafts** tab or **History**. Sessions are private to their creator.
- **ADD: Note.** Building uses the plan allowance. When it's used up, the session shows **Go to billing**.
- **UPDATE :83-106 From Conversation History.**
  - There's no flag and no separate wizard. It's the same assistant session, starting with an upload (`.json` or `.jsonl`, max 256 MB).
  - Format: a JSON list of conversations, or one per JSONL line, each with `messages` containing `role` (user, assistant or bot) and `content`.
  - The assistant proposes goals, counts, segments and attributes, then asks you to choose **Use it as it is** or **Adjust the counts**.
- **UPDATE :108-110 Import a File.** There is no field mapping.
  - Steps: **Upload** / **Edit** / **Review**. Accepts `.csv` and `.json` up to 10 MB; **Download template** gives a JSON file.
  - Required fields: Goal, User Profile and Agent Context. Optional: name, checklist, and up to 5 knowledge items.
  - CSV columns: `name`, `goal`, `user_profile`, `agent_context`; `knowledge_name`/`knowledge_content` with suffixes `_2` to `_5`; `checklist_*` with suffixes `_2` to `_4`.
  - Assertions can only be imported in JSON, and they're read-only in the wizard.
  - The button reads **Import N Scenarios**, and invalid rows are skipped.
- **UPDATE :20 group creation.** **Create group** asks for a Name only, defaulting to "Untitled group". There's no description field.
- **ADD: group management.**
  - Cards show the name, date and creator, with ⋮ **Delete**.
  - The group page has rename (pencil icon), **Download all** (JSON) and **Add Scenarios**.
  - The scenarios table has User Goal, Knowledge and a trash action.
- **UPDATE :116-154 Drafts.** The tab lists your unsaved assistant sessions; the empty state reads "Nothing in progress". Old wizard runs appear only under "Unfinished runs from earlier generation methods", and they can't be continued. Delete the status table and the **Continue** action.
- **UPDATE :152-154 group delete rule.** A group can't be deleted while any of its scenarios is used by a simulation. Deleting a group also deletes its sessions.
- **UPDATE :156-164 anatomy.** A scenario has:
  - User Profile, with segment and attributes.
  - User Goal.
  - **User Goal Checklist** (Required/Optional items, up to 4). It ends the simulated conversation but doesn't grade the agent.
  - **Assertions**.
  - **Agent Context**.
  - **Knowledge**: up to 5 passages *copied into* the scenario, with no link back to the library.
- **ADD: Assertions** (shipped).
  - Types: **LLM judge**; **Text match** (Any reply contains / No reply contains / Any reply matches); **Tool calls** (Must call / Must never call, order and argument matching, needs trace capture).
  - Limits: 50 per scenario, one tool-calls assertion, judge text up to 1000 characters.
  - "A conversation passes only if every assertion passes." Changes apply to new simulations.
- **UPDATE :166-168 tip.** Knowledge also shapes the generated checklist and assertions.
- **UPDATE :176 editing.** Click **Edit**, then a single **Save** (not per-field). Saving may prompt "Update the generated fields too?" (**Save without regenerating** / **Save and regenerate**). After a profile change, **Sync attributes** appears. The name isn't editable.
- **UPDATE versions.** Use the **Version** dropdown; historical versions are read-only. A save with no changes creates no version.
- **UPDATE :178 delete.** There's no delete on the detail page. Use the trash icon in the group table. It's blocked if any simulation used the scenario.
- **UPDATE :184-191 selector.** Groups start expanded. Rows and search use the goal text. The preview shows name, goal, profile, agent context and knowledge. The buttons are **Select All**/**Deselect All** and **Select Group**/**Deselect Group**. **Created by me** doesn't exist here, so remove it.
- **UPDATE FAQ :198-224.** Change "up to 20" to "up to 100 per **Generate**". Rewrite the half-finished answer around sessions.
- **DO NOT ADD YET.** MCP "Connected tools" as a source is main-only.

### simulations.mdx
Source: `FE pages/SimulationsPage/NewSimulationSheet.jsx`, `simulationLimits.js`, `RerunSheet.jsx`, `components/SimulationTable`, `pages/SimulationDetailPage`; `BE base/arkdock/simulation.go`; `PY service/tasks/simulation.py`.

- **UPDATE :16 list columns.** Simulation Name, Agent, Scenario Groups, Model, Created By, Last Run (start date and time), Status.
- **UPDATE :18-24 statuses.**
  - **Completed**: every conversation ended and at least one succeeded.
  - **Failed**: none succeeded, or the run couldn't start.
  - **Pending** is brief.
- **UPDATE :26.**
  - Search matches the simulation name only.
  - The filter is Agent, **Created by me**, and Status (Completed / Running / Failed only).
- **UPDATE :35.** The sheet shows "No agents available..." and "No scenarios available..." messages, with no links.
- **UPDATE :54-56.** Conversations per scenario: max **10**. The total is capped at **2,000**, and the sheet shows "Will generate N conversations across M scenarios."
- **ADD: new fields.**
  - **Max concurrent conversations**: default 10, max 50, remembered per browser.
  - **Capture tool calls**, with **Trace wait time (seconds)** under **Advanced**.
- **UPDATE :62-64 Model.**
  - Pre-filled with the default model; a **Provider** select appears if more than one provider is configured.
  - The model plays the simulated user and judges when each turn should end.
- **UPDATE :71.** After **Run Simulation** the sheet closes ("Simulation created successfully."), and the run appears in the list.
- **UPDATE :78 header.** Add the model, **Download all** (JSON), and "Collecting tool calls for X of Y conversations".
- **DELETE :94.** There's no **Summary** column. The columns are Scenario Name, Goal, Turns, Status and **Tool calls**.
- **UPDATE :97 conversation statuses.** Done, Running, Pending (waiting for a slot), Failed (hover for the error) and Cancelled.
- **UPDATE :104 transcript modal.**
  - Unlabeled arrow buttons and a "Conversation N of M" header.
  - A **Download** menu (TXT, JSON).
  - A **Scenario** panel with goal, profile, checklist and knowledge.
  - A failure banner, and expandable tool calls.
  - A shareable `?conv=` link.
- **ADD: early endings.** A conversation can end before max turns when the checklist is complete, the user gives up, or the agent refuses or hands off.
- **ADD :112-116 actions.**
  - **Cancel** (Pending only; a running simulation can't be stopped).
  - **Delete** is blocked while Pending or Running, or while any evaluation uses the run.
  - **Rerun** keeps every setting and only renames the run. Scenarios run at their *latest* version.
- **UPDATE FAQ :132.** Failed conversations keep their partial transcript. Evaluations show them as **Agent error during simulation**, unscored.
- **UPDATE FAQ :136.** The Free plan allows 2 simulations pending or running at once.
- **DELETE FAQ :139-141.** "Completion rate" isn't shown anywhere in the UI.

### evaluations.mdx
Source: `FE pages/EvaluationsPage/NewEvaluationSheet.jsx`, `ReevaluateSheet.jsx`, `components/EvaluationTable`, `pages/EvaluationDetailPage/*`; `BE base/arkdock/evaluation.go`; `PY service/metrics/registry.py`, `service/simuser_eval`.

- **UPDATE :6.** An evaluation scores one simulation. Annotation applies to agent evaluations.
- **UPDATE :22 list.**
  - Columns: Evaluation Name (with an **Agent**/**Simulated user** badge), Simulations Evaluated, Created By, Date, Status. There's no Errors column.
  - Search matches the evaluation name only.
  - Running rows can't be opened.
- **DELETE :28.** There's no **New Evaluation** in the simulation ⋮ menu. The second entry point is an agent's **Evaluations** tab.
- **UPDATE :38-40 Simulation.** You pick **one** simulation from a single-select list, and it must be completed. If the simulation captured tool calls, add a Note: the evaluation can wait up to 5 minutes for collection.
- **UPDATE :43 and FAQ :138-140.** Goal Completion is no longer forced. The metric is renamed **User Goal Completion** and is optional. Helpfulness, Coherence and Relevance are preselected. "Each scenario's assertions always run and decide its Pass or Fail."
- **ADD: fields.**
  - **Evaluate** (Agent / Simulated user) comes first.
  - **Model** is last: a judge model with a default; **Provider** appears only if more than one provider is configured.
- **UPDATE :54-66 scales and bands.**
  - Built-ins use 1-5, except User Goal Completion (0-1). Custom metrics use 1-5 or 0-1.
  - There's no final score anymore (the v2 result shape removed it).
  - Bands apply to every quantitative metric. Colors are green, blue, amber and red.
- **UPDATE :70 header.**
  - Stats: conversations, avg turns, **judge model**.
  - Badge "Evaluating: Agent".
  - **Re-evaluate** button, and **Cancel Evaluation** while running.
  - Annotations and Annotation Calibration appear only on agent evaluations.
- **ADD Quantitative Metrics.**
  - A **Conversations passed** card (N / total, failed, inconclusive, not evaluated, errored; a "Did not pass" list).
  - Mean and median per metric.
  - An **Expected tool calls** card.
- **UPDATE :82.** Qualitative examples: use **Agent Behavior Failure** and **Tool call behavior**, not sentiment or tone.
- **UPDATE :90 Unique Errors.** One sorted list, with failed assertions first and then by severity. High is a lighter red and Low is blue. There's no suggested fix. Judge-flagged errors require **Agent Behavior Failure** to be selected.
- **UPDATE :100-107 Conversations table.**
  - Columns: Conv #, **Result** (Pass / Fail / Inconclusive / Not evaluated / Error), Simulation Evaluated, Scenario Name, **Assertions** (passed/total), and User goal completion (only if that metric was selected).
  - Goal, Final Score and Done/Running/Failed no longer exist.
- **UPDATE :109 modal.**
  - Arrow buttons, plus **Comments** and **Download** (.txt).
  - A per-turn **metrics** link.
  - Reasoning is an info icon on qualitative labels.
  - **Metrics**, **Assertions** (with **Show calls**) and **Scenario** blocks.
  - Tool calls, and **Evidence used for this score** for MCP metrics.
- **UPDATE :131.** "Rerun" is now **Re-evaluate** (menu and header). It offers **Use saved tool results** or **Fetch new tool results** (MCP only), pins the original metric versions, and creates a new evaluation.
- **ADD: Cancel or delete.** **Cancel** works while running. **Delete** is disabled while running.
- **ADD: section "Evaluate the simulated user".**
  - Metrics are fixed: Character Consistency, Goal Adherence and Realism.
  - The detail page has **Overview** ("Scores" / "Problems found") and **Conversations** (In character, Off-goal turns, Required sub-goals, Issues).
  - There's no annotation for these evaluations.

### annotations.mdx
Source: `FE pages/EvaluationDetailPage/AnnotationsTab.jsx`, `ConversationModal/*`, `CalibrationTab.jsx`; `BE base/arkdock/annotation.go`.

- **UPDATE :6.** Owners (not "admins") resolve disagreements. Past values are overwritten (`ON DUPLICATE KEY UPDATE`), so drop the claim that full history is kept for audit.
- **UPDATE :9,16.** The tabs exist only on agent evaluations (`metric_capability.go:59`).
- **UPDATE :18.** There's no disagreement styling. **Majority** shows for Owners only, and only when more than 50% agree. Resolved rows turn green; edited rows show **unsaved** or **incomplete**.
- **UPDATE :23-24.** Owners see two reviewers per cell plus **+N more**.
- **UPDATE :31.** Members see only their own scores in the app. However, **Export CSV** is open to every role and contains all reviewers' scores. Note this, or treat it as a product bug.
- **UPDATE :56-60 inputs.**
  - Buttons appear for any whole-number range up to 10 steps.
  - 0-1 uses a one-decimal text box.
  - Other ranges use a step-0.1 number input.
  - Qualitative metrics without labels fall back to free text.
  - "Auto: X.X" sits *below* the input in the spreadsheet.
- **UPDATE :62 saving.** **Save Annotation** (bottom right) saves all edited rows in the current scope. Every metric in a row must be filled. Drafts persist in the browser tab.
- **UPDATE :69-85 resolution.**
  - Strict majority means more than 50% identical values.
  - In the spreadsheet the button is **Resolve**. It sends no override, so a tie fails with a generic error.
  - Ties are settled in the conversation modal: **Resolve Turn** opens **Resolve by Majority Vote**, which marks tied metrics "Tie - required" and confirms with **Confirm Resolution**.
  - **Resolve All** breaks ties *randomly*. It skips rows missing any metric and has no confirmation step. The toast reads "Resolve all successful (N turns skipped, M conversations skipped)."
  - The override marker isn't visible, so delete that sentence.
  - The lock applies to Owners too: **Unresolve**, edit, then resolve again.
- **ADD: Annotate from a conversation.** The modal side panel has **Comments** / **Annotate** / **History** tabs. Open it via the **not annotated** / **annotated (x/y)** / **resolved** links under each agent turn, or beside **Metrics** for conversation scope.
- **UPDATE :91-102 comments.** Open them from the **Comments** header button or the **comments** link under a turn. Enter submits and Shift+Enter adds a newline. The limit is 2000 characters. Comments can't be edited or deleted.
- **UPDATE/DELETE :110-122 history.** The **History** tab shows the judge's current scores, the resolved scores, and each reviewer's *latest* values. Delete the "past values", "per-run auto-score history" and audit-trail Note.
- **UPDATE :128 CSV.** One row per reviewer annotation, with reviewer scores and resolved scores. There are no judge scores and no Comments-tab text. Metric columns use IDs and `turn_id` is 0-based.
- **ADD: Calibration section.**
  - Cards: **Overall Agreement**, **Need Attention** (under 75%) and **Coverage**.
  - An **Evaluator Calibration** per-metric chart with MAE.
  - Agreement means the human score is within 1 of the judge's (quantitative) or the labels match exactly (qualitative).
  - Only resolved *turns* count, and the data refreshes on page load.
- **UPDATE FAQ :142, :146, :150.**
  - The previous value is replaced, with no history.
  - The button is named **Re-evaluate**.
  - Only turns the judge scored appear.

### knowledge.mdx
Source: `FE pages/KnowledgePage/index.jsx`, `components/KnowledgeFolder*`, `components/KnowledgeItemDetailPage`; `BE base/arkdock/knowledge.go`; `PY service/parsers/*`, `service/tasks/knowledge.py`.

- **UPDATE :25-43.** The page ("Knowledge Base") opens on **Websites**, and the tab order is Websites, then Documents.
- **UPDATE :33 statuses.** **Pending**, **Running**, **Success**, **Error**.
- **UPDATE :33 upload.**
  - **Upload Documents** takes multiple files (click or drag), up to 10 MB each, with optional **Tags** and **Folder**.
  - Columns: Document, Tags, Status, Updated On, Actions. 50 rows per page.
- **UPDATE :20, :77 formats.** pdf, doc, docx, txt, csv, json, md, jpg/jpeg, png, pptx, xls and xlsx. Images and PPTX are OCR'd. `.ppt` isn't accepted.
- **UPDATE :35 failures.** The detail page shows "Processing failed: <reason>", for example no extractable text or the knowledge quota being exceeded. Fix the cause, then click **Reprocess**.
- **UPDATE :37, :59 delete.** You can delete from the row ⋯ menu, the bulk bar, or the detail page. It frees the item's tokens.
- **UPDATE :47, :81 Add Domain.**
  - Fields: **Domain URL** and **Number of Links to Crawl** (default 50, range 1-1000).
  - This only *discovers* pages. When the row shows **Success**, click **Review URLs**, select pages, then **Add N Pages**.
  - Discovery stays under the URL path you entered and skips file links.
- **UPDATE :59 Websites tab.** It has separate **Domains** and **Links** tables. Deleting a domain entry, which has no confirmation, keeps the pages already added.
- **UPDATE :65 attach.** Knowledge is chosen inside the assistant session, not on the scenario detail page. Scenarios store copied excerpts, so later edits, reprocessing or deletion don't affect saved scenarios.
- **UPDATE :85 refresh.** Use **Reprocess**. It discards manual edits. There's still no automatic re-crawl.
- **UPDATE :37, :89.** Deleting an item doesn't remove knowledge from existing scenarios.
- **ADD: Folders.**
  - The sidebar shows **All items**, **Unfiled** and **FOLDERS**, with **+** to add a folder.
  - Subfolders go 6 levels deep. Names are up to 120 characters, and there are 500 folders per organization.
  - **Include subfolders** toggle.
  - Move items with **Move to folder** or drag.
  - Deleting a folder offers to keep or delete its contents, and never deletes documents.
- **ADD: Tags.** Add them in the add dialogs, **Edit tags**, or the bulk **Tag** action. They filter items in the **Choose documents** picker.
- **ADD: Search and filter.** "Search names and document text...", plus a **Filter** for Status and Tags.
- **ADD: Bulk actions.** **Reprocess**, **Tag**, **Move to folder**, **Delete**, **Clear**.
- **ADD: Detail page.**
  - **Rename**.
  - The AI summary "What this covers".
  - Edit or add content when the status is **Success**; saving re-indexes and counts tokens.
- **ADD.** Only items with **Success** status can be used for scenarios.

### metrics.mdx
Source: `FE pages/MetricsPage/index.jsx`, `components/MetricsSelector.jsx`; `BE base/arkdock/metric.go`; `PY service/metrics/registry.py`; `arksim/evaluator/builtin_metrics.py`.

- **UPDATE :6-22 built-ins.** There are 8 agent metrics:
  - Helpfulness, Coherence, Verbosity, Relevance, Faithfulness (1-5, turn).
  - **User Goal Completion** (0-1, conversation; optional, not preselected).
  - **Agent Behavior Failure** (qualitative, turn).
  - **Tool call behavior** (qualitative, turn; needs captured tool calls).

  There are also 3 simulated-user metrics: Character Consistency, Goal Adherence and Realism. The **Built-in** tab has an **Evaluates** column.
- **UPDATE :19.** Verbosity rewards concise replies; higher means more concise.
- **UPDATE :22.** Agent Behavior Failure labels are: false information, disobey user request, lack of specific information, failure to ask for clarification, repetition, no failure. It's still shipped; the retirement branches aren't merged.
- **UPDATE :32.** The button is **Add Custom Metric**, and the page has **Built-in** and **Custom** tabs.
- **UPDATE :34-60 form.**
  - Field order: Name, Description (**required**, 512 characters), System Prompt, User Prompt Template, Scope, MCP tools, Type, then **Labels** or **Scoring System** (1-5 integer or 0-1 decimal).
  - Defaults: Qualitative, Conversation scope.
  - Name rules: max 100 characters, must start with a letter, unique.
- **ADD: template variables.** `{chat_history}` (required), `{current_turn}`, `{knowledge}`, `{user_goal}`, `{profile}`; `{mcp_evidence}` is required once MCP tools are added. Update the banking example to include `{chat_history}`.
- **ADD: MCP tools on a metric.**
  - Only Owner-approved tools can be added.
  - Turn scope also needs `{current_turn}`.
  - Limits: 5 calls per turn and 20 per conversation.
  - From the second tool on, each tool has a **Required** switch.
- **UPDATE :39.** There's no description tooltip in evaluation results.
- **UPDATE :84.** The buttons are **Create Metric** and **Save Changes**.
- **UPDATE :90-96 management.**
  - Clicking a custom metric opens the edit form; clicking a built-in opens a read-only view.
  - Type and scope can't change after creation.
  - A new version is created only if the current version was used in an evaluation; otherwise the save overwrites it. A version dropdown shows read-only history.
  - Delete is the trash icon on the **Custom** tab, and only for metrics no evaluation has used.
- **UPDATE :102 selector.**
  - Defaults: Helpfulness, Coherence, Relevance.
  - **Create Custom Metric** shortcut and **Search metrics…**.
  - Note that assertions always run.
- **UPDATE :110-112.** Add an availability Note for Metric Alignment.
- **UPDATE FAQ.** Scores can be 1-5, 0-1 or labels.

### metric-alignment.mdx (flag off in prod)
Source: `FE pages/MetricsPage/components/MetricAlignment*.jsx`; `BE base/arkdock/alignment.go`; `PY service/tasks/alignment.py`.

- **ADD: availability Note** at the top (the AGENTS.md rule).
- **UPDATE :12, :85-87 scope.** Alignment supports **turn-scope** metrics only. It isn't available for User Goal Completion, the simulated-user metrics, conversation-scope metrics, or metrics with MCP tools.
- **UPDATE :21, :39 buttons.** Click **Align**, fill in "Start metric alignment", then **Start alignment**. On custom metrics the section is inside the edit form; on built-ins it's at the bottom of the read-only view.
- **UPDATE :23-37 fields.**
  - Only evaluations with resolved turn annotations for the current metric version are listed, each showing "N resolved".
  - **Reflection LM**, **Judge LM** (renamed) and **Delta threshold** (quantitative only, default 1.0) are under **Advanced settings**.
  - Don't quote the UI's stale "gpt-4o (default)" placeholders.
- **UPDATE :57-64 review.**
  - The latest Aligned run shows "Distilled from N disagreements (M agreeing rows skipped)" with **Review & Accept**, then **Accept & Promote**.
  - Built-ins fork into "<name> (aligned)". Guidelines are appended to the system prompt.
  - Editing a custom metric after a run requires a rerun before accepting.
  - Zero disagreements ends in **Error**. There's no discard; just don't accept.
- **UPDATE :46, :50 statuses.** Errors show inline with no retry button, so start a new run. Delete the idempotence sentence. Only one run per metric version can be in progress.
- **VERIFY :39.** The four phases are time-based labels, not real progress.
- **UPDATE :70 history.** Only the 20 most recent runs load, with 5 shown before "Show all (N)". Only Aligned runs expand. Accepted runs show an **Accepted** badge, and evaluation chips show IDs.

### settings.mdx
Source: `FE pages/SettingsPage/*`, `components/PlatformLayout/OrgMenuItems.jsx`, `pages/InviteLandingPage`; `BE base/auth/*`, `base/arkdock/billing/billing.go`, `database/scripts/migrations/arkdock/000004_arkdock_billing.up.sql`.

- **UPDATE :3, :6 tabs.** The tabs are Profile, Members, **API keys**, Usage, Billing.
- **ADD Profile.**
  - Photo: crop dialog with Zoom, then **Apply**. The 2 MB, JPEG/PNG/WebP limits are correct.
  - **Save Changes**.
  - Company and Role refer to the *active* organization. Only that organization's Owners can edit Company.
- **UPDATE :28-31 roles.**
  - Owner-only: API keys, invites, member removal, billing changes, company name, MCP tool run and approve, and annotation resolution.
  - Members can view the team list, Usage and Billing, but can't change billing.
- **UPDATE :33 invite.**
  - **Add New Member** opens "Invite Team Member" (Email Address, Role Owner/Member), then **Send Invitation**.
  - Invites expire in 7 days.
  - The **Invited** tab is Owner-only and offers **Copy Invite Link**, **Resend Invite** and **Revoke Invite**.
- **DELETE :35.** There's no role dropdown; role changes aren't in the UI.
- **UPDATE :37 remove.** Use **Remove from organization**. The member's API keys stop working, and a user with no other organization gets a new personal one.
- **ADD: Leave organization.** **Leave Organization** appears only when you belong to more than one. It's refused for the only Owner.
- **ADD: API keys section.** Link it to `api.mdx`.
- **UPDATE :43 Usage (wrong today).**
  - Cards: **Tokens Consumed** and **Compute Time**.
  - **Token Usage Over Time** chart.
  - Ranges: **Last 7 Days**, **Last 30 Days** (default), **Last 12 Months**.
  - **Breakdown by Type**: Simulation, Scenario Generation, Evaluation.
  - It doesn't count API calls or conversations.
- **UPDATE :60.** Free allows 2 at a time *per operation type*. Pro and Enterprise have no plan limit.
- **UPDATE :61.** Pro overage is $0.01 per 1K input tokens and $0.05 per 1K output tokens. Enterprise is "Custom".
- **ADD :63.** Support row: Community / Email / Dedicated CSM + SLA. SSO/SAML depends on decision 4.
- **UPDATE :71-72.** Input and output tokens count only Arklex's models (simulated user, scenario generation, judge). Your agent's tokens aren't counted.
- **UPDATE :73, :77 knowledge tokens.** A hard limit on *every* plan, Pro included, with no overage. Deleting items frees tokens. "Contact support to raise the limit."
- **UPDATE :67-73 meters.**
  - Labels: "(never resets)", "Approaching limit", "Allowance exceeded".
  - "Billing period: start - end". Free periods are rolling 30-day windows.
  - Pro shows "Estimated charge".
- **UPDATE :77 spending cap.**
  - Owners use **Adjust cap** ("Set monthly spend limit"). The cap is at most $2,000, and $0 blocks overage.
  - Reaching the cap pauses new simulations, evaluations and generation until the next period.
- **UPDATE :81-83 buttons.**
  - **Upgrade to Pro**, **Manage subscription**, **Downgrade**, **Contact us**.
  - **Cancel subscription** finishes in the Stripe portal. **Reactivate** undoes a scheduled cancellation.
  - Only Owners can change the plan.
- **UPDATE FAQ :96-98 (wrong).** Multiple organizations are supported. Accept an invite to join one, then switch from the account menu. You can't create organizations yourself.
- **UPDATE FAQ :104-106.** Members also can't see pending invites.
- **ADD: account menu.** Organizations switcher, **Theme** (Light/Dark/System), **Log out**, and the UI and API versions.
- **VERIFY.** Google sign-in ("Continue with Google") and email verification depend on runtime config. Check prod before documenting them.

### support.mdx
- See Site-wide. Rebrand it, fix heading case, align LinkedIn, reword the GitHub card, and add the version tip.

---

## 5. Images

**Every screenshot shows the old sidebar** (no MCP or API Reference), so a full re-shoot is simplest. Specific problems:

| Image | Problem |
| --- | --- |
| `create-a-scenario.jpg`, `generate-at-scale.jpg`, `drafts-status-1.jpg`, `draft-failed.jpg` | The flows they show are removed. Delete them. |
| `draft-only.jpg`, `import-a-file.jpg` | Old Drafts layout; "Create a new group instead" link no longer exists |
| `run-simulation.png` | "max 50", no concurrency or capture fields, "GPT-5.4" |
| `simulation-table.png` | Used as the "Detail header" but shows the list page |
| `simulations-convo-table.png` | No **Tool calls** column |
| `evaluation-table.png`, `agent-evaluations.jpg` | Errors column removed; no Agent/Simulated user badge |
| `run-evaluation.png`, `metrics-types.png` | No **Evaluate** toggle or assertions note; Goal Completion locked |
| `evaluation-quant-metrics.png` | No Conversations passed card; "Goal Completion" |
| `evaluation-convo-view.png` | Shows the removed final score and status |
| `metrics-built-in.png` | 7 rows (now 11), no **Evaluates** column |
| `metrics-custom.png` | Captioned as the creation form but shows the list |
| `knowledge-documents.png`, `knowledge-websites.png` | No folders sidebar; old search placeholder |
| `agent-simulations.jpg`, `agent-config.jpg` | Old columns; no Custom HTTP; no tracing card |

**Unreferenced** (delete or reuse): `arklex-ai-dark-logo.svg`, `arklex-ai-light-logo.svg`, `billing.png`, `checks-passed.png`, `connect-agent-form.png`, `drafts-status.jpg` (duplicate), `evaluation-convos-table.png`, `evaluation-detail.png`, `images/favicon.svg`, `hero-dark.png`, `hero-light.png`, `new-simulation.png`, `simulation-table-1.png` (duplicate), `logo/dark.svg`, `logo/light.svg`.

---

## 6. Product bugs noticed (not docs changes)

| Area | Issue | Where |
| --- | --- | --- |
| Evaluations list | **Failed** filter sends `failed`; the backend only knows `error`, so it returns 400 | `FE constants/evaluation.js:13`, `BE base/arkdock/evaluation.go:785-790` |
| Agents | The A2A form pre-fills `model`, which the backend rejects on save | `FE pages/AgentsPage/agentFormUtils.js:4`, `BE base/arkdock/agent.go:688` |
| Agents | A saved description can't be cleared | `FE ConfigurationTab.jsx:386`, `BE agent.go:921` |
| Agents | Test Connection hides the reason for a 400 (shows only "Connection failed") | `FE ConfigurationTab.jsx:271` |
| Sign up | The UI says 7+ characters; the backend requires 8 | `FE constants/passwordCriteria.js:2`, `BE base/auth/arkdock_user.go:558` |
| Annotations | The "Resolution computation" tooltip describes averaging and spread thresholds that don't exist | `FE AnnotationsTab.jsx:782-792` |
| Annotations | The spreadsheet **Resolve** on a tie shows a generic error instead of opening the tie dialog | `FE services/evaluation/index.js:991-1009` |
| Annotations | The footer says "Admin - edit any cell to override" (the role is Owner) | `FE AnnotationsTab.jsx:1126` |
| Annotations | Members can export every reviewer's scores via CSV, which bypasses blind review | `BE main.go:406` |
| MCP | The page subtitle says simulated users can call tools; the simulation runner never uses MCP | `FE pages/McpPage/index.jsx:460` |
| Import | The "Your knowledge" picker is always empty | `FE ImportScenariosWizard.jsx:445` |
| Alignment | Stale "gpt-4o-mini (default)" / "gpt-4o (default)" placeholders | `FE MetricAlignmentStartDialog.jsx:224,245` |
| Knowledge | The quota error says "for this billing period" for a lifetime limit | `PY service/tasks/knowledge.py` |
| Scenarios | The error says "Context tab"; the tab is **Plan** | `PY service/session_turn.py:1218` |
| Simulations | The rerun toast says "rerun completed!" when the rerun was only created | `FE RerunSheet.jsx:43` |
| Simulations | The New Simulation sheet loads only the first 20 agents and 20 groups | `FE hooks/useSimulationPrerequisites.js` |
| API keys | Revoke confirmation says "Delete" | `FE ApiKeysTab.jsx` |
| Settings | The `ownerSettingsTour` says "Usage limits reset monthly" (it's never run) | `FE tours/definitions/ownerSettingsTour.js` |
