---
title: "Event 3: FCAJ x Agentic AI Build Week 2026"
date: 2026-07-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# FCAJ x Agentic AI Build Week: Show Up. Build. Pitch. WIN!

* **Event Name:** FCAJ x Agentic AI Build Week 2026
* **Date:** 25/07/2026
* **Format:** Offline & Online (YouTube Livestream)
* **Organizer:** AWS Study Group / First Cloud AI Journey (FCAJ) in collaboration with JI Fund
* **Event Recording:**

<iframe 
  width="100%" 
  height="400" 
  src="https://www.youtube.com/embed/hz32VBrvW7M?start=3" 
  title="FCAJ x Agentic AI Build Week 2026" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
  allowfullscreen>
</iframe>

---

### COMPREHENSIVE EVENT REPORT

#### I. OPENING KEYNOTE & VISION
* **Special Guests:** Mr. Nguyen Gia Hung (*Head of Solution Architect - AWS Vietnam*) and Mr. Joseph Marazzota (*Head of Technology - AWS ASEAN*).
* **Core Takeaways from Mr. Joseph Marazzota:**
  * **Mental Model Shift:** 20 years ago, software release cycles took quarters or weeks. Today, AI Agents enable continuous deployments and automated updates by the minute.
  * **Human-in-the-Loop Concept:** Amazon operates over 1 million robots in fulfillment centers, but they are useless without human guidance. Humans remain the ultimate decision-makers in configuring, orchestrating, and verifying AI suggestions.
  * **Lifelong Learning:** Challenge traditional approaches, experiment constantly through hackathons, and shape the industry's future over the next 2–3 years.

---

#### II. PROJECT PRESENTATIONS & DEMOS

#### 1. One Team – KFC AI Voice Order Agent (1st Place - AWS Track)
* **Team:** 5 multinational members (USA, India, Vietnam) formed at the event.
* **Real-world Problem:** Learning from McDonald's AI drive-thru failure (where natural conversation issues and hallucinations caused accidental orders like 100 chicken nuggets).
* **Solution & Architecture:**
  * **No App Switching:** Enables customers to place orders directly within Zalo/WhatsApp via Webhook Adapters without downloading apps or creating accounts.
  * **AWS Bedrock Agent Core:** Utilizes Memory capabilities to retain user order history over several weeks.
  * **Data Scraping:** Uses Tiny Fish to scrape real-time menu and promotional data from KFC's website into AWS databases.
* **Cost & Performance:** 
  * Operating cost: **$0.006 per order**.
  * Saves **75% on Bedrock costs** and **60% on infrastructure costs** (~**$88/month** for 500 orders/day).
  * End-to-End latency: **3 to 5 seconds**.

#### 2. Signal Scout – Multi-Agent Competitive Intelligence Canvas (2nd Place - AWS Track)
* **Team:** FPT University Students (Hoang Hieu, Quoc Hao, Minh Quan, Cong Minh, Di Khiem, Tuan Luc).
* **Real-world Problem:** Helps corporate strategy teams gather scattered competitor signals (financial reports, investor slides) to forecast ROI when adopting competitor strategies.
* **Solution & Architecture:**
  * **Frontend & Security:** React Dashboard hosted on AWS Amplify, secured by AWS WAF and Amazon Cognito authentication.
  * **Multi-Agent Flow:** A *Supervisor Agent* orchestrates specialized Sub-agents:
    * *Crawler Sub-agent:* Uses **Apify** (static pages) and **Tiny Fish** (dynamic pages / login wall bypass).
    * *Analysis Sub-agent:* Processes data via **Bedrock Guardrails** and monitors evaluation scores using **Langfuse**.
  * **Self-Healing Loop:** Automatically retries up to 2 times if data scores are low before tagging for human review.
* **Cost:** ~**$35/month** (Medium usage) up to **$130/month** (Maximum usage).

#### 3. S.A. Plan – SA Professional AI-Native Assistant
* **Team:** Long, Vi, Phat, An, Nghia.
* **Real-world Problem:** Relieves pressure on Solution Architects (SAs) when clients demand urgent architecture diagrams and cost estimations overnight.
* **Solution & Workflow:**
  * Accepts natural language prompts or corporate policy documents.
  * Analyzes ETL flows and automatically renders standardized architecture diagrams on **Draw.io**.
  * Generates cost estimation reports and exports modular **Terraform/CloudFormation IaC** code.
  * **Output Validation:** Enforces Blacklist/Whitelist checks at output gates to prevent unauthorized services (e.g., App Runner or Elastic Beanstalk).

#### 4. Team 3K – Sheper (Smart Human Flow & Queue Management)
* **Team:** Nguyen Huy, Huynh An Khuong, Hoang Minh Duc, Ngo Khoi, Dang Nguyen Phuoc Loc.
* **Real-world Problem:** Monitors and alleviates crowd congestion at airport check-in gates, supermarkets, or major event venues.
* **Solution & Architecture:**
  * **Video Streaming:** Streams live video feeds from cameras/mobile devices via **Amazon Kinesis Video Streams**.
  * **Computer Vision:** Runs **YOLOv8 (Small)** and **ByteTrack** on **AWS Fargate** for real-time person detection, ID tracking, and zone-based crowd counting.
  * **Storage & AI Agent:** Stores density metrics in DynamoDB/S3. An AI Agent connected via **Amazon Bedrock** (querying OpenAI/Claude) analyzes congestion, sends alerts, and suggests staff allocation.

#### 5. Six Pillars – Adaptive Workflow Engine (Anti-Money Laundering - AML)
* **Team:** Viet, Nguyen Van Linh, Nguyen, Minh Nhat, Huyen.
* **Real-world Problem:** Reduces high False Positive rates (up to 90–95%) in traditional banking AML systems, which waste $20–$25 per manual review.
* **3-Layer Architecture:**
  * *Layer 1 (Fast Detection):* Uses **Amazon Kinesis**, **AWS Lambda**, and an **XGBoost** model on SageMaker to quickly filter out 90–95% of normal transactions at minimal cost.
  * *Layer 2 (Core Investigation - Multi-Agent):* Uses **AWS Step Functions** to orchestrate 3 Sub-agents (*KYC Agent*, *Money Flow Agent*, *Sanction Check Agent*) querying evidence from **OpenSearch Vector DB**. Employs a secondary LLM for double-checking to prevent Hallucinations.
  * *Layer 3 (Case Management & Enterprise Security):* Features a human-in-the-loop dashboard for final decision-making (*Hold, Dismiss, Escalate*). Fully secured with **AWS KMS, Secret Manager, GuardDuty, Security Hub, CloudWatch**, and **AWS X-Ray**.

---

![Event Photo](/images/4-EventParticipated/4.3-event3/25-7-2026.png)