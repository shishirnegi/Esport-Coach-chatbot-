PUBG Mobile / BGMI Tactical Coach — AI Chatbot

A memory-enabled conversational AI coach for PUBG Mobile / BGMI players, built on Google's Gemini API with an interactive chat UI. The bot covers zone rotations, map IQ, weapon/attachment meta, utility usage, vehicle strategy, role-specific KPIs, team scouting methodology, and mental game coaching — grounded in a structured knowledge base rather than generic advice.

Overview

This project started as a coaching resource for scrim/tournament players, then became a working chatbot as part of an LLM & Agentic AI learning roadmap (Module 2: Prompting & APIs). It demonstrates a system-prompt-driven assistant with a domain-specific knowledge base, an interactive chat interface, and conversational memory across turns.

Features
8-module tactical knowledge base covering:
Zone & Rotation Theory
Map-Specific IQ (Erangel, Miramar, Sanhok, Vikendi, Livik, Nusa, Rondo)
Weapon & Attachment Meta
Utility & Throwables
Vehicle Rotations & Terrain Decisions
Role-Specific Playbooks & KPIs (IGL, Entry, Support, Flanker, Scout, Flex)
Team Scouting Methodology
Mental Game (tilt management, morale, team identity)
Risk-vs-reward reasoning behind every recommendation, not just flat answers
Skill-level-aware responses — adjusts explanation depth based on the player's stated experience
Conversation memory — maintains context across multiple turns in a single session
Interactive chat UI built with Panel, runnable directly in a notebook (Colab/Jupyter)
Tech Stack
Component	Tool
LLM	Google Gemini (via google-genai SDK)
Chat UI	Panel + Bokeh
Runtime	Python 3.13, Jupyter/Google Colab
Project Structure
├── notebook.ipynb          # Main notebook — model calls, chat logic, UI
├── system_prompt.md        # Role definition + knowledge base (Part A + Part B)
└── README.md
Setup & Installation
Install dependencies:
bash
   pip install google-genai panel
Get a Gemini API key from Google AI Studio.
Set your API key (don't hardcode it — use an environment variable or Colab secret):
python
   import os
   API_KEY = os.environ.get("GEMINI_API_KEY")  # or use google.colab.userdata in Colab
Initialize the client with retry handling (see Known Issues below for why this matters):
python
   from google import genai
   from google.genai import types

   retry_options = types.HttpRetryOptions(
       attempts=5,
       initial_delay=2.0,
       max_delay=64.0,
       http_status_codes=[408, 429, 500, 502, 503, 504]
   )

   client = genai.Client(
       api_key=API_KEY,
       http_options=types.HttpOptions(retry_options=retry_options)
   )
Usage
Run all notebook cells from the top, in order, once.
The chat panel will appear at the bottom of the notebook.
Type a message (e.g., "I want to be an entry fragger") and click Chat!.
The bot will ask diagnostic questions about skill level and goals before giving tailored advice.
Known Issues & Troubleshooting

These are real issues hit during development — documented here so they don't need re-diagnosing:

503 ServerError — "This model is currently experiencing high demand"

This is a transient issue on Google's side, not a bug in the code. The google-genai SDK retries automatically a few times by default, but sustained demand spikes can outlast that window. Fix: wrap the generation call in your own retry loop with a longer backoff (see the HttpRetryOptions setup above, or a manual try/except retry around the specific generate_content call).

Duplicate responses (same message answered 2x, 3x, etc.)

Cause: Re-running the cell that creates the chat button (pn.bind(...) / button_conversation) without a full kernel restart attaches an additional click handler each time, on top of the old one(s). N re-runs = N responses per click. Fix: Restart the runtime completely (Colab: Runtime → Restart session; Jupyter: Kernel → Restart) and run every cell top-to-bottom exactly once. Editing other cells (like the response-generation function) afterward is safe and does not require a restart — only re-running the button/pn.bind cell itself causes this.

BokehUserWarning: reference already known '<uuid>'

Same root cause as the duplicate-response issue — a leftover widget reference from a previous run that wasn't cleared. Restarting the session (above) resolves it. To prevent it from recurring, add this to the top of the cell that builds the chat display:

python
from IPython.display import clear_output
clear_output(wait=True)
Roadmap
 Migrate to pn.chat.ChatInterface for more robust chat state handling (reduces the duplicate-handler risk class entirely)
 Add RAG over the knowledge base for more precise retrieval on specific queries (Module 3 of the learning roadmap)
 Persist conversation memory across sessions
 Add verified, sourced team-scouting reports as a dedicated module
Notes on Accuracy

Weapon tier lists, map mechanics, and removed-weapon lists reflect the meta at the time of writing and should be verified against the current game patch before being treated as authoritative — PUBG Mobile/BGMI balance changes frequently. Team/roster information in any scouting module should likewise be verified before being presented as fact, since rosters change often.

Author

Built by Shishir Negi

