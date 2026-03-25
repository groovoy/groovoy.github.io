h1. Polarion \u2192 Jira Cloud Integration
*Built on Atlassian Forge \u00b7 Internal Application \u00b7 Status: MVP deployed*

----

{toc:maxLevel=2|minLevel=2|type=list|printable=false}

----

h2. Overview

This page documents the *JiraPolarionApp* \u2014 an internal Atlassian Forge application that surfaces Polarion ALM requirements directly inside Jira Cloud issue panels.

Engineers working in Jira can now see the Polarion requirements linked to their work without leaving Jira, switching tools, or asking the systems team for a report.

{info:title=Current status}
*MVP is live* on rowojdevs.atlassian.net as of March 2025. The panel displays 8 sample Electric Vehicle Platform requirements using mock data. Real Polarion connectivity is the next milestone.
{info}

----

h2. What the app does

h3. For engineers using Jira

Every Jira issue now has a *Polarion* panel in the right sidebar. The panel shows all Polarion requirements associated with that issue, including:

|| Field || Description ||
| Requirement ID | Polarion item ID, e.g. REQ-1, REQ-3 |
| Status | Current Polarion workflow status (draft, open, inProgress, inReview, done) |
| Priority | Requirement priority (low, medium, high, urgent) |
| Title | Full requirement text |
| Module | Which Polarion document/module the requirement belongs to |
| Last updated | How long ago the requirement was last modified in Polarion |

h3. Screenshot \u2014 panel in action

!polarion-panel-screenshot.png|thumbnail!

_Screenshot: the Polarion panel displaying 8 Electric Vehicle Platform requirements inside a Jira issue. Status and priority are colour-coded using Jira's native badge system._

{note:title=Adding your screenshot}
Upload your screenshot to this page as an attachment and replace the image macro above with the filename you used.
{note}

h3. Status badge colour guide

