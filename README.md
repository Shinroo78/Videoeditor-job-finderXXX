# 📹 Video Editor Job Finder — Automated Outreach Pipeline

![Status](https://img.shields.io/badge/status-working%20workflow-brightgreen)
![Type](https://img.shields.io/badge/type-n8n%20automation-blue)
![Platform](https://img.shields.io/badge/platform-n8n-orange)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blueviolet)
![Delivery](https://img.shields.io/badge/delivery-Telegram-2CA5E0)
![License](https://img.shields.io/badge/license-MIT-green)

An **n8n workflow** that scrapes **Facebook Groups**, **Instagram hashtags**, and **YouTube** for video editor job postings, filters out noise and service providers, generates a personalized outreach pitch using **Google Gemini**, and delivers a formatted alert to **Telegram** — automatically, in one run.

> ⚠️ **This workflow uses web scraping.** It is intended for personal use, lead generation, and learning automation. You are responsible for complying with the Terms of Service of every platform it touches.

---

## ⚠️ What this project is and is not

### ✅ What this is
- A **working n8n workflow** you can import and run in minutes.
- A **multi-source lead-generation pipeline** combining three scrapers, one smart filter, one AI agent, and one notification channel.
- **Bilingual** — recognizes both English and Arabic job intent (`looking for`, `hiring`, `مطلوب`, `أبحث عن`).
- **Human-in-the-loop** — it *suggests* a pitch; **you** decide whether to send it.

### ❌ What this is not
- ❌ Not a SaaS product — it runs on **your** n8n, **your** Apify, **your** Gemini, **your** Telegram.
- ❌ Not a spam tool — it never messages clients automatically.
- ❌ Not guaranteed to find every job post — coverage depends on the groups and hashtags you configure.
- ❌ Not compliant-by-default — scraping Facebook/Instagram/YouTube may violate their ToS.

---

## 🎯 Purpose

Freelance video editors miss real client requests every day because those requests are buried inside Facebook groups, Instagram hashtags, and YouTube comment sections — written in mixed languages, posted at random times, and surrounded by noise from other editors promoting their own services.

This workflow exists to solve one problem:

> **Surface real "I need a video editor" posts, filter out the rest, and hand the user a ready-to-send, personalized pitch — in Telegram, within minutes.**

---

## 🚀 Quick start

### 1. Import the workflow
```text
n8n → Workflows → ⋯ menu → Import from File → select Get Client.json
2. Prepare credentials
Credential	Type in n8n	Where to get it
Apify API	
apifyApi
https://console.apify.com/account/integrations
Google Gemini	googlePalmApi	https://makersuite.google.com/app/apikey
Telegram Bot	telegramApi	https://t.me/BotFather
3. Set your Telegram chatId
Open the Send a text message node and set chatId to either:

text
123456789           ← your personal numeric user ID
@your_channel       ← a channel/group the bot was added to
4. Enable the sources you want
Only Facebook is enabled by default. Instagram and YouTube ship disabled to keep the first run cheap.

Node	Default	How to enable
FB	✅ Enabled	—
IG	❌ Disabled	Right-click → Enable
YT	❌ Disabled	Right-click → Enable
5. Run it
Click Execute Workflow. Within a few minutes, Telegram will start receiving matched job alerts.

Requirements: n8n 1.x+, Apify account with credits, Google AI Studio key, Telegram bot.

🔄 Workflow Overview
text
Manual Trigger
      │
 ┌────┴────┬─────────┐
 ▼         ▼         ▼
FB        IG        YT
 │         │         │
 └────┬────┴────┬────┘
      ▼         ▼
     Merge
       │
       ▼
 JS Filter (recency + intent + spam rules)
       │
       ▼
   AI Agent (Gemini)
       │
       ▼
    Telegram
🧩 Feature breakdown by node
🎛 When clicking 'Execute workflow' — Manual Trigger
Single entry point.

Fans out to FB, IG, and YT simultaneously.

📘 FB — Facebook Groups Scraper (Apify)
Actor: apify/facebook-groups-scraper (2chN8UQcH1CfxLRNE)

Default groups (replace with your own): videoeditor0, ineedavideoeditor, mohsenfouad, plus 8 more niches

viewOption: CHRONOLOGICAL → newest posts first

resultsLimit: 5 per group per run

maxTotalChargeUsd: 1 — hard spend ceiling per execution

📸 IG — Instagram Hashtag Scraper (Apify, disabled)
Actor: apify/instagram-scraper (shu8hvrXbJbY3Eb9W)

Hashtags: hiringvideoeditor, videoeditorhiring, lookingforvideoeditor, مطلوب_مونتير

resultsLimit: 20

onlyPostsNewerThan must be updated manually before re-enabling

▶️ YT — YouTube Scraper (Apify, disabled)
Actor: streamers/youtube-scraper (h7sDV53CddomktSi5)

Queries:

"looking for video editor"

"need a video editor"

"hiring video editor"

"مطلوب مونتير"

maxResultsShorts: 10

🔀 Merge — 3-input fan-in
Concatenates outputs from FB (index 0), YT (index 1), IG (index 2)

Normalized shape: { text, url, authorName, groupTitle, createdAt, ... }

🧹 Code in JavaScript — Smart Filter
javascript
✅ Keep when:
   - total text length > 15 chars
   - createdAt / publishedAt within 3 days
   - contains client-intent keywords:
       looking for, hiring, need an editor, مطلوب, أبحث عن

❌ Drop when:
   - #editorforhire, "i am a video editor", "hire me",
     "my services", or title starts with "need a video editor?"
Adjust the window with:

javascript
const THREE_DAYS_IN_MS = 3 * 24 * 60 * 60 * 1000;
🤖 AI Agent — Gemini-powered pitch writer
Model: Google Gemini Chat Model

System prompt injects the post's author, group, URL, and original text:

text
- اسم العميل: {{ $json.user?.name || $json.authorName || 'غير محدد' }}
- المصدر/المجموعة: {{ $json.groupTitle || $json.channelName || 'غير محدد' }}
- رابط المنشور: {{ $json.url }}
- نص المنشور الأصلي:
{{ $json.text || $json.description || $json.title }}
Output format (fixed Telegram template):

text
🚀 فرصة عمل جديدة!

👤 العميل: ...
👥 المصدر: ...
🔗 الرابط: ...

📝 الرد المقترح (Pitch):
[short, tailored, in the language of the request]
The pitch is written in the same language as the post.

📨 Send a text message — Telegram delivery
Sends {{ $json.output }} to the configured chatId

One message per qualified job post

🏗️ Architecture
text
┌───────────────────────────────────────────────────────────┐
│  Trigger — Manual                                         │
└─────────────────────────┬─────────────────────────────────┘
                          │
     ┌────────────────────┼────────────────────┐
     ▼                    ▼                    ▼
┌─────────┐         ┌─────────┐          ┌─────────┐
│   FB    │         │   IG    │          │   YT    │
│ Apify   │         │ Apify   │          │ Apify   │
│ (on)    │         │ (off)   │          │ (off)   │
└────┬────┘         └────┬────┘          └────┬────┘
     │                   │                    │
     └──────────┬────────┴────────────────────┘
                ▼
         ┌──────────────┐
         │    Merge     │   (3-input fan-in)
         └──────┬───────┘
                ▼
         ┌──────────────┐
         │  JS Filter   │   (recency + intent + spam rules)
         └──────┬───────┘
                ▼
         ┌──────────────┐
         │  AI Agent    │◄── Google Gemini
         └──────┬───────┘
                ▼
         ┌──────────────┐
         │  Telegram    │
         └──────────────┘
💰 Cost considerations
Service	Estimated cost per run
Apify — Facebook Groups	~$0.50–$1.00
Apify — Instagram	~$0.50–$1.50 (20 posts)
Apify — YouTube	~$0.50
Gemini API	Free tier available
Telegram	Free
Each Apify node has maxTotalChargeUsd: 1 set by default.

📱 Sample Telegram Output
text
🚀 فرصة عمل جديدة!

👤 العميل: Ahmed Karim
👥 المصدر: Video Editors & Creators
🔗 الرابط: https://facebook.com/groups/.../posts/...

📝 الرد المقترح (Pitch):
مرحباً أحمد، لاحظت أنك تبحث عن مونتير فيديو.
أعمل على مونتاج محتوى اليوتيوب والسوشيال ميديا
منذ 5 سنوات، ولدي أعمال مشابهة يمكنني مشاركتها.
هل تود أن نتناقش في التفاصيل؟
🔧 Customization
Modify search queries
Edit customBody inside any Apify node:

json
{
  "searchQueries": [
    "\"looking for video editor\"",
    "\"need a video editor\"",
    "\"hiring video editor\"",
    "\"مطلوب مونتير\""
  ]
}
Adjust time window
javascript
const THREE_DAYS_IN_MS = 3 * 24 * 60 * 60 * 1000;
Modify AI prompt
Edit the AI Agent's system message to change pitch tone, language, or format.

Add scheduling
Replace the Manual Trigger with a Schedule Trigger (e.g., every 6 hours).

⚠️ Known limitations
No deduplication. Same post may be processed twice within the 3-day window.

No scheduled execution. Add a Schedule Trigger for automation.

Fixed Instagram date filter. onlyPostsNewerThan must be updated manually.

No retry / backoff on Apify timeouts or Telegram 429s.

Prompt is opinionated toward Arabic-first outreach.

Legal surface area is yours. Scraping may violate platform ToS.

🐛 Troubleshooting
Issue	Solution
No Facebook results	Verify group URLs are public and accessible
AI Agent fails	Check Gemini API key and quota
Telegram not sending	Verify bot is added to the chat/channel
Merge node errors	Ensure all scraper outputs have compatible data
Apify actor timeout	Reduce resultsLimit or split runs
📁 File structure
text
.
├── Get Client.json    # n8n workflow export
└── README.md          # This file
🤝 Contributing
Fork the repository

Create a feature branch

Submit a pull request

📄 License
MIT License. See LICENSE file for details.

Disclaimer: This workflow uses web scraping, which may violate platform Terms of Service. Use responsibly and at your own risk.

📞 Support
Open a GitHub issue

Check n8n community forums
