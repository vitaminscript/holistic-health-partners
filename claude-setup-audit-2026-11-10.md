# Claude Setup Audit (Detox, read-only pass)

**Date:** November 10, 2026
**Prepared for:** Michael Burton (Holistic Health Partners, Covenant Health Products, VitaminScript)
**Status:** Read-only. Nothing was deleted, disabled, rewritten, migrated, purchased or scheduled.

One change was made during this session, with your approval and before the audit began: `www.youtube.com` was added to the allowed hosts of your Default cloud environment. It did not help (YouTube still blocks the cloud machine), so removing it is on the approval list below.

---

## 1. Executive summary

1. **Product Scout emails are not reaching you.** The weekly and monthly Product Scout tasks send through the Gmail connector, and all three sends this month bounced (Nov 1, Nov 2, Nov 9). The bounce notices are in Gmail Trash. The scheduler shows each run as succeeded, so nothing flagged it.
2. **Robinhood is attached to nine business scheduled tasks.** Those tasks run unattended with automatic approval, and the Robinhood connector includes order-placement tools. None of the nine tasks uses it.
3. **Most of your loaded skills are not yours and not relevant.** About 81 plugin skills load in this session (Sales, Product Management, Design, Operations, Productivity, Superpowers) against 15 skills you wrote. Each session also tries and fails to connect 25 plugin connectors such as Salesforce, HubSpot, Gong and Asana.
4. **The Superpowers plugin changes how every session behaves.** It injects a rule that a skill must be invoked before any response, and it forced a design-approval process onto your first message here. It is built for software development.
5. **Three instruction conflicts need your decision**, listed in section 3: nurse vs. medical practitioner wording, auto-publish vs. the claims QA gate, and Firecrawl vs. no Firecrawl in Product Scout.
6. **Your master prompt is seven months stale.** It still says "Current Focus (April 2026)" and loads into every session.
7. **One task calls a top-tier model 24 times a day to run a script.** The hourly spreadsheet watcher finishes in about four seconds per run; the model adds nothing on the runs where the file is unchanged.

What I could not see is listed in section 8. The biggest gaps are your Mac's local files, your regular Claude Desktop chats, and Cowork's locally stored tasks.

---

## 2. Inventory

### 2.1 Your custom skills (15)

All are synced from your claude.ai account. In this session they live under `/root/.claude/skills/synced/<account>/`.

| Skill | SKILL.md size | Notes |
|---|---|---|
| `cpd` | 67.0 KB | Version 13. Auto-publishes to BigCommerce. Largest skill by far. |
| `cpd-rewrite` | 34.2 KB | Version 11, standard release CPD-2026-09-08-1 |
| `cpd-audit` | 20.9 KB | Audits against the `cpd-rewrite` P01–P22 standard |
| `vs-cohort-retention` | 12.6 KB | Used by a monthly scheduled task |
| `blog-claims-qa` | 12.1 KB | Compliance gate for blog posts and product descriptions |
| `covenant-blog-from-url` | 11.1 KB | URL in, DropInBlog draft out |
| `product-scout` | 10.7 KB | Used by weekly and monthly scheduled tasks |
| `covenant-blog-post` | 9.7 KB | NotebookLM and podcast-embed route |
| `m4a-to-mp3` | 9.1 KB | Audio conversion and embed code |
| `klaviyo-retention-flow` | 8.5 KB | Dated April 2026 |
| `search-banner` | 8.4 KB | Built Oct 30, 2026 |
| `cominv` | 8.3 KB | Customs invoices |
| `weekly-retention-report` | 6.0 KB | Created Apr 25, 2026. No scheduled task found for it. |
| `hash-hermes-setup` | 3.7 KB | Written for Manus, not Claude |
| `blog-image` | 3.2 KB | Blog cover images |

### 2.2 Built-in skills enabled on your account (15)

`docs`, `google-workspace`, `computer-use`, `chrome-browser`, `built-in-browser`, `deep-research`, `import-memory`, `morning`, `skill-creator`, `mcp-builder`, `canvas-design`, `xlsx`, `pptx`, `pdf`, `docx`.

