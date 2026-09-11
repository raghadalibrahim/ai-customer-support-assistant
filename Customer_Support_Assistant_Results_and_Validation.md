├── 1. System Overview
├── 2. Architecture
├── 3. Knowledge Base
├── 4. RAG Validation
├── 5. Conflict Detection
├── 6. Decision & Guardrails
├── 7. API Validation
├── 8. Test Cases
├── 9. Results
├── 10. Limitations
└── 11. Future Improvements

# Customer Support Assistant
## Results and Validation Report

---

## 1. System Overview

### 1.1 Project Description

The Customer Support Assistant is an AI-powered customer support automation
workflow designed to answer customer questions using an approved knowledge
base while applying retrieval-based validation, decision logic, safety
guardrails, and human escalation.

The system is designed to distinguish between:

- Questions that can be safely answered using available knowledge.
- Questions where the knowledge base does not provide sufficient information.
- Sensitive or high-risk cases that require human support intervention.

The system uses Retrieval-Augmented Generation (RAG) to ground AI responses
in the approved NovaShop knowledge base.

### 1.2 Main Objectives

The system aims to:

1. Provide accurate answers based on approved knowledge.
2. Retrieve relevant information from the knowledge base.
3. Avoid answering when sufficient evidence is unavailable.
4. Detect sensitive and high-risk customer cases.
5. Escalate appropriate cases to a human support agent.
6. Return structured API responses for integration with external applications.
7. Provide a confidence/evidence signal based on retrieval quality.
8. Detect conflicts between retrieved knowledge sources.

### 1.3 Technology Stack

| Component | Technology |
|---|---|
| Workflow Automation | n8n |
| AI Model | Google Gemini |
| Embeddings | Google Gemini Embeddings |
| Vector Database | Qdrant |
| Knowledge Base | Markdown documents |
| API Interface | HTTP Webhook |
| API Testing | Postman |
| Human Escalation | Telegram |
| Containerization | Docker |

---

## 2. System Architecture

The current workflow follows this general processing pipeline:

Customer Request
        ↓
Webhook
        ↓
Input Normalization
        ↓
Knowledge Retrieval
        ↓
RAG / AI Answer Generation
        ↓
Decision & Guardrails
        ↓
   ┌───────────────┐
   │               │
Resolved       Human Escalation
   │               │
   ↓               ↓
Customer        Telegram
Response
   ↓
API Response

The system also includes a retrieval validation layer used to evaluate
retrieved evidence and detect conflicts between knowledge sources.

### 2.1 Main Processing Components

#### Webhook

Receives customer questions through an HTTP POST request.

Example request:

```json
{
  "question": "How long does standard shipping take?"
}

Knowledge Retrieval

Relevant documents are retrieved from the novashop_knowledge Qdrant
collection using vector similarity search.

RAG Answer Generation

The retrieved knowledge is provided to the AI model so that the generated
answer is grounded in the approved knowledge base.

Decision & Guardrails

The generated response is evaluated against system rules.

The decision layer can produce states such as:

resolved
insufficient_knowledge
human_escalation_required
Human Escalation

Cases requiring human intervention are forwarded to a support agent through
Telegram together with relevant case information.

API Response

Resolved requests are returned through the webhook as structured JSON.

Example:

{
  "status": "resolved",
  "response": "Standard shipping takes 3–5 business days. Please note that shipping times are estimates and may be affected by weekends, public holidays, or unexpected carrier delays.",
  "confidence": 0.94,
  "escalated": false
}
## 3. Knowledge Base

### 3.1 Knowledge Base Structure

The system uses an approved internal knowledge base containing eight
Markdown documents covering the main customer support domains.

| File | Domain |
|---|---|
| `company.md` | Company information and support availability |
| `products.md` | Products and product information |
| `shipping.md` | Shipping policies and delivery times |
| `returns-refunds.md` | Returns and refund policies |
| `payments.md` | Payment policies and payment-related safety rules |
| `warranty.md` | Warranty information |
| `account-security.md` | Account access and security |
| `faq.md` | Frequently asked questions |

The knowledge base is stored locally and ingested into the Qdrant vector
database.

### 3.2 Vector Database

The knowledge base is stored in the Qdrant collection:

`novashop_knowledge`

Gemini embeddings are used to convert the knowledge documents into vector
representations. Customer questions are embedded and compared against the
stored vectors to retrieve relevant knowledge.

The current knowledge base contains eight source documents.

### 3.3 Knowledge Grounding Rules

The AI assistant is designed to provide information based only on the
approved knowledge base.

The system specifically avoids inventing:

- Product specifications
- Product availability
- Prices
- Compatibility information
- Exact delivery dates
- Unverified warranty coverage

The knowledge base also defines cases that must be escalated to a human
support agent, including:

- Refund approval requests
- Disputed charges
- Duplicate charges
- Fraud or unauthorized transactions
- Suspected account compromise
- Security incidents
- Warranty cases requiring inspection or exceptions
- Cases where the available knowledge is insufficient

---

## 4. RAG Validation

### 4.1 Retrieval Validation

The RAG pipeline was tested using customer questions covering different
knowledge domains.

For each question, the system retrieves relevant documents from the
`novashop_knowledge` collection and provides the retrieved information to the
AI answer-generation stage.

The retrieval layer also records:

- Retrieved source documents
- Retrieval similarity scores
- Top retrieval score
- A normalized retrieval confidence value

### 4.2 Retrieval Confidence

The current confidence value is derived from the highest retrieval score and
is normalized using the following calculation:

```text
confidence = min(top_retrieval_score / 0.75, 1)
This value represents an evidence/retrieval quality signal, rather than a
calibrated probability that the generated answer is correct.

