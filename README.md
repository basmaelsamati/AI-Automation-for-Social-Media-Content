# AI Automation for Social Media Content Posting

An n8n workflow that automatically generates, reviews, and publishes social media posts (with AI-generated images) for a Medicine Reminder Android app.

## Overview

This workflow runs on a daily schedule, uses an AI agent to write a unique social media post, generates a matching photorealistic image, sends both to the owner on Telegram for approval, and — once approved — stores the finalized post in a data table ready for publishing to the connected social accounts.

## Features

- **Automated content generation** — an AI agent (via Alibaba Cloud / Qwen) writes a fresh, on-brand post every day covering topics like medication reminders, caregiver tips, app features, and general health tips.
- **AI image generation** — for every post, a photorealistic, text-free "cinematic scene" image is generated with Qwen-Image to match the topic (no logos, captions, or on-image text).
- **Human-in-the-loop review** — before anything goes live, the post + image are sent to a Telegram bot with inline buttons:
  - ✅ Approve
  - ❌ Reject
  - ✂️ Make it shorter
  - 🧑‍💼 Make it more professional
  - ✨ Make it more engaging
  - 🔄 Change the topic
- **AI-powered revisions** — if the owner requests a change, a second AI agent pulls the original post from the data table and rewrites only what was asked for, then re-sends it for another round of approval.
- **State tracking** — every post is stored in an n8n Data Table with a `review_id`, status (`pending` / `approved`), text, hashtags, topic, and generated image reference, so nothing is published twice and the full history is auditable.

## Workflow Architecture

```
Schedule Trigger (daily)
   └─▶ AI Agent (writes post: text + hashtags + topic)
         └─▶ Insert row (Data Table, status = pending)
               └─▶ Generate Image (Qwen-Image)
                     └─▶ Send photo + caption to Telegram
                           └─▶ Send approval buttons

Telegram Trigger (button clicks)
   └─▶ Parse callback data (review_id + action)
         └─▶ Get row from Data Table
               └─▶ Switch on action:
                     ├─ Approve        → Update row (status = approved) → Publish
                     ├─ Reject         → (no further action)
                     └─ Edit request   → AI Agent (revise post using feedback)
                                           └─▶ Update row + send revised post for review
```

## Tech Stack

| Component | Tool |
|---|---|
| Automation platform | [n8n](https://n8n.io) |
| Text generation | Alibaba Cloud / Qwen (via n8n LangChain Agent node) |
| Image generation | Qwen-Image (`qwen-image-3.0`) |
| Approval interface | Telegram Bot API |
| Data storage | n8n Data Table |
| Publishing | Facebook Graph API *(Instagram, X, and LinkedIn integrations planned)* |

## Setup

1. **Import the workflow** into your n8n instance (`Workflows → Import from File`).
2. **Credentials** — add and connect:
   - Telegram Bot API (for approval messages and button callbacks)
   - Alibaba Cloud API (Qwen chat model + Qwen-Image)
   - Facebook Graph API (page access token, for publishing)
3. **Data Table** — create a data table with the following columns and link it in every relevant node:
   - `review_id` (string)
   - `message_id` (string)
   - `post_text` (string)
   - `hashtags` (string)
   - `topic` (string)
   - `image` (string)
   - `statue` (string) — post status: `pending` / `approved`
4. **Schedule** — set the `Schedule Trigger` node to your preferred daily posting time.
5. **Telegram chat ID** — replace the `chatId` values in the Telegram nodes with your own chat ID.
6. **Activate** the workflow.