### 2.3 Plugin skills loaded in this session (about 81)

Location: `/root/.claude/plugins/synced/<account>/`. Your account's plugin list came back empty, yet these are synced into the session, so they may be coming from an organization-level or Cowork setting. I could not determine which.

| Plugin | Skills | Relevant to your work? |
|---|---|---|
| Sales | 36 | Unlikely. Built for B2B account executives (pipeline, forecast, renewals). |
| Superpowers (v6.4.2, third-party) | 15 | Only for coding sessions |
| Operations | 9 | Occasionally |
| Product Management | 8 | Unlikely |
| Design | 7 | Occasionally |
| Productivity | 4 | Possibly |
| Cowork plugin management | 2 | Only when building plugins |

### 2.4 Connectors (26)

| Status | Connectors |
|---|---|
| Connected (24) | Ahrefs, Calendly, Canva, Cloudflare, Descript, DropinBlog, Gmail, Google Calendar, Google Drive, Granola, Intuit QuickBooks, Jotform, Klaviyo, Microsoft 365, Pipeboard-Google, Pipeboard-Meta, PubMed, RingCentral Chat, RingEX, Robinhood, Slack, SuperScale, Vercel, Zapier |
| Connect incomplete (2) | "Slack cd890b01" (a duplicate of Slack), Stripe |
| Failed this session (25, from plugins) | Asana, Atlassian, Figma, Intercom, Linear, Notion, Amplitude, Amplitude EU, ClickUp, Fireflies, Monday, Pendo, Similarweb, Apollo, Clay, Close, Crunchbase, Gong, HubSpot, Lusha, Otter, Outreach, Salesforce, Zoom, ZoomInfo |

Your scheduled tasks also rely on local connectors on your Mac that I cannot see from here: BigCommerce, Salesforce, Google Sheets, Klaviyo-VS, Klaviyo-Covenant, Outlook-CHP and Firecrawl.

### 2.5 Scheduled tasks (13 cloud tasks, all enabled, all last runs "succeeded")

| # | Task | Schedule (Eastern) | Model | Last run | Notes |
|---|---|---|---|---|---|
| 1 | Morning brief | Daily, about 8:25 am | default | Nov 9 | System-generated brief |
| 2 | Heartbeat | 5 times a day | default | Nov 9 | System-generated check-ins |
| 3 | CHP blog: daily categories + most read | Daily 6 am | Sonnet 5.5 | Nov 9 | Only DropinBlog attached. Pushes to `main` of `chp-help-widget`. |
| 4 | Product Scout monthly | 1st, 5:52 am | Opus 5.5 | Nov 1 (27 min) | Email bounced |
| 5 | Product Scout weekly | Mondays 5:47 am | Opus 5.5 | Nov 9 (14 min) | Email bounced |
| 6 | Order-history feed freshness check | 8 am and 4 pm | Opus 5 | Nov 9 | Push notification only |
| 7 | Watch CHP_Brands_Ordered_2026.xlsx | Hourly at :47 | Opus 5 | Nov 9 (4 sec) | 24 runs a day |
| 8 | Morning Ops Reports dispatch | Weekdays 7 am | Opus 5 | Nov 9 (4 sec) | Kicks off a GitHub workflow |
| 9 | Consult Attribution & Bonus Report | 13th and 27th–30th, 7 am | Opus 5 | Oct 30 | Fires 5 days a month, acts on 2 |
| 10 | Morning Consult Call Sheet | Weekdays 7 am | Opus 5 | Nov 9 (9 min) | Emails you and Carolyn |
| 11 | VS Cohort Report pre-flight check | 27th, 7 am | Opus 5 | Oct 27 | |
| 12 | VS Monthly Cohort Retention Report | 1st, 7 am | Opus 5 | Nov 1 (36 min) | |
| 13 | (none found) | | | | `weekly-retention-report` says "every Monday" but has no cloud task |

Tasks stored locally by the Cowork desktop app are not visible from here, so the monthly sales report and anything else scheduled in Cowork may exist without appearing above.

