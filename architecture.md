# 🏗️ Jiddu Architecture & Setup Guide

This document breaks down the deep technical architecture of the Jiddu Voice AI project.

## 1. Vapi (Voice Orchestrator)
Vapi is configured to handle the telephony and LLM processing.
- **Provider:** Twilio (for phone numbers and outbound calling)
- **STT Provider:** Azure (configured for `te-IN` and `en-IN` to understand Telugu and Indian English accents seamlessly)
- **LLM:** gpt-4o-mini / gemini-1.5-flash (for cost-effective, low-latency reasoning)
- **System Prompt:** Instructs the persona "Jiddu" to ask 3 specific questions:
  1. Did you take your morning/evening tablets?
  2. What is your Blood Pressure today?
  3. What is your Blood Sugar today?

## 2. Webhook & Make.com (The Glue)
When the call ends, Vapi sends an `end-of-call-report` webhook payload. Make.com receives this payload via a Custom Webhook module.
- **Data Parsing:** The Make.com scenario uses a text parser or LLM parser to extract the `BP`, `Sugar`, and `Tablets_Status` from the call transcript.
- **Error Handling:** If the elderly person didn't pick up the call, or the data was not clearly stated, Make.com routes the flow to a "Failed/Missed" log.

## 3. Notion (The Database)
The extracted data is pushed into a Notion Database using the Notion API (`Create a Database Item` module in Make).
- **Properties:**
  - Date (Date)
  - Time (Select: Morning/Evening)
  - BP (Text)
  - Sugar (Number)
  - Tablets Taken? (Checkbox)
  - Raw Transcript (Text - for auditing)

## 🔄 How to Replicate
1. Create a Vapi account and configure an outbound assistant.
2. In the Vapi dashboard, set the STT provider to Azure and paste your Azure Speech credentials.
3. Create a Notion Database with the properties listed above, and generate an Internal Integration Token.
4. Create a Make.com scenario: `Custom Webhook` -> `OpenAI (Parse Data)` -> `Notion (Create Item)`.
5. Copy the Make.com Webhook URL and paste it into the `Server URL` field in your Vapi assistant settings.
