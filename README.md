```markdown
# AURA - Adverse Event Unit Relational Analyzer

Multi-agent SLM system for real-time clinical trial safety monitoring with zero context loss through structured memory layers.

## The Problem

In clinical trials, adverse event reports can take hours or days to process:
- Safety coordinators manually read reports
- Teams cross-reference protocols manually  
- Risk assessment requires checking patterns across patients manually
- Recommendations are drafted manually

Result: Critical safety signals get delayed. A headache reported on Day 1 might not trigger an alert until Day 9. Lives are at stake.

## What is AURA?

AURA solves the context shredding problem in clinical trial safety monitoring using three specialized Small Language Models (each under 3B parameters) that collaborate in real-time:

- **Foundation Layer** (EventExtractor-SLM): Extracts structured data from adverse event reports
- **Strategic Layer** (RiskAnalyzer-SLM): Scores risk, identifies patterns, references prior events
- **Synthesis Layer** (RecommendationSLM): Generates safety alerts and protocol recommendations

Total processing time: under 500ms

Every recommendation traces back through the risk analysis to the original extracted events with full provenance.

## Why Small Language Models?

Instead of one massive LLM doing everything:
- Faster inference - Each specialized model is optimized for one task
- Lower cost - Models under 3B parameters run efficiently on Trainium
- Better accuracy - Fine-tuned for specific medical reasoning tasks
- Distributed reasoning - Models collaborate, don't duplicate work

This architecture proves that orchestrated small models outperform monolithic large models for real-time critical systems.

## Tech Stack

- **Infrastructure**: AWS Trainium (for SLM fine-tuning and inference)
- **SLM Base Models**: Llama 3.2 1B / Phi-3 / TBD based on availability
- **Frontend**: React + Vite + Tailwind CSS
- **Backend**: Express.js REST API
- **Database**: MongoDB Atlas (structured context store)
- **Architecture**: Sequential SLM orchestration with real-time UI updates

## Quick Start

### Prerequisites

- AWS Trainium access
- Node.js 18+
- MongoDB Atlas account
- Base SLM models available

### Setup

1. Install dependencies:
```bash
npm install
```

1. Configure environment (.env):

```bash
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/
VITE_AWS_TRAINIUM_ENDPOINT=your_trainium_endpoint
VITE_DB_NAME=clinical_safety_monitor
```

1. Fine-tune the three SLMs (on Trainium):

```bash
# See /fine-tuning/README.md for detailed instructions
python fine_tune_event_extractor.py
python fine_tune_risk_analyzer.py  
python fine_tune_recommendation.py
```

1. Test connections:

```bash
node test-connections.js
```

1. Run the system:

```bash
npm run dev:all
```

Opens:

- Backend API: <http://localhost:3000>
- Frontend: <http://localhost:5173>

## Architecture

```
┌─────────────────────┐
│   React Frontend    │ ← Real-time polling (500ms)
└──────────┬──────────┘
           │ HTTP