4.3 Retrieval Test: Shipping Information

Question:

How long does standard shipping take?

The system successfully retrieved the relevant shipping information and
generated a grounded response.

Observed values:

Top retrieval score: 0.7080519
Confidence: 0.94

The resulting customer response correctly stated that standard shipping takes
3–5 business days and included the applicable disclaimer regarding weekends,
public holidays, and carrier delays.

Result: PASS

4.4 Retrieval Test: Unknown Information

Question:

Who is the CEO of NovaShop?

The requested information was not available in the approved knowledge base.

The system generated an insufficient-information response and triggered the
human escalation logic.

Observed values:

Status: insufficient_knowledge
Requires human: true
Confidence: 0
Top retrieval score: 0.7374174

This test demonstrates that a relatively high retrieval similarity score does
not automatically cause the system to treat the answer as valid. The generated
response is also evaluated for insufficient knowledge before the final
decision is made.
Result: PASS

## 5. Conflict Detection

### 5.1 Purpose

The system includes a conflict-detection mechanism to identify cases where
different retrieved knowledge sources provide inconsistent information.

This is important because retrieving multiple relevant documents does not
necessarily mean that all retrieved information is mutually consistent.

### 5.2 Conflict Detection Test

A controlled validation test was performed using the customer support hours
question.

The system compared the relevant information retrieved from:

- `company.md`
- `faq.md`

The test was designed to verify whether the system could detect inconsistent
information between multiple knowledge sources.

The conflict-detection logic successfully identified the inconsistency.

**Result: PASS**

### 5.3 Knowledge Consistency

After validation, the knowledge base was restored to a consistent state.

The approved customer support hours are:

```text
Monday to Friday, 09:00–18:00
The Qdrant collection was subsequently recreated and the corrected knowledge
base was ingested again to ensure that stale conflicting vectors were removed.

6. Decision and Guardrails Validation
6.1 Decision Layer

The system does not rely solely on the AI-generated answer.

A dedicated decision layer evaluates the response together with the retrieved
evidence and the original customer question.

The decision layer can identify:

Resolved requests
Insufficient knowledge
Sensitive or high-risk requests requiring human review
6.2 Human Escalation Rules

The system escalates cases involving:

Unauthorized account access
Suspected account compromise
Fraud
Duplicate charges
Disputed charges
Payment disputes
Security incidents
Requests requiring actions that the AI is not authorized to perform

The system also escalates cases when the knowledge base does not contain
sufficient information to provide a reliable answer.

6.3 Account Security Escalation Test

Question:

Someone accessed my account without authorization. What should I do?

The system correctly identified the request as a sensitive security case.

Observed result:

Status: human_escalation_required
Requires human: true
Confidence: 1
Top retrieval score: 0.7720347

The system also generated a response explaining that suspected account
compromise or unauthorized access must be reviewed by a human support agent.

The case was successfully forwarded to the human escalation path through
Telegram.

Result: PASS

6.4 Insufficient Knowledge Escalation

When the system cannot find sufficient approved information to answer a
question, it does not attempt to invent an answer.

Instead, the response is classified as:

status: insufficient_knowledge
requires_human: true

This provides a controlled fallback mechanism for questions outside the
available knowledge base.

Result: PASS

## 7. API Validation

### 7.1 API Interface

The Customer Support Assistant exposes an HTTP POST endpoint through an n8n
Webhook node.

The API accepts a customer question in JSON format.

Example request:

```json
{
  "question": "How long does standard shipping take?"
}
The request is processed through the complete support pipeline, including
knowledge retrieval, AI response generation, decision logic, and guardrails.