### 2.6 Sessions (46 Claude Code sessions)

24 archived, 21 idle, 1 running (this one). This covers Claude Code sessions in the cloud and on your Macs. It does not cover regular Claude Desktop chats or Cowork sessions.

### 2.7 Instruction files

| Item | Found |
|---|---|
| Master prompt (user preferences) | Yes. Version 1.0, April 2026. Loads into every session. |
| `CLAUDE.md` or `AGENTS.md` in `holistic-health-partners` | None |
| Global `CLAUDE.md` in this cloud session | None |
| Claude memory | Exists on your Mac (a Nov 2 session saved a rule to it). Not readable from here. |

---

## 3. Issues

Severity: **High** means something is failing or exposed now. **Medium** means a real conflict or waste. **Low** means tidiness.

### High

**H1. Product Scout emails bounce and are lost.**
- *Evidence:* Three bounce notices in the personal Gmail account, all in Trash: "Product Scout — Monthly New Products, November 2026" (Nov 1), "…week of November 2" (Nov 2), "…week of November 9" (Nov 9). Each says the account no longer has access to the old `mburton@` send-as address. Your order-history watchdog's prompt already documents this exact failure and avoids Gmail for that reason; the two Product Scout tasks were never updated.
- *Impact:* You are not getting the new-product lists that feed the Thursday New Arrivals email. Findings may still be in the Google Sheet, which I did not check.
- *Proposed fix:* Switch both tasks to the Microsoft 365 send (see H2 for why that appears to work), or fix the Gmail send-as address in Gmail settings.

**H2. The watchdog's note about Microsoft 365 contradicts what is happening.**
- *Evidence:* The order-history task prompt says Microsoft 365 cannot send mail (permission error, as of Nov 9). Yet the Consult Call Sheet arrived by that same Microsoft 365 send on Nov 5, 6 and 9, to you and Carolyn.
- *Impact:* One of the two statements is wrong. If the note is wrong, the watchdog is needlessly limited to push notifications. If it is right for some sessions, the call sheet could stop without warning.
- *Proposed fix:* Run one test send from a fresh cloud session, then correct the note.

**H3. Robinhood is attached to nine unattended business tasks.**
- *Evidence:* Tasks 4 through 12 list Robinhood among their connectors. The connector includes tools that place, review and cancel equity, option and crypto orders. Eight of the nine tasks run in automatic permission mode. None of their prompts mention Robinhood.
- *Impact:* A personal brokerage account is reachable by automated runs that read outside content (supplier websites, email, spreadsheets). The likelihood of misuse is low, but there is no upside to the exposure.
- *Proposed fix:* Remove Robinhood from all nine tasks. Consider whether it belongs on the business Claude account at all.

**H4. Scheduled tasks carry 22 to 26 connectors each and use one to three.**
- *Evidence:* The spreadsheet watcher runs one local script yet has 22 connectors attached. The CHP blog task is the good example: it has only DropinBlog.
- *Impact:* More access than needed on unattended runs, plus extra context on every run.
- *Proposed fix:* Trim each task to the connectors named in its prompt.

### Medium

**M1. "Nurse" vs. "medical practitioner" wording conflict.**
- *Evidence:* A Nov 2 session changed the Black Friday page and homepage panel to say "medical practitioner", removed "nurse", and saved that as a rule to memory. Your master prompt and seven skills still require the CTA "Carolyn Mulally BS, RN, HN-BC" (`cpd`, `covenant-blog-post`, `covenant-blog-from-url`, `klaviyo-retention-flow`, `blog-claims-qa`, `vs-cohort-retention`).
- *Impact:* I can't read the memory rule, so I don't know its scope. If it is broad, it conflicts with the credential line in every blog and product CTA.
- *Proposed fix:* You decide the rule. For example: credentials stay in bylines and CTAs, while the word "nurse" is avoided in promotional copy. Then I record it in one place.

