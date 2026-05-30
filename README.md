# Cybersec Research Agent

![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)
![AWS Bedrock](https://img.shields.io/badge/AWS_Bedrock-DeepSeek_v3.2-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)
![Status](https://img.shields.io/badge/Status-Active-brightgreen.svg)

An AI-powered research pipeline that automates cybersecurity tool discovery, feature extraction, and comparison matrix generation.

## Overview

Cybersec Research Agent discovers subdomains within 19 cybersecurity domains, identifies relevant enterprise and open-source tools, extracts differentiating features, and generates structured Excel comparison matrices.

## AI Pipeline

```mermaid
flowchart LR
    subgraph M1["M1"]
        direction TB
        D1[("Domain")] --> WS1["Web Search"]
        WS1 --> L1["LLM + Consensus"]
        L1 --> SD[("Subdomains")]
    end

    subgraph M2["M2"]
        direction TB
        SD2[("Subdomain")] --> WS2["Web Search"]
        WS2 --> L2["LLM"]
        L2 --> T[("Tools")]
    end

    subgraph M3["M3"]
        direction TB
        T3[("Tools")] --> WS3["Web Search"]
        WS3 --> L3["LLM"]
        L3 --> F[("Features")]
    end

    subgraph M4["M4"]
        direction TB
        F4[("Features")] --> L4["Parallel LLMs"]
        L4 --> SF[("Sub-features")]
    end

    subgraph M5["M5"]
        direction TB
        IN["Tools + Sub-features"] --> L5["Batch LLM"]
        L5 --> MX[("Matrix")]
    end

    M1 --> M2 --> M3 --> M4 --> M5
    M5 --> EXCEL[("Excel Export")]
```

## Architecture

```mermaid
flowchart LR
    subgraph CLI["Entry Point"]
        MAIN["main.py"]
    end

    subgraph ORCH["Orchestrator"]
        GRAPH["graph.py"]
    end

    subgraph AGENTS["Agents"]
        A1["discovery.py"]
        A2["tool_discovery.py"]
        A3["feature_discovery.py"]
        A4["subfeature_discovery.py"]
        A5["matrix_population.py"]
    end

    subgraph LLM["LLM Layer"]
        BEDROCK["bedrock.py"]
    end

    subgraph TOOLS["External Tools"]
        TAVILY["Tavily"]
        BRIGHT["BrightData"]
        FIRE["Firecrawl"]
    end

    subgraph DATA["Data Layer"]
        DB[("SQLite")]
        EXCEL[("Excel")]
    end

    MAIN --> CLI_MODE{"Mode?"}
    CLI_MODE -->|"Streamlit UI"| TUI["Streamlit Web App"]
    CLI_MODE -->|"CLI"| GRAPH

    GRAPH --> A1 & A2 & A3 & A4 & A5
    A1 & A2 & A3 & A4 & A5 --> BEDROCK
    A1 & A2 & A3 --> TOOLS
    TOOLS --> WEB[("Web")]
    A5 --> EXCEL
    AGENTS --> DB
```

## Prerequisites

| Service | Purpose | Required |
|---------|---------|----------|
| AWS Bedrock | LLM inference (DeepSeek v3.2) | Yes |
| Tavily API | Primary web search | Yes |
| BrightData API | Fallback SERP | Optional |
| Firecrawl API | Web scraping | Optional |

## Installation

```bash
# Clone the repository
git clone <repo-url>
cd cybersec-research-agent

# Install dependencies (using uv)
uv sync

# Or using pip
pip install -r requirements.txt
```

## Configuration

Create a `.env` file from the example:

```bash
cp .env.example .env
```

| Variable | Description | Default |
|----------|-------------|---------|
| `AWS_REGION` | AWS region for Bedrock | `us-east-1` |
| `BEDROCK_MODEL_ID` | Model identifier | `deepseek.v3.2` |
| `TAVILY_API_KEY` | Tavily search API key | Required |
| `BRIGHTDATA_API_KEY` | BrightData SERP API key | Optional |
| `FIRECRAWL_API_KEY` | Firecrawl scraping API key | Optional |
| `DB_PATH` | SQLite database path | `data/agent.db` |
| `EXCEL_OUTPUT_PATH` | Output Excel file | `output/cybersec_matrix.xlsx` |
| `MAX_WORKERS` | Parallel subdomain pipelines | `3` |
| `LOG_DIR` | Log directory | `logs/` |
| `LOG_DISPLAY_LINES` | Lines to show in log panel | `100` |

## Usage

### Interactive Mode (Streamlit Web UI)

```bash
python main.py
# or explicitly
python main.py --mode streamlit
```

The Streamlit web interface provides:
- **Domain Explorer** - Navigate and manage cybersecurity domains/subdomains
- **Active Pipelines** - Monitor running research pipelines in real-time
- **Logs Panel** - View session logs with auto-refresh
- **Documentation** - Built-in README viewer with interactive Mermaid diagrams

#### Streamlit UI Features

| Feature | Description |
|---------|-------------|
| Domain Tree | Collapsible treeview with checkboxes for bulk operations |
| Detail Panel | Master-detail view for domains and subdomains |
| Pipeline Progress | Live progress bars with stage indicators (M2-M5) |
| Bulk Actions | Run, export, or clear multiple subdomains |
| Documentation Modal | View README with rendered Mermaid diagrams |

### Discover Subdomains

```bash
python main.py --mode discover --domain "Network Security"
```

### Process Single Subdomain

```bash
python main.py --mode single --domain "Network Security" --subdomain "Firewall Management"
```

### Batch Processing

```bash
python main.py --mode batch --domains "Network Security" "Cloud Security" "DevSecOps"
```

## Supported Domains

The agent covers 19 cybersecurity domains:

| | | | |
|---|---|---|---|
| Network Security | Application Security | Cloud Security | Endpoint Security |
| DevSecOps | Identity & Access Management | GRC | Security Operations (SOC) |
| Threat Intelligence | Malware Analysis | Incident Response | OT/ICS Security |
| Mobile Security | AI Security | Cryptography | Information Security |
| Cyber Defense | Digital Forensics | Offensive Security | |

## Output

The pipeline generates:

- **`data/agent.db`** - SQLite database with all discovered entities
- **`output/cybersec_matrix.xlsx`** - Formatted Excel workbook containing:
  - One sheet per subdomain with tool-feature matrices
  - Support levels: ✔ (Full), Partial, ✘ (None)
  - Summary and legend sheets

## License

MIT License