|| Badge colour || Polarion status ||
| {color:#00875A}*Green*{color} | done |
| {color:#0052CC}*Blue*{color} | open |
| {color:#6554C0}*Purple*{color} | inReview |
| {color:#FF991F}*Orange*{color} | inProgress |
| {color:#5E6C84}*Grey*{color} | draft |

----

h2. Technical architecture

h3. Platform

The app runs entirely on *Atlassian Forge* \u2014 Atlassian's serverless hosting platform. There are no external servers, no Docker containers, and no infrastructure to manage. Atlassian hosts and executes all app code.

{panel:title=Key platform properties|borderStyle=solid|borderColor=#0052CC|titleBGColor=#E6F1FB}
* *Serverless* \u2014 runs inside Atlassian's cloud, scales automatically
* *Zero hosting cost* \u2014 internal single-install apps stay within Forge's free tier
* *Encrypted storage* \u2014 all credentials stored in Forge's built-in KV store, never logged
* *No third-party SaaS* \u2014 data stays inside Atlassian's trust boundary
{panel}

h3. File structure

{code:title=Project layout}
jirapolarionapp/
\u251c\u2500\u2500 manifest.yml              \u2190 App definition, permissions, module config
\u251c\u2500\u2500 package.json
\u251c\u2500\u2500 src/
\u2502   \u251c\u2500\u2500 index.js              \u2190 Entry point (exports handler)
\u2502   \u251c\u2500\u2500 resolvers/
\u2502   \u2502   \u2514\u2500\u2500 index.js          \u2190 Backend: getPolarionData handler
\u2502   \u251c\u2500\u2500 frontend/
\u2502   \u2502   \u2514\u2500\u2500 index.jsx         \u2190 UI: Jira issue panel (UI Kit 2)
\u2502   \u2514\u2500\u2500 sync/
\u2502       \u2514\u2500\u2500 mockData.js       \u2190 8 sample EV Platform requirements
{code}

h3. Data flow (current MVP \u2014 mock mode)

{noformat}
Jira issue panel loads
        \u2193
  invoke('getPolarionData')
        \u2193
  resolver reads mockData.js
        \u2193
  returns array of 8 requirements
        \u2193
  UI renders RequirementRow for each item
{noformat}

h3. Data flow (future \u2014 live Polarion)

{noformat}
Jira issue panel loads
        \u2193
  invoke('getPolarionData')
        \u2193
  resolver calls Polarion REST API v1
  GET /polarion/rest/v1/projects/{id}/workitems?query=type:requirement
        \u2193
  returns paginated work items
        \u2193
  UI renders RequirementRow for each item
{noformat}

h3. Technology stack

|| Layer || Technology || Notes ||
| Runtime | Atlassian Forge / Node.js 24 | ARM64, 256 MB memory |
| Frontend | UI Kit 2 (@forge/react) | Native Jira components, no custom CSS |
| Backend | @forge/resolver | Serverless function handlers |
| Storage | Forge KV Storage | Encrypted, per-install isolated |
| External API | Polarion REST API v1 | Siemens OpenAPI spec |
| Auth (future) | Polarion Personal Access Token (PAT) | Stored in Forge encrypted storage |

h3. Polarion REST API

The app is built against the official Polarion REST API v1 specification published by Siemens:
[Polarion REST API spec|https://developer.siemens.com/polarion/polarion-rest-apispec.json]

Work item response shape used in the app:

{code:language=javascript|title=Polarion work item structure}
{
  type: "workitems",
  id: "ProjectId/REQ-1",
  attributes: {
    id: "REQ-1",
    title: "Battery management system shall monitor cell voltage within \u00b15 mV",
    description: { type: "text/html", value: "<p>...</p>" },
    status:   { id: "inReview" },
    priority: { id: "high" },
    type:     { id: "requirement" },
    updated:  "2025-02-03T14:22:00Z"
  },
  relationships: {
    author:  { data: { id: "m.kowalski", type: "users" } },
    module:  { data: { id: "ProjectId/Requirements/BMS_System" } }
  }
}
{code}

----

h2. Installation and configuration

h3. Prerequisites

|| Requirement || Details ||
| Atlassian Forge CLI | npm install -g @forge/cli |
| Node.js | Version 18 or higher |
| Jira Cloud site | Admin access required for install |
| Forge account | forge login |

h3. Deploy and install

{code:language=bash|title=Deploy to development}
# Clone or copy the project
cd jirapolarionapp

# Install dependencies
npm install

# Deploy to Atlassian's servers
forge deploy

# Install on your Jira site
forge install
# Select: Jira
# Enter site: rowojdevs.atlassian.net
{code}

h3. Configuration (current MVP)

No configuration is required for the mock data MVP. The panel works immediately after install.

When connecting to a real Polarion server (see roadmap), admins will configure the following in *Jira Settings \u2192 Apps \u2192 Polarion Integration Settings*:

|| Setting || Example value || Notes ||
| Polarion host URL | https://polarion.yourcompany.com | No trailing slash |
| Personal Access Token | pat-abc123... | Generate in Polarion: My Profile \u2192 Personal Access Tokens |
| Polarion project ID | ElectricVehiclePlatform | Case-sensitive |
| Work item query | type:requirement | Polarion Lucene syntax |
| Jira project key | PROJ | Target project for sync |

h3. Switching from mock to live data

When the Polarion server is ready, one line changes in {{src/sync/syncEngine.js}}:

{code:language=javascript}
// Change this:
const USE_MOCK_API = true;

// To this:
const USE_MOCK_API = false;
{code}

Then run {{forge deploy}} again. No other code changes needed.

----

h2. Roadmap

h3. Phase 1 \u2014 Production ready _(next 8 weeks)_

{checklist}
{checklist:status=TODO}Connect to real Polarion REST API (AWS Data Center){checklist}
{checklist:status=TODO}Admin settings page \u2014 store credentials securely{checklist}
{checklist:status=TODO}Scheduled automatic sync (hourly/daily){checklist}
{checklist:status=TODO}Create Jira issues from Polarion requirements{checklist}
{checklist:status=TODO}Update existing Jira issues when Polarion changes{checklist}
{checklist:status=TODO}Attachment sync{checklist}
{checklist:status=TODO}Comment sync{checklist}
{checklist:status=TODO}Audit log \u2014 history of all sync runs{checklist}
{checklist}

h3. Phase 2 \u2014 Enterprise features _(weeks 9\u201320)_

{checklist}
{checklist:status=TODO}Bidirectional sync (Jira \u2192 Polarion){checklist}
{checklist:status=TODO}No-code field mapping UI for admins{checklist}
{checklist:status=TODO}Multi-project support (multiple Polarion \u2192 Jira pairs){checklist}
{checklist:status=TODO}Traceability matrix view in Jira{checklist}
{checklist:status=TODO}Inline Polarion detail preview with deep link{checklist}
{checklist:status=TODO}Conflict detection and resolution UI{checklist}
{checklist}

h3. Phase 3 \u2014 Differentiated _(weeks 21\u201340)_

{checklist}
{checklist:status=TODO}AI-assisted field mapping suggestions{checklist}
{checklist:status=TODO}Requirements coverage dashboard{checklist}
{checklist:status=TODO}Compliance report generator (ISO 26262, IEC 62443){checklist}
{checklist:status=TODO}Polarion Live Document sync (chapters \u2192 epics){checklist}
{checklist:status=TODO}Change impact analysis \u2014 suspect link warnings in Jira{checklist}
{checklist}

----

h2. Competitive comparison

|| Feature || Our app || OpsHub OIM || Notes ||
| Jira Cloud support | {color:#00875A}\u2713{color} | {color:#00875A}\u2713{color} | OpsHub is the only other Cloud option |
| Forge-native (no ext. server) | {color:#00875A}\u2713 Advantage{color} | {color:#BF2600}\u2717 SaaS{color} | Our app stays inside Atlassian's trust boundary |
| No third-party credentials exposure | {color:#00875A}\u2713{color} | {color:#BF2600}\u2717{color} | OpsHub requires sending credentials to their platform |
| Bidirectional sync | Planned Phase 2 | {color:#00875A}\u2713{color} | |
| Custom field mapping UI | Planned Phase 2 | {color:#00875A}\u2713{color} | |
| Compliance report export | Planned Phase 3 | {color:#BF2600}\u2717{color} | Our differentiator |
| Monthly cost (200 users) | ~$0 internal | ~$200-500/mo | Forge free tier covers single internal install |

----

h2. Frequently asked questions

*Q: Does this app send any data outside of Atlassian?*
In mock mode: no, nothing leaves Atlassian. In live mode: the app calls your company's own Polarion server. No data goes to any third party.

*Q: Where are the Polarion credentials stored?*
In Forge's built-in encrypted key-value storage, hosted by Atlassian. They are never logged, never visible in the browser, and never sent anywhere except to your Polarion server.

*Q: Will this slow down Jira?*
No. The panel loads asynchronously \u2014 Jira renders immediately and the Polarion data appears a moment later. A loading spinner shows while data is fetching.

*Q: What happens if Polarion is down?*
The panel will show an error message. Jira itself is unaffected.

*Q: How is this different from buying OpsHub from the Atlassian Marketplace?*
OpsHub runs as an external SaaS platform \u2014 your Polarion and Jira credentials are sent to a third-party service. Our app runs entirely inside Atlassian's own infrastructure. It is also free for internal use versus several hundred dollars per month for OpsHub at typical team sizes.

----

h2. Contact and ownership

|| Item || Detail ||
| App name | JiraPolarionApp |
| Environment | Development (internal) |
| Jira site | rowojdevs.atlassian.net |
| Platform | Atlassian Forge |
| App ID | ari:cloud:ecosystem::app/aecaa025-d792-4435-ad47-c467c9dd3d49 |
| Status | MVP \u2014 mock data live |
| Next milestone | Connect to real Polarion REST API |

----

_Last updated: March 2025 \u00b7 Page maintained by the integration team_