7.2 Successful Resolution Test

A shipping-related request was submitted through Postman to validate the
complete API flow.

Request:

{
  "question": "How long does standard shipping take?"
}

Response:

{
  "status": "resolved",
  "response": "Standard shipping takes 3–5 business days. Please note that shipping times are estimates and may be affected by weekends, public holidays, or unexpected carrier delays.",
  "confidence": 0.94,
  "escalated": false
}
7.3 API Validation Result

The API successfully:

Accepted the HTTP POST request.
Extracted the customer question from the request body.
Retrieved relevant knowledge from Qdrant.
Generated a grounded response using the AI model.
Applied the decision and guardrail logic.
Prepared the final customer response.
Returned a valid structured JSON response to Postman.

Result: PASS

7.4 Response Schema

The API response currently contains the following fields:

Field	Description
status	Final processing state
response	Customer-facing response
confidence	Retrieval/evidence confidence signal
escalated	Indicates whether human escalation is required

The current API therefore provides both the customer-facing answer and the
decision outcome in a machine-readable format.

## 8. Test Cases and Results

### 8.1 Test Overview

The system was validated using multiple test scenarios designed to evaluate
knowledge retrieval, response generation, insufficient-knowledge handling,
security escalation, conflict detection, and API behavior.

The following table summarizes the main validation scenarios.

| Test Case | Purpose | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| Standard Shipping | Validate normal knowledge retrieval and response generation | Correct grounded answer | Correct shipping response returned | PASS |
| Unknown Information | Validate insufficient-knowledge handling | Do not invent an answer; escalate | `insufficient_knowledge`, human escalation | PASS |
| Account Compromise | Validate security escalation | Escalate to human support | `human_escalation_required`, high priority | PASS |
| Knowledge Conflict | Validate conflicting-source detection | Detect inconsistency | Conflict detected between sources | PASS |
| API Response | Validate complete webhook/API flow | Return structured JSON | Valid JSON returned in Postman | PASS |

### 8.2 Test Case 1 — Standard Shipping

**Input:**