┌──────────▼──────────┐
│    Express API      │ ← Sequential SLM execution
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   MongoDB Atlas     │ ← Single document with geological layers
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  AWS Trainium       │ ← 3 fine-tuned SLMs (under 3B params each)
│                     │
│ EventExtractor-SLM  │ → Extraction
│ RiskAnalyzer-SLM    │ → Classification  
│ RecommendationSLM   │ → Generation
└─────────────────────┘
```

## Execution Flow

1. User uploads adverse event report (text or PDF)
1. Backend API runs SLMs sequentially:

- EventExtractor-SLM writes context_layers.foundation (structured event data)
- RiskAnalyzer-SLM reads foundation, writes context_layers.strategic (risk scores, patterns)
- RecommendationSLM reads both, writes context_layers.synthesis (safety alerts, protocol changes)

1. Frontend polls MongoDB via API, displays live updates with color-coded layers
1. Result: Complete safety assessment in under 500ms with full reasoning trace

## MongoDB Document Structure

```javascript
{
  case_id: "trial_adverse_event_123",
  patient_id: "PT-2847",
  status: "complete", // "ready" | "extracting" | "analyzing" | "recommending" | "complete"
  
  // Input data
  adverse_event_report: {
    raw_text: "Patient reports severe headache, confusion, elevated BP 180/110...",
    report_date: "2024-10-11T14:30:00Z",
    reporter: "Site Coordinator"
  },
  
  // The three geological layers
  context_layers: {
    
    // FOUNDATION LAYER - Built by EventExtractor-SLM
    foundation: {
      extracted_events: [
        {
          id: "event_001",
          symptom: "severe headache",
          severity: "high",
          onset: "acute",
          temporal_pattern: "sudden onset at 2PM",
          vital_signs: { bp: "180/110", hr: 92 }
        },
        {
          id: "event_002",
          symptom: "confusion",
          severity: "moderate",
          cognitive_domain: "orientation"
        }
      ],
      patient_context: {
        age: 34,
        baseline_bp: "120/80",
        concurrent_medications: ["Study Drug XYZ-001"],
        relevant_history: []
      },
      extraction_complete: true,
      processed_at: "2024-10-11T14:30:12Z"
    },
    
    // STRATEGIC LAYER - Built by RiskAnalyzer-SLM
    strategic: {
      risk_assessments: [
        {
          id: "risk_001",
          assessed_event_ids: ["event_001", "event_002"], // References foundation layer
          risk_category: "cerebrovascular",
          severity_score: 0.89,
          urgency: "immediate",
          pattern_match: {
            similar_events_in_trial: 2,
            known_drug_profile_match: true,
            reference_safety_signals: ["ARIA-E", "hypertensive crisis"]
          },
          causal_likelihood: 0.78,
          reasoning: "Acute neurological symptoms with severe hypertension suggest potential cerebrovascular event..."
        }
      ],
      overall_case_severity: 0.89,
      requires_escalation: true,
      analysis_complete: true,
      processed_at: "2024-10-11T14:30:14Z"
    },
    
    // SYNTHESIS LAYER - Built by RecommendationSLM
    synthesis: {
      safety_alerts: [
        {
          alert_id: "alert_001",
          alert_level: "CRITICAL",
          title: "Potential Cerebrovascular Event - Immediate Action Required",
          source_risk_ids: ["risk_001"], // Links back to strategic layer
          source_event_ids: ["event_001", "event_002"], // Links back to foundation
          immediate_actions: [
            "Patient requires immediate medical evaluation",
            "Hold study drug administration",
            "Perform neurological assessment and brain imaging"
          ],
          reasoning_chain: "High severity neurological symptoms (event_001, event_002) + elevated risk score (0.89) + pattern match with 2 similar cases = critical safety signal"
        }
      ],
      protocol_recommendations: [
        {
          recommendation_id: "rec_001",
          recommendation_type: "protocol_amendment",
          description: "Enhanced neurological monitoring for all active participants",
          justification: "Pattern of 3 cerebrovascular events suggests systematic safety signal",
          regulatory_action: "Notify Data Safety Monitoring Board within 24 hours"
        }
      ],
      synthesis_complete: true,
      processed_at: "2024-10-11T14:30:16Z"
    }
  },
  
  // Tracks how SLMs passed context
  agent_handoffs: [
    {
      from_agent: "EventExtractor-SLM",
      to_agent: "RiskAnalyzer-SLM",
      message: "Extracted 2 adverse events with neurological presentation",
      context_snapshot: { events_count: 2, severity_flags: 1 },
      timestamp: "2024-10-11T14:30:12Z"
    },
    {
      from_agent: "RiskAnalyzer-SLM",
      to_agent: "RecommendationSLM",
      message: "High-risk cerebrovascular pattern identified, severity 0.89",
      context_snapshot: { risk_assessments: 1, escalation_required: true },
      timestamp: "2024-10-11T14:30:14Z"
    }
  ],
  
  // Audit trail
  agent_actions: [
    {
      agent_name: "EventExtractor-SLM",
      action_type: "extracted_events",
      details: "Identified 2 adverse events from report",
      inference_time_ms: 180,
      timestamp: "2024-10-11T14:30:12Z"
    },
    {
      agent_name: "RiskAnalyzer-SLM",
      action_type: "risk_assessment",
      details: "Scored severity at 0.89, flagged for escalation",
      inference_time_ms: 150,
      timestamp: "2024-10-11T14:30:14Z"
    },
    {
      agent_name: "RecommendationSLM",
      action_type: "generated_recommendations",
      details: "Created 1 critical alert and 1 protocol recommendation",
      inference_time_ms: 170,
      timestamp: "2024-10-11T14:30:16Z"
    }
  ],
  
  total_processing_time_ms: 480
}
```

## API Endpoints

```
GET  /api/health                    # Check MongoDB + Trainium status
GET  /api/context/:caseId           # Fetch context document
POST /api/execute/:caseId           # Execute SLM workflow
POST /api/upload                    # Upload adverse event report (text/PDF)
POST /api/reset/:caseId             # Clear context
GET  /api/stats                     # Performance metrics (inference times)
```

## Fine-Tuning the SLMs

See /fine-tuning/README.md for detailed instructions. Overview:

### EventExtractor-SLM

**Task**: Named Entity Recognition + Structured Extraction  
**Training data**: Adverse event reports to JSON structure  
**Optimization**: Speed (target under 200ms inference)

### RiskAnalyzer-SLM

**Task**: Classification + Pattern Recognition  
**Training data**: Structured events to risk scores and categories  
**Optimization**: Medical reasoning accuracy

### RecommendationSLM

**Task**: Text Generation (regulatory-compliant)  
**Training data**: Risk assessments to safety alerts and recommendations  
**Optimization**: Actionable output quality

## Demo Scenario

### Input

Adverse event report:

```
"Patient PT-2847, Day 42 of study. Reports severe headache onset 2PM, 
confusion, difficulty speaking. Vital signs: BP 180/110, HR 92. 
Patient on study drug XYZ-001, no other medications."
```

### Output (generated in under 500ms)

- **Foundation Layer**: 3 extracted events (headache, confusion, speech difficulty), structured vitals
- **Strategic Layer**: Risk score 0.89, cerebrovascular category, pattern match with 2 prior cases
- **Synthesis Layer**: CRITICAL alert, immediate medical evaluation required, DSMB notification recommended

### Evidence Chain

Click alert, traces to risk assessment, traces to extracted events

## Team Roles

- **ML Engineer**: Fine-tune 3 SLMs on Trainium, optimize inference
- **Backend Engineer**: Build agent orchestration, MongoDB integration, API
- **Frontend Engineer**: Real-time visualization, Evidence Chain UI

## Project Structure

```
aura/
├── README.md
├── package.json
├── .env
├── src/
│   ├── frontend/
│   │   ├── components/
│   │   │   ├── AgentPanel.jsx
│   │   │   ├── ContextEvolution.jsx
│   │   │   ├── LayerView.jsx
│   │   │   └── EvidenceChain.jsx
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── backend/
│   │   ├── server.js
│   │   ├── routes/
│   │   │   └── api.js
│   │   ├── agents/
│   │   │   ├── EventExtractorSLM.js
│   │   │   ├── RiskAnalyzerSLM.js
│   │   │   └── RecommendationSLM.js
│   │   └── services/
│   │       ├── mongoClient.js
│   │       └── trainiumClient.js
│   └── utils/
│       └── mockData.js
├── fine-tuning/
│   ├── README.md
│   ├── fine_tune_event_extractor.py
│   ├── fine_tune_risk_analyzer.py
│   ├── fine_tune_recommendation.py
│   └── training_data/
│       ├── event_extraction_examples.jsonl
│       ├── risk_analysis_examples.jsonl
│       └── recommendation_examples.jsonl
└── test-connections.js
```

## License

Open Source - MIT

```

```