**M2. `cpd` auto-publishes while `blog-claims-qa` says nothing is final before QA.**
- *Evidence:* `cpd` says it writes "directly to BigCommerce via MCP — no confirmation needed when a Product ID is provided." `blog-claims-qa` says "Always run this skill … BEFORE a CPD description is considered final" and "Never run it in the same chat that drafted the content." Your master prompt says to confirm the approach before executing.
- *Impact:* A product description can go live before the compliance gate has run.
- *Proposed fix:* Your call between two options: keep auto-publish and treat QA as a follow-up audit, or publish only after a QA pass. I am flagging this, not choosing.

**M3. Product Scout tasks disagree on Firecrawl, and the skill has an unfilled setting.**
- *Evidence:* The weekly task says "never use Firecrawl." The monthly task says "firecrawl MCP for scraping." The skill itself has `PRODUCT_SCOUT_SHEET_ID = FILL_ME_IN`, which both task prompts override. A Nov 9 session ("Product Scout prompt updates") is blocked waiting for you to paste revised prompts on your Mac.
- *Proposed fix:* Finish that pending update, apply one Firecrawl rule to both, and put the sheet IDs into the skill so the task prompts can shrink.

**M4. Plugin skills and failed plugin connectors.**
- *Evidence:* About 81 plugin skills listed in every session. 25 plugin connectors fail with a 403 on every session start here.
- *Impact:* More chances of the wrong skill triggering (for example, Sales `daily-briefing` vs. your `morning` brief), and a longer skill list for the model to sort through.
- *Proposed fix:* Turn off Sales and Product Management at least. Decide on Design, Operations and Productivity.

**M5. Superpowers forces a developer workflow onto every request.**
- *Evidence:* Its session-start text says a skill must be invoked "BEFORE any response or action." In this session it loaded a 17.5 KB brainstorming skill and required a written-spec approval path for what you asked.
- *Impact:* Slower, more expensive sessions for marketing and operations work.
- *Proposed fix:* Disable it, or keep it only in coding projects.

**M6. Hourly spreadsheet watcher uses a model to run a script.**
- *Evidence:* 24 runs a day on Opus 5. Each run executes one Python script and, when nothing changed, replies "Unchanged."
- *Proposed fix:* Either have a launchd job on the Mac run the script and notify only on change, or cut the frequency to business hours and use a smaller model.

**M7. The master prompt is stale and partly contradicted by practice.**
- *Evidence:* "Current Focus (April 2026)" describes a 9-project build. The tool list names Gorgias, Motion and Lindy.ai, which do not appear in any skill, task or connector I could see, and omits Salesforce, Calendly, DropinBlog-driven automation and Vercel, which are in daily use. Rule 2 ("never assume, ask") conflicts with skills and tasks that say "run end to end without asking."
- *Proposed fix:* A shorter version 2.0 with current focus, the actual tool list, and a line that scheduled tasks and auto-publish skills are standing approvals.

**M8. Overlapping skills.**
- `covenant-blog-post` and `covenant-blog-from-url` both trigger on "write a blog post." They differ only in the NotebookLM and podcast steps. Both publish as draft, which is consistent.
- `cpd`, `cpd-rewrite` and `cpd-audit`: `cpd` is at version 13 and `cpd-rewrite` at version 11 with its own named standard. `cpd-audit` checks against the `cpd-rewrite` standard only. I did not verify that `cpd` follows the same P01–P22 rules; if it does not, new products and rewritten products are held to different standards.
- `vs-cohort-retention` (monthly, scheduled) and `weekly-retention-report` (weekly, not scheduled) cover similar ground.

### Low

**L1. Skill metadata problems.** `weekly-retention-report` has no description field, a display-style name ("Weekly Retention Report"), and extra top-level keys (`version`, `created`, `author`, `stores`). Its listed description is a version stamp, so it will rarely trigger on its own. `cpd` has a top-level `version` key. Current rules for uploaded skills allow only `name`, `description`, `license`, `compatibility`, `metadata` and `allowed-tools`, so a re-upload of either may be rejected until fixed.

**L2. `hash-hermes-setup` targets a different assistant.** It tells Manus what to do. In Claude it is only a reference document.