```text
How long does standard shipping take?
Expected behavior:

The system should retrieve the shipping policy and provide the correct
delivery timeframe without escalating the request.

Observed result:

Status: resolved
Requires human: false
Confidence: 0.94
Top retrieval score: 0.7080519

The returned answer correctly stated that standard shipping takes 3–5
business days and included the relevant delivery disclaimer.

Result: PASS

8.3 Test Case 2 — Unknown Information

Input:

Who is the CEO of NovaShop?

Expected behavior:

The system should not invent information that is absent from the approved
knowledge base. The case should be treated as insufficient knowledge.

Observed result:

Status: insufficient_knowledge
Requires human: true
Confidence: 0
Top retrieval score: 0.7374174

The system correctly avoided providing an unsupported answer and triggered
the human escalation path.

Result: PASS

8.4 Test Case 3 — Account Compromise

Input:

Someone accessed my account without authorization. What should I do?

Expected behavior:

The request should be classified as a high-risk security case and escalated
to a human support agent.

Observed result:

Status: human_escalation_required
Requires human: true
Confidence: 1
Top retrieval score: 0.7720347

The system generated an appropriate security response and forwarded the case
to the human escalation path through Telegram.

Result: PASS

8.5 Test Case 4 — Knowledge Conflict

Input:

What are NovaShop's customer support hours?

Expected behavior:

The system should compare the retrieved sources and detect inconsistent
support-hour information.

Observed result:

Sources compared:
- company.md
- faq.md

Conflict detected: true

The conflict-detection mechanism successfully identified the inconsistency.

After the test, the knowledge base was restored to the approved consistent
value of Monday–Friday, 09:00–18:00.

Result: PASS

8.6 Test Case 5 — Complete API Flow

Input:

{
  "question": "How long does standard shipping take?"
}

Expected behavior:

The complete workflow should process the request and return a structured API
response.

Observed result:

{
  "status": "resolved",
  "response": "Standard shipping takes 3–5 business days. Please note that shipping times are estimates and may be affected by weekends, public holidays, or unexpected carrier delays.",
  "confidence": 0.94,
  "escalated": false
}

The response was successfully received in Postman.

Result: PASS

8.7 Overall Validation Result

All currently executed validation scenarios completed successfully.

The tests demonstrate that the current system can:

Retrieve relevant knowledge.
Generate grounded customer responses.
Handle questions outside the available knowledge.
Detect sensitive security cases.
Escalate appropriate requests to human support.
Detect conflicting knowledge sources.
Return structured API responses.

Overall current validation status: PASS

## 9. Results Summary

### 9.1 Functional Validation

The current validation results demonstrate that the system successfully
performs the main functions defined for the Customer Support Assistant.

The tested workflow successfully demonstrated:

- Knowledge retrieval using vector similarity search.
- RAG-based response generation.
- Evidence-based response handling.
- Insufficient-knowledge detection.
- Human escalation for sensitive cases.
- Security-related escalation.
- Knowledge conflict detection.
- Telegram-based human notification.
- Structured API response generation.
- End-to-end API testing through Postman.

### 9.2 Validation Status

| Capability | Validation Status |
|---|---|
| Knowledge Retrieval | PASS |
| RAG Response Generation | PASS |
| Insufficient Knowledge Handling | PASS |
| Security Escalation | PASS |
| Human Escalation | PASS |
| Knowledge Conflict Detection | PASS |
| Telegram Notification | PASS |
| API/Webhook Processing | PASS |
| Structured JSON Response | PASS |

The current implementation therefore satisfies the main functional
requirements defined for the validated version of the system.

---

## 10. Limitations

The current validation provides evidence for the tested scenarios, but it
does not represent exhaustive production-level testing.

The current limitations include:

1. The test dataset is relatively small and uses a controlled knowledge base.
2. Retrieval confidence is an evidence-quality signal and is not a calibrated
   probability of answer correctness.
3. The current conflict-detection logic focuses on identified knowledge
   patterns rather than providing a general-purpose semantic contradiction
   engine.
4. Customer-specific information such as live order status and inventory is
   not currently connected to an external operational database.
5. The current system does not perform real refund processing or financial
   actions.
6. Production authentication, authorization, rate limiting, and API security
   hardening have not yet been fully validated.
7. The current tests focus primarily on functional behavior rather than
   large-scale performance, latency, or load testing.

These limitations define the boundary of the current validation results and
should be considered before deploying the system in a production environment.

---

## 11. Future Improvements

The following improvements are planned or suitable for future versions of the
system:

### 11.1 Retrieval Improvements

- Improve document chunking and metadata handling.
- Evaluate retrieval quality across a larger test dataset.
- Introduce more robust retrieval evaluation metrics.
- Investigate hybrid or reranked retrieval.

### 11.2 Decision and Safety Improvements

- Expand the rule-based escalation framework.
- Introduce more comprehensive sensitive-case detection.
- Improve semantic conflict detection across knowledge sources.
- Add stronger validation between retrieved evidence and generated answers.

### 11.3 Production Improvements

- Add API authentication and authorization.
- Add rate limiting and abuse protection.
- Add structured logging and monitoring.
- Add persistent conversation and case tracking.
- Connect the assistant to real customer/order systems where appropriate.

### 11.4 Evaluation Improvements

Future validation should include:

- Larger and more diverse test datasets.
- Adversarial and edge-case testing.
- Retrieval precision and recall evaluation.
- Response faithfulness evaluation.
- Latency measurements.
- Concurrent request and load testing.
- Regression testing after knowledge-base updates.

---

## 12. Conclusion

The current implementation demonstrates a functional AI-powered customer
support workflow that combines Retrieval-Augmented Generation, vector search,
decision logic, safety guardrails, conflict detection, and human escalation.

The validation tests successfully demonstrated the system's ability to:

- Answer supported customer questions using approved knowledge.
- Avoid unsupported answers when information is unavailable.
- Identify sensitive security cases.
- Escalate appropriate cases to human support.
- Detect inconsistencies between knowledge sources.
- Provide structured API responses for external integration.

The current results establish a functional baseline for further development,
testing, and production hardening of the Customer Support Assistant.