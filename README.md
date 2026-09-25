# \# n8n-automation-portfolio

# 

# A collection of n8n workflows built while learning AI agent development — covering triggers, AI Agents with tools, memory, multi-agent coordination, human-in-the-loop patterns, batch processing, and error handling.

# 

# Built as a practice track alongside a Udemy course on RAG agents and n8n automation, as preparation for AI Agent Developer roles.

# 

# \## Tech stack

# 

# \- \*\*n8n\*\* (self-hosted, Community Edition)

# \- \*\*Google Gemini API\*\* (Chat Model — free tier, no credit card required)

# \- \*\*wttr.in\*\* — free weather API (no key required)

# \- \*\*Open-Meteo\*\* — free geocoding + weather API (no key required)

# 

# \## Workflows

# 

# | # | Name | What it demonstrates |

# |---|------|----------------------|

# | 01 | Webhook Age Calculator | Webhook trigger + basic data processing |

# | 02a | AI Agent Weather (Single Tool) | AI Agent calling one external API as a Tool |

# | 02b | AI Agent Weather (Two Tools) | Chaining two independent APIs (geocoding → weather) via Agent tool calls |

# | 03 | AI Agent Memory | Session-based memory (Simple Memory node) across multiple messages |

# | 04 | Multi-Agent Coordinator (Weather + Age) | A coordinator agent delegating to two specialist sub-agents |

# | 05 | Sequential Agent Human-in-the-Loop | Lead capture flow with explicit user confirmation before saving data |

# | 06 | Batch Product Description Generator | Code node + Basic LLM Chain to generate marketing copy for multiple items in one run |

# | 07 | Resilient Weather Agent (Error Handling) | HTTP-level error handling — branches on the HTTP Request node's error output (500 response) instead of relying on the LLM to detect and report failures |

# 

# Each workflow's exported JSON is in \[`workflows/`](./workflows).

# 

# \## Key learnings

# 

# \- \*\*Don't rely on the LLM to detect failures it wasn't designed to detect.\*\* In task 07, an AI Agent correctly recognized an invalid city name and replied sensibly — but that was luck, not a guarantee. Moving the error handling to the HTTP Request node's error output makes the behavior deterministic instead of dependent on the model's judgment call.

# \- \*\*HTTP-level vs. data-level failure.\*\* Not every failed lookup comes back as a bad status code — some APIs return `200 OK` with an empty or missing field instead. Always check which kind of failure you're dealing with before deciding where to branch.

# \- \*\*Explicit tool descriptions matter.\*\* An AI Agent with tools attached won't necessarily call them without being told when and why — a clear System Prompt naming each tool and its purpose makes tool use reliable rather than optional.

# \- \*\*Session memory needs an explicit Memory node.\*\* Without one, each message to an AI Agent is stateless by default.

# 

# \## Setup

# 

# ```bash

# git clone https://github.com/lwa4tester-bot/n8n-automation-portfolio.git

# ```

# 

# Import any workflow JSON from `workflows/` into a local n8n instance (`npx n8n` or Docker). Each workflow needs its own credentials configured (Google Gemini API key at minimum); no other secrets are required since the external APIs used (wttr.in, Open-Meteo) are key-free.

