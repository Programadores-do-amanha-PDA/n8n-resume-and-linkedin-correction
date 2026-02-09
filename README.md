# N8N Resume and LinkedIn Profile Correction Automation

> An automated N8N workflow system designed to analyze and provide structured feedback on student resumes and LinkedIn profiles for the "Programadores do Amanhã" (Programmers of Tomorrow) program. The system uses AI-powered analysis to ensure students meet professional standards and best practices for tech career applications.

## 📋 Overview

This project implements an intelligent automation system that:

- **Processes student applications in batches** from Google Sheets
- **Analyzes resumes** against institutional guidelines using AI
- **Scrapes and evaluates LinkedIn profiles** for completeness and professionalism
- **Provides detailed, structured feedback** on both resumes and LinkedIn profiles
- **Updates tracking spreadsheets** with results and status

The system is built with n8n (a workflow automation tool) and integrates with Google Sheets, AI models (Groq), and LinkedIn scraping services (Apify).

## 🏗️ Architecture

### Main Workflow: `workflow.json`

**Name:** `[Automatização] Simulador PS`

This is the orchestrator workflow that:

1. Reads student data from Google Sheets (resumes URLs and LinkedIn profiles)
2. Filters and validates entries
3. Loops through items in batches to avoid rate limits
4. Calls three sub-workflows sequentially:
   - Resume Correction
   - LinkedIn Profile Data Fetch
   - LinkedIn Profile Correction
5. Handles errors gracefully and updates status in Google Sheets
6. Implements wait/throttling mechanisms to respect API limits

**Key Components:**

- **Split in Batches nodes**: Processes items in manageable groups
- **Filter nodes**: Validates data before processing
- **Execute Workflow nodes**: Calls sub-workflows
- **Google Sheets nodes**: Reads input data and writes results
- **Wait nodes**: Implements rate limiting (30 seconds \* run index)
- **Error handling**: Continues execution on errors and logs issues

### Sub-Workflows

#### 1. `resume-correction.json`

**Name:** `[PS] Resume Correction`

**Purpose:** Analyzes student resumes using AI against institutional guidelines.

**Process:**

1. Receives resume URL or text
2. Extracts text content from PDF
3. Sends to AI model (OpenRouter) with structured prompt
4. AI evaluates 9 key sections:
   - Header (name, contact, location, links)
   - Objective
   - Summary (3-5 lines)
   - Education (highlighting "Programadores do Amanhã" training)
   - Projects (with tech stack and links)
   - Professional Experience
   - Technical Skills
   - Courses
   - Final Review (grammar, formatting, length)
5. Returns structured JSON feedback with status (✅/⚠️) and detailed comments
   - AI Model: Groq (compound-mini model)

#### 2. `linkedin-get-profiles.json`

**Name:** `[PS] Linkedin Fetch Profiles`

**Purpose:** Scrapes LinkedIn profile data using Apify.

**Process:**

1. Receives array of LinkedIn profile URLs
2. Calls Apify Actor: "LinkedIn Profile Scraper + Email"
3. Extracts profile data (no email version, $4 per 1k profiles)
4. Returns structured profile information for analysis

**Integration:** Apify API

#### 3. `linkedin-correction.json`

**Name:** `[PS] Linkedin Correction`

**Purpose:** Analyzes LinkedIn profile data using AI.

**Process:**

1. Receives scraped LinkedIn profile data
2. Sends to AI model (OpenRouter) with structured prompt
3. AI evaluates 11 key sections:
   - Custom URL (personalized, no random numbers)
   - Headline (area + tech stack, professional)
   - Contact Information
   - Summary (first person, 3+ paragraphs)
   - Experience (action verbs, quantified results)
   - Education (complete data, PDA-specific format)
   - Licenses & Certificates
   - Volunteering (optional but recommended)
   - Skills (hard + soft skills balance)
   - Projects & Publications
   - Languages
4. Sends to AI model (OpenRouter) All images for visual analysis (profile photo, cover image) with separate prompt and schema
5. Returns structured JSON feedback with status and detailed comments
   - AI Model: Groq (compound-mini model)

## 📁 Prompts System

The `/prompts` folder contains specialized AI prompts and JSON schemas:

### Resume Prompts

- **`resume-text-input.md`**: System prompt for resume analysis
  - Defines the AI role as a Career Specialist and Tech Recruiter
  - Contains detailed validation rules for each resume section
  - Specifies output format requirements

- **`resume-schema.json`**: JSON schema for structured resume feedback
  - Defines the exact output structure the AI must follow
  - Includes status enum values (✅ Tudo certo / ⚠️ warnings)
  - Ensures consistent, parseable feedback

### LinkedIn Prompts

- **`system-text-input.md`**: System prompt for text-based LinkedIn analysis
  - Validates profile structure and content
  - Includes specific rules for PDA (Programadores do Amanhã) course formatting
  - Checks professional presentation standards

- **`system-text-schema.json`**: JSON schema for LinkedIn text feedback

- **`system-images-input.md`**: System prompt for visual LinkedIn analysis
  - Evaluates profile photo and cover image
  - Checks professionalism, quality, and appropriateness

- **`system-images-schema.json`**: JSON schema for LinkedIn image feedback

## 💻 Prerequisites

Before running this project, ensure you have:

- **n8n** (self-hosted or cloud): Version 1.0+ with workflow execution capabilities
- **Google Sheets API** credentials configured in n8n
- **Groq API** key for AI model access
- **Apify API** account and credits for LinkedIn scraping
- Access to a **Google Spreadsheet** with the following structure:
  - Column: `Currículo:` (Resume URL)
  - Column: `Link LinkedIn:` (LinkedIn profile URL)
  - Column: `ID` (Unique identifier)
  - Additional columns for status tracking

**Supported Platform:** Linux, macOS, Windows (via n8n)

## 🚀 Installation

### 1. Set up n8n

If you don't have n8n installed:

**Using npm:**

```bash
npm install n8n -g
```

**Using Docker:**

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### 2. Clone this repository

```bash
git clone <repository-url>
cd n8n-resume-and-linkedin-correction
```

### 3. Import workflows into n8n

1. Access your n8n instance (default: `http://localhost:5678`)
2. Go to **Workflows** → **Import from File**
3. Import in this order:
   - `sub-workflows/resume-correction.json`
   - `sub-workflows/linkedin-get-profiles.json`
   - `sub-workflows/linkedin-correction.json`
   - `workflow.json` (main workflow)

### 4. Configure credentials

For each workflow, configure the following credentials:

- **Google Sheets API**: Create OAuth2 credentials or Service Account
- **Groq API**: Add your Groq API key
- **Apify API**: Add your Apify API token

### 5. Update Google Sheets references

In the main workflow (`workflow.json`), update the following nodes with your spreadsheet ID:

- `Get row(s) in sheet`
- `Append row in correction sheet`
- `Update row status to finished`
- All error status update nodes

## ☕ Using the System

### Running the Main Workflow

1. Open the main workflow: `[Automatização] Simulador PS`
2. Ensure your Google Sheet has data in the expected format
3. Click **"Execute Workflow"** or configure a trigger (manual, schedule, webhook)

### Workflow Execution Flow

```text
┌─────────────────────────────┐
│  Read data from Sheets      │
└──────────┬──────────────────┘
           │
┌──────────▼──────────────────┐
│  Filter valid entries       │
└──────────┬──────────────────┘
           │
┌──────────▼──────────────────┐
│  Loop over items (batches)  │
└──────────┬──────────────────┘
           │
┌──────────▼──────────────────┐
│  1. Resume Correction       │
│     (AI Analysis)           │
└──────────┬──────────────────┘
           │
┌──────────▼──────────────────┐
│  2. Fetch LinkedIn Data     │
│     (Apify Scraping)        │
└──────────┬──────────────────┘
           │
┌──────────▼──────────────────┐
│  3. LinkedIn Correction     │
│     (AI Analysis)           │
└──────────┬──────────────────┘
           │
┌──────────▼──────────────────┐
│  Update Sheets with Results │
└─────────────────────────────┘
```

### Output Structure

Each analysis returns structured JSON feedback:

**Resume Feedback Example:**

```json
{
  "sections": {
    "header": {
      "status": ["✅ Tudo certo"],
      "feedback": "All contact information is complete and professional."
    },
    "summary": {
      "status": ["⚠️ Ultrapassa 5 linhas"],
      "feedback": "Summary is too long (7 lines). Reduce to 3-5 lines."
    }
  },
  "general_comments": "Overall structure is good, minor adjustments needed."
}
```

**LinkedIn Feedback Example:**

```json
{
  "sections": {
    "custom_url": {
      "status": "✅ De acordo com as orientações",
      "feedback": "URL is personalized: /in/john-doe"
    },
    "headline": {
      "status": "⚠️ Precisa de ajustes",
      "feedback": "Add tech stack to headline. Example: 'Full Stack Developer | React | Node.js'"
    }
  },
  "general_comments": "Profile shows good structure, enhance technical details."
}
```

### Monitoring and Error Handling

- **Wait Nodes**: Built-in rate limiting prevents API throttling
- **Error Outputs**: Failures are logged but don't stop the entire workflow
- **Status Tracking**: Google Sheets updated with: `finished`, `error`, or specific error messages
- **Retry Mechanism**: Configure retry on fail for Google Sheets operations

## 🛠️ Customization

### Modifying AI Prompts

Edit the files in `/prompts` to customize evaluation criteria:

- Change validation rules
- Add new sections to evaluate
- Modify status options
- Adjust feedback guidelines

### Adjusting Batch Sizes

In `Loop Over Items` nodes, modify the `batchSize` parameter to process more/fewer items at once.

### Changing AI Models

In sub-workflows, replace the Groq node with other AI providers:

- OpenAI
- Anthropic Claude
- Local LLM via Ollama

Update model parameters in the `lmChatGroq` nodes.

## 📊 Performance Considerations

- **Rate Limiting**: 30-second delays between batches prevent API throttling
- **Batch Processing**: Items processed in groups for efficiency
- **Error Isolation**: Errors in one item don't affect others
- **Cost**: Apify scraping costs ~$4 per 1,000 LinkedIn profiles
- **AI Tokens**: Resume analysis typically uses 2,000-3,000 tokens per document

## 📫 Contributing

To contribute to this project:

1. Fork this repository
2. Create a branch: `git checkout -b feature/your-feature-name`
3. Make your changes and commit: `git commit -m 'Add new validation rule for resume skills'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Create a pull request

## 📝 License

This project is part of the "Programadores do Amanhã" initiative. See the [LICENSE](LICENSE.md) file for details.

---

Built with ❤️ for Programadores do Amanhã
