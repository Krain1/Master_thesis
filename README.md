# Master_thesis
AI Agent simulation of a Marketing Study

This project runs an agent simulation that:

- builds psychological profiles from Excel,
- runs a brand congruence survey (Q9/Q10/Q11 items) with reasoning,
- performs bidding decisions using Playwright (headless browser) on a demo shop,
- applies a learning/feedback loop (matching agents to human training data),
- saves all results back to Google Drive (CSV/XLSX/JSON).
- It’s designed to run in Google Colab with zero local setup.


What’s inside (high level)

- EnhancedMemoryModule: Vector long-term memory (Chroma), short-term memory, episodic memories, chain-of-thought tracking, and a global learning store.
- BrandPersonalityToolOrchestrator: Critical brand personality analysis + “Big Five” scoring and user–brand matching.
- FeedbackToolOrchestrator: Learns from real human training data (Excel) using semantic similarity; generates survey & bidding adjustments.
- PriceResearchToolOrchestrator: Lightweight, selector-based price scraping with fallbacks per brand (Apple / Nike / Levi’s).
- AgentSurveyOrchestrator: Step-by-step reasoning for Q9/Q10/Q11 items, optional brand-personality challenge, confidence scoring.
- AgentBiddingOrchestrator: CoT bidding using cross-brand experiences + learning adjustments, then Playwright places bids in a headless browser.
- Full experiment runner: Processes every agent: ① profile & memory → ② survey → ③ bidding → ④ feedback → ⑤ learning update.
- - - Includes checkpoint saving and branch resume (Cells A–C) if your session stops.

   
Required files in Google Drive
Create these folders in your Drive and put the Excel files there (in online Appendix):

/MyDrive/profiles_survey/All_profiles.xlsx
/MyDrive/profiles_survey/Training_survey_data.xlsx


Expected columns
All_profiles.xlsx should have:

username
age (number)
gender
occupation
income
actual_1, actual_2, actual_3, actual_4
actual_top_strength
ideal_1, ideal_2, ideal_3, ideal_4
ideal_top_strength
Training_survey_data.xlsx should include (examples used by the code’s mapping):
For Nike: nike_actual_consistent, nike_actual_mirror, nike_ideal_consistent, nike_ideal_mirror, nike_affection, nike_love, nike_connection, nike_passion, nike_delight, nike_captivation, and bid Nike
For Apple: same pattern (apple_*, bid Apple)
For Levi’s: same pattern with escaped apostrophe in mapping (e.g., levis_*, bid Levis)

If a column is missing, the code safely skips it.


What the notebook installs
The notebook installs all dependencies itself:

playwright openai nest_asyncio pandas openpyxl jsonschema
langchain langchain-community chromadb sentence-transformers
duckduckgo-search beautifulsoup4 requests langgraph

It also runs:
playwright install --with-deps
This may take a few minutes the first time.


API keys
The code uses an OpenAI-compatible API endpoint:

os.environ["OPENAI_API_KEY"]  = "*****"
os.environ["OPENAI_API_BASE"] = "https://gpt.uni-muenster.de/v1"
model = "Llama-3.3-70B"

In Colab, set your key at runtime before running the main cells:

import os
os.environ["OPENAI_API_KEY"]  = "YOUR_KEY_HERE"
os.environ["OPENAI_API_BASE"] = "https://gpt.uni-muenster.de/v1"  # or your compatible endpoint


Outputs (saved automatically to Drive)
The notebook creates (if missing) and writes to:

/MyDrive/agent_survey_results/
/MyDrive/agent_bidding_results/
/MyDrive/agent_learning_results/

Typical files:

Survey (with learning): agent_survey_results_...csv
Survey (pre-learning): agent_survey_results_pre_learning_...xlsx
Brand analysis impact: agent_brand_analysis_impact_...xlsx
Survey reasoning: agent_survey_reasoning_...csv
Communications (survey/bidding): agent_*_communications_...csv
Bidding results: agent_bidding_results_...csv
Feedback messages: agent_feedback_messages_...csv
Learning summary: agent_learning_summary_...json
You’ll also see final statistics printed in the Colab logs.

IMPORTANT: THE PRE-FEEDBACK BIDS NEED TO GET EXTRACTED FROM THE CONSOLE (CODE TO DO SO IN ONLINE APPENDIX). THE AGENT PLACES ITS LEARNING-ADJUSTED BID ON THE AUCTION WEBSITE.


Brands covered
The default brands are: brands = ["Nike", "Apple", "Levi's"]


Bidding demo site
The Playwright flow visits: https://auction-shop-agent.web.app/

It:

- reads 3 products (names/descriptions),
- computes bids using the agent’s reasoning + learning,
- places the bid values in the site’s form.
Everything runs headless in Colab.


Resuming after Colab interruption (Branches)
If your Colab session restarts, use the branch resume cells:

CELL A: re-setup + load/translate profiles; set start_idx = <first not-yet-translated index>.
CELL B: re-create shared learning memory by merging past checkpoint JSONs you saved earlier (paste their full Drive paths in paths = [...]).
CELL C: re-run the full experiment from where you left off; it continues saving checkpoints every 10 agents.

You’ll see printed summaries confirming how many adjustments/wisdom entries were restored.


Troubleshooting
Drive or file not found:
- Re-run drive.mount('/content/drive').
- Double-check your file paths match the ones above.
Playwright errors or browser timeouts
- First run can take longer due to install --with-deps.
- If Chromium fails, go to Runtime → Restart runtime and re-run the install cell.
HTTP 403 / no prices found
- The price tool has a fallback price per brand; the run continues.
- Some sites block scraping; this is expected, and the code is robust to it.
- > To see the references prices used in the simulation when I conducted it, see "Konsolenausgabe" in the online Appendix or agent communication file.

Missing Python modules
- Run the pip install cell again (it’s at the top of the notebook).
- API errors or model not found
Make sure your OPENAI_API_BASE, OPENAI_API_KEY, and model name are valid for your provider.


Repro steps (quick checklist)
1. Click Open in Colab badge above.
2. Run the first cell → mount Google Drive.
3. Ensure these files exist in Drive:

/MyDrive/profiles_survey/All_profiles.xlsx
/MyDrive/profiles_survey/Training_survey_data.xlsx

4. Set your API key at runtime (see API keys section).
5. Run all cells from top to bottom.
6. Watch progress logs. Check exports in your Drive folders.

Notes for reviewers

The code is written as a Colab notebook (multiple “CELL x” sections).

Uses ChromaDB in-session (ephemeral).

Uses SentenceTransformers for semantic similarity, HuggingFace tokenizer/model for Big Five classifier, LangGraph/LangChain for tool orchestration.

The learning module aggregates differences and applies small step adjustments over time.

The feedback is based on top-3 training data matches per agent (semantic similarity on traits + demographics).
