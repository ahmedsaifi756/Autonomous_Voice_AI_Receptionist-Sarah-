# 🎙️ Autonomous AI Voice Receptionist (n8n + Groq + ElevenLabs)

A 24/7 **Audio-to-Audio AI Voice Receptionist** built in **n8n** that listens to callers, checks live **Google Calendar** availability, remembers multi-turn conversations by phone number, and enforces a strict JavaScript guardrail so it never confirms an appointment unless the event is actually booked.

---

## 🎯 The Problem It Solves
Most AI voice demos fail in real business use because the LLM hallucinates—telling a customer *"Your appointment is confirmed!"* when nothing was actually written to the calendar. 

This workflow fixes that by separating **conversation** (LLM) from **booking verification** (deterministic JavaScript + Google Calendar API).

---

## 🏗️ 7-Node Straight-Line Architecture

```text
[1. Webhook (Audio Intake)]
        │
        ▼
[2. Groq Whisper STT] ──► Transcribes incoming audio (.mp3/.wav) in milliseconds
        │
        ▼
[3. Prepare Session]  ──► Scopes session_id to caller_phone & cleans transcript
        │
        ▼
[4. AI Agent (LLM)]   ──► Uses Simple Memory + Dual Calendar Tools (Get_Free_Slots & Book_Consultation)
        │
        ▼
[5. JS Guardrail]     ──► Sanitizes JSON & blocks "Confirmed" speech unless calendar_event_id exists
        │
        ▼
[6. ElevenLabs TTS]   ──► Converts verified reply into natural human speech (MP3)
        │
        ▼
[7. Respond Webhook]  ──► Streams MP3 audio response directly back to the caller

✨ Key Engineering Highlights
Zero Premature Bookings: On Turn 1 ("Is anything free tomorrow?"), the agent calls Get_Free_Slots, proposes open times, and keeps customer_confirmed: false. It only calls Book_Consultation on Turn 2 after explicit user confirmation.

Caller-Scoped Memory: Uses Simple Memory keyed by caller_phone so multi-turn conversations stay isolated per caller without cross-talk.

Deterministic Spoken Guardrail (JavaScript): Before generating audio, a custom JS gatekeeper parses the LLM's structured output and enforces strict rules—blocking confirmation messages unless Google Calendar returns a real calendar_event_id.

End-to-End Audio Pipeline: Accepts raw audio via multipart/form-data and returns playable audio/mpeg binary output.

🛠️ Tech Stack
Orchestration: n8n (Self-Hosted / Cloud)

Speech-to-Text (STT): Groq API (whisper-large-v3-turbo)

LLM & Tool Calling: Gemini / Llama-3.3-70b + n8n AI Agent & Simple Memory

Calendar Integration: Google Calendar API (Get Many & Create Event)

Validation Layer: Native JavaScript (n8n Code Node)

Text-to-Speech (TTS): ElevenLabs API (eleven_turbo_v2_5)