**L3. Two half-connected connectors.** "Slack cd890b01" duplicates your working Slack connector. Stripe is incomplete; a Nov 7 session dealt with a Stripe key incident, so decide deliberately whether to finish or remove it.

**L4. Stale note in Morning Ops task.** It warns that the schedule must move when clocks change in November. The schedule is already pinned to Eastern time, so the note is obsolete.

**L5. Consult Attribution task wakes five days a month to act on two.** Minor cost; the model is invoked just to decide whether to run.

**L6. Mixed model versions.** Seven tasks use Opus 5, two use Opus 5.5, one uses Sonnet 5.5. The 4-second dispatch and watchdog tasks do not need a top-tier model. Changing models is your decision.

**L7. Is the Call Sheet output wanted?** On Nov 5 Carolyn replied to the Morning Consult Call Sheet asking what it is for, since the information is already in Salesforce. Worth confirming the team uses it before paying for a daily 9-minute run.

**L8. Sessions waiting on you.** Three are blocked on a decision:
- "Product Scout prompt updates": paste the saved prompts on your Mac.
- "Searchease vs conversational search review": start Phase 1 or hold.
- "Metagenics deck": bring back the meeting recording.

Two more are marked ready for review: "File capacity exceeded" and "Orphan product redirects and cleanup".

**L9. Sessions safe to archive.** 16 idle sessions are completed work. Organizational benefit only; archiving them saves no tokens.

---

## 4. Token and cost opportunities

### Measured in this session

| Item | Measurement |
|---|---|
| Your 15 custom skills, full text | 226 KB total. Loads only when a skill is used. |
| `cpd` when invoked | 67 KB, roughly 17,000 tokens each time |
| Superpowers brainstorming skill, forced on first request | 17.5 KB, roughly 4,400 tokens |
| Plugin connectors failing at start | 25 per session |
| Hourly watcher | 24 model runs a day, about 4 seconds each |

### Estimated (assumes about 4 characters per token)

| Item | Estimate | Assumption |
|---|---|---|
| Connector tool names listed in each session | 1,350–1,400 names, roughly 12,000 tokens | Hand count. In this session only the names load; full definitions load on demand. |
| Connector instruction blocks | Roughly 6,000–7,000 tokens | 14 connectors supply standing instructions |
| Skill list (names and descriptions) | Roughly 5,000–6,000 tokens | Plugin skills are the majority |
| Master prompt | Roughly 1,400 tokens | Loads in every session |
| Superpowers start-up text | Roughly 900 tokens | Loads in every session |

Largest connectors by tool count (approximate): Vercel 250, Ahrefs 135, Pipeboard-Meta 125, Klaviyo 115, QuickBooks 110, Robinhood 100.

### What these numbers do not tell you

- They describe a Claude Code cloud session. Claude Desktop chat and Cowork load things differently, and I could not measure those.
- I have no usage or billing data, so there are no dollar figures here.
- Cached context is charged differently from fresh context. A large fixed start-up load costs less than the raw token count suggests.
- Archiving old sessions does not reduce what a new session loads.
- The likely real savings are in scheduled tasks (run frequency, model choice, connector lists), not in tidying.

### On `disable-model-invocation`

Verified against current Claude Code documentation: setting it to `true` keeps a skill's description out of context and makes it manual-only. It is a Claude Code setting. It is not an accepted field for skills uploaded to claude.ai, which is where all 15 of your custom skills live, so I do not recommend adding it to them. The equivalent for your setup is the enable/disable toggle in Settings.

---

## 5. Changes requiring your approval

Nothing below has been done.

**Fix what is failing**
1. Change Product Scout weekly and monthly to send by Microsoft 365 instead of Gmail.
2. Test-send from a cloud session, then correct the email note in the order-history watchdog.
3. Fix or remove the revoked send-as address in Gmail (you do this in Gmail settings).

**Reduce exposure**
4. Remove Robinhood from scheduled tasks 4–12.
5. Trim each scheduled task to the connectors its prompt uses.
6. Remove the duplicate "Slack cd890b01" connector.
7. Finish or remove the Stripe connector.

