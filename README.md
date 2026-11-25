# subscription-bill-auditor-agent
# Autonomous Subscription & Bill Auditor Agent

An agentic AI financial assistant built with **LangGraph**, **Google Gemini**, **Pandas**, and **ChromaDB** that autonomously audits bank statements, detects hidden recurring subscriptions and billing anomalies, schedules `.ics` calendar renewal reminders, and drafts policy-grounded dispute emails behind a **Human-in-the-Loop (HITL)** approval gate.

## Architecture & Control Flow

```text
[User Uploads Bank CSV]
         │
         ▼
┌─────────────────────┐
│ 1. Auditor LLM Node │ ◄─── Decides which Python tools to invoke
└────────┬────────────┘
         │ (Tool Calls)
         ▼
┌─────────────────────┐
│ 2. Tool Execution   │ ───► Pandas: Detects recurring bills, price hikes & duplicates
│    Node             │ ───► ChromaDB: Retrieves merchant refund/cancellation policies
│                     │ ───► Calendar Tool: Generates .ics renewal reminders
└────────┬────────────┘
         │ (Observations returned to LLM)
         ▼
┌─────────────────────┐
│ 3. Draft Action     │ ───► Gemini outputs structured Pydantic dispute email JSON
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ 4. Human Gate       │ ───► LangGraph interrupt() PAUSES execution & saves state
│    (HITL Checkpoint)│      Waits for explicit user approval ("approve" / "reject")
└────────┬────────────┘
         │ (Resumed via Command(resume=...))
         ▼
┌─────────────────────┐
│ 5. Dispatch Node    │ ───► Dispatches email ONLY if approved
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ 6. Interactive Q&A  │ ───► Stateful follow-up chat over spending & savings
└─────────────────────┘
