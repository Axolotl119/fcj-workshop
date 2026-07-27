---
title: "FC Community Day"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# FC Community Day

* **Event Name:** FC Community Day - GenAI, Cloud Infrastructure & Voice AI
* **Date:** 04/07/2026
* **Format:** Offline (26th & 36th Floor) & Online (YouTube Livestream)
* **Event Recording:**

<iframe 
  width="100%" 
  height="400" 
  src="https://www.youtube.com/embed/G8-WlI7f6dE?start=271" 
  title="FC Community Day" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
  allowfullscreen>
</iframe>

---

### COMPREHENSIVE EVENT REPORT

#### 1. Career Journey & Cloud Agentic Platform
* **Speakers:** Mr. Steve Tran (*Founder & CEO - Cloud Thinker*, Former SA at AWS).
* **Key Takeaways & Content:**
  * **Career Growth:** Started at 19 managing physical servers in a contact center, overcame multiple exam failures to achieve AWS certifications, and advanced to Solution Architect at AWS Vietnam in just 4 years.
  * **Impact of AI:** The rise of AI coding tools has shifted the hiring market, leading companies to prioritize Senior engineers who can effectively leverage AI.
  * **Cloud Thinker's Solution:** Solves infrastructure complexity and technical debt using an Agentic Platform:
    * *Incident Response:* Analyzes logs and conducts investigations in minutes instead of hours.
    * *Code Review & Security:* Performs automated code reviews and pentesting before production release.
    * *FinOps:* Automatically optimizes AWS resource costs.
  * **Single vs. Multi-Agent Architecture:** Multi-Agent architectures reduce context windows and manage Role-Based Access Control (RBAC) more efficiently by delegating tasks to Specialist Agents.

#### 2. Voice AI for Enterprise
* **Speakers:** Mr. Hieu Nghi (*Renova Cloud*), Mr. Kiet (*AWS Student Community*), Mr. Trung Do (*Founder & CEO - R AI*).
* **Key Takeaways & Content:**
  * **3-Stage Architecture for Vietnamese:** As Vietnamese is a low-resource language, the most optimal pipeline consists of: 
    Audio Input $\rightarrow$ STT $\rightarrow$ LLM $\rightarrow$ TTS $\rightarrow$ Audio Output.
  * **Real-world Banking Implementations:**
    * Content control and action execution (*Tool Calling*) for tasks like card locking and account inquiries.
    * Natural barge-in handling, pause detection, and gender identification for proper polite honorifics.
    * Training models with 10–20% regional accent data to improve recognition accuracy.

#### 3. DevOps AI Agent
* **Speakers:** Ms. Bao & Mr. Nguyen Nguyen (*Cloud Engineers - Cloud Kinetics*).
* **Key Takeaways & Content:**
  * Solves fragmented telemetry issues that prolong Mean Time to Recovery (MTTR).
  * **4-Step Automated Workflow:** Trigger $\rightarrow$ Root Cause Investigation $\rightarrow$ Mitigation Plan $\rightarrow$ System Improvement.
  * Utilizes *Agent Space* to build infrastructure topology, reducing incident resolution time by 75–77%.

#### 4. Recruitment & HR Management with Amazon Q (Quick)
* **Speakers:** Mr. Truong "Wynn" & Ms. Minh Anh (*Noventic*).
* **Key Takeaways & Content:**
  * Overcomes manual CV screening limitations and eliminates data leak risks associated with public AI tools.
  * Custom *Skills* creation (e.g., *HR Talent Review Assistant*) integrating data from OneDrive, Sharepoint, S3, Jira, etc.
  * Automated JD generation, CV data extraction via OCR, candidate evaluation against technical benchmarks, and interactive dashboard reporting.

#### 5. Private Security Configuration for Amazon Q via MCP Server
* **Speakers:** Mr. Toan Nguyen (*AWS Security Builder*) & Mr. Hieu Nghi (*Renova Cloud*).
* **Key Takeaways & Content:**
  * Eliminates exposure risks and DoS/MITM attack vectors associated with public internet endpoints.
  * Private Connection Architecture: Places the MCP Server in a Private Subnet within a VPC, establishing a VPC Connection/Interface Endpoint so all traffic stays strictly inside the AWS internal network.
  * Integrates TLS encryption via ALB, AWS Cognito authentication, and internal DNS resolution using Route 53 Resolver (Estimated infrastructure maintenance cost: $250 - $350/month).

---

![Event Photo](/images/4-EventParticipated/4.1-event1/27-6-2026.png)