**Resolve conflicts (I need your ruling first)**
8. Nurse vs. medical practitioner: state the rule and its scope.
9. `cpd` auto-publish vs. claims QA: choose the order.
10. Product Scout: one Firecrawl rule for both tasks; move sheet IDs into the skill.

**Cut dead weight**
11. Disable the Sales and Product Management plugins.
12. Disable Superpowers, or limit it to coding projects.
13. Decide on Design, Operations and Productivity plugins.
14. Archive or retire `hash-hermes-setup`.
15. Merge `covenant-blog-post` and `covenant-blog-from-url` into one skill with two modes, or sharpen their descriptions so they don't overlap.
16. Fix the metadata in `weekly-retention-report` and `cpd`.
17. Decide whether `weekly-retention-report` should be scheduled, merged into `vs-cohort-retention`, or retired.

**Scheduled-task tuning**
18. Replace the hourly spreadsheet watcher with a local job, or reduce its frequency and model.
19. Confirm with Carolyn whether the Morning Consult Call Sheet is used.
20. Review model choice on the short-running tasks (6, 7, 8).
21. Delete the stale daylight-saving note in Morning Ops.

**Instructions and housekeeping**
22. Rewrite the master prompt as version 2.0 (I draft, you approve).
23. Archive the 16 completed idle sessions (list on request).
24. Remove `www.youtube.com` from the Default environment's allowed hosts.
25. Build a reusable `/detox` skill of your own from this audit's method, to rerun quarterly on demand. Not a recurring job unless you ask for one.

---

## 6. Backup and rollback plan

| Change | Backup before | Rollback |
|---|---|---|
| Scheduled task prompts and connector lists | Full current prompts and settings for all 13 tasks are captured in this session; I will save them to a file before the first edit | Paste the saved prompt and re-attach connectors |
| Custom skills | Export all 15 skill folders as a dated zip before any edit | Re-upload the saved version |
| Plugins | Record which are enabled | Re-enable in Settings |
| Connectors | Note current status | Reconnect (requires you to sign in again) |
| Master prompt | Keep version 1.0 text in a file | Paste it back |
| Sessions | Archive only, never delete | Unarchive |
| Memory | You export or screenshot it on your Mac before any edit | Restore from the export |

Changes go one group at a time, with a check after each: a test run of the affected task or skill before moving on.

---

## 7. Recommended order of work

1. **Email delivery** (items 1–3). About 20 minutes. You are missing reports today.
2. **Robinhood and connector trimming** (items 4–7). About 30 minutes.
3. **Your three rulings** (items 8–10). About 10 minutes of your time; then I apply them.
4. **Plugins** (items 11–13). About 10 minutes in Settings. Largest drop in clutter for the least effort.
5. **Scheduled-task tuning** (items 18–21). About 45 minutes.
6. **Skill cleanup** (items 14–17). About 1–2 hours, done after a backup zip.
7. **Master prompt 2.0** (item 22). About 30 minutes.
8. **Housekeeping and the reusable skill** (items 23–25). About 1–2 hours.

---

## 8. What I could not inspect

- **The Detox skill download.** The link is in the YouTube description, and YouTube blocks this cloud machine. I have not seen, installed or run it. If you download the file and upload it here, I will inspect it for destructive steps, data transfers and paid services before anything else.
- **Your Mac.** Local connector configuration, local skills, `~/.claude` settings, Claude memory files and launchd jobs.
- **Regular Claude Desktop chats and Cowork sessions.** No tool here lists or archives them.
- **Cowork's locally stored scheduled tasks.**
- **Usage evidence.** I have no record of which skills or connectors you used, or how often, so nothing above is recommended for removal on the basis of "unused." Recommendations rest on relevance, overlap, conflicts and failures.
- **Billing and token usage data.**
- **Connector permission scopes**, beyond what the Gmail bounce and task prompts reveal.
- **Whether `cpd` and `cpd-rewrite` apply the same standard.** I compared headers, not all 100 KB of rules.
- **Project instructions inside claude.ai Projects.**

All brand and compliance rules for CHP, VitaminScript and HHP are untouched, as are existing workflow approvals.
