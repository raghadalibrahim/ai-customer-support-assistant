# Customer Support Assistant — Project Summary

## Overview

The Customer Support Assistant is an AI-powered customer support automation platform built with n8n, Google Gemini, Qdrant, and Telegram.

The system combines Retrieval-Augmented Generation (RAG), structured decision logic, confidence evaluation, policy-based guardrails, conflict detection, and human escalation to provide reliable automated support while reducing the risk of unsupported or unsafe AI responses.

The assistant is designed to answer customer questions using an approved knowledge base and escalate cases when the available evidence is insufficient, conflicting, sensitive, or requires human judgment.

---

## Core Architecture

```text
Customer
   |
   v
Webhook
   |
   v
Input Normalization
   |
   v
Knowledge Retrieval
   |
   v
Conflict Detection + Confidence Evaluation
   |
   +--------------------+
   |                    |
   | Conflict           | No Conflict
   v                    v
Human Escalation     RAG Question Answering
                           |
                           v
                    Decision & Guardrails
                           |
                    +------+------+
                    |             |
                    | Escalate    | Resolve
                    v             v
              Human Support   Customer Response
Technology Stack
Component	Technology
Automation Platform	n8n
Large Language Model	Google Gemini
Embeddings	Google Gemini Embeddings
Vector Database	Qdrant
Knowledge Format	Markdown
Human Escalation	Telegram
API Testing	Postman
Containerization	Docker
Network Communication	Webhook / HTTP
Key Features
1. Retrieval-Augmented Generation

The assistant retrieves relevant information from the NovaShop knowledge base before generating an answer.

The model is instructed to answer only from the retrieved context and not rely on unsupported general knowledge.

This reduces hallucination and keeps customer responses aligned with approved company information.

2. Vector Search

Knowledge documents are converted into embeddings and stored in a Qdrant collection.

The current collection is:

novashop_knowledge

The knowledge base contains eight approved documents covering:

Account and security
Company information
Frequently asked questions
Payments
Products
Returns and refunds
Shipping
Warranty
3. Confidence Evaluation

The system evaluates retrieval quality using the highest available retrieval score.

The current confidence calculation is:

confidence = min(top_retrieval_score / 0.75, 1)

The resulting value is included in the API response and is also used by the decision layer.

4. Knowledge Conflict Detection

The workflow can detect conflicting information between retrieved knowledge sources.

For example, if two approved documents provide different customer-support hours, the system identifies the conflict instead of silently selecting one source.

Conflicting information results in human escalation.

5. Decision and Guardrails

The final decision is not based solely on the language model.

A separate decision layer evaluates:

Retrieved evidence
Confidence
Knowledge sufficiency
Customer request
Security-related conditions
Financial-risk conditions
Escalation rules

This separates answer generation from operational decision-making.

6. Human Escalation

The system escalates cases that require human judgment.

Examples include:

Unauthorized transactions
Suspected fraud
Duplicate charges
Payment disputes
Account compromise
Security incidents
Refund approval requests
Policy exceptions
Insufficient knowledge
Conflicting knowledge sources

Escalation notifications are sent through Telegram.

7. Sensitive Information Protection

The assistant must never request or store:

Passwords
Authentication codes
Complete payment card numbers
CVV codes
Other sensitive payment credentials

The system also prevents the AI from making financial decisions or approving refunds.

API Response

For successfully resolved requests, the API returns a structured response similar to:

{
  "status": "resolved",
  "response": "Standard shipping takes 3–5 business days.",
  "confidence": 0.94,
  "escalated": false
}

For cases requiring human intervention:

{
  "status": "human_escalation_required",
  "response": "Your request has been forwarded to a support agent.",
  "confidence": 1,
  "escalated": true
}
Workflows

The project contains two n8n workflows.

Knowledge Ingestion

Responsible for:

Reading the Markdown knowledge base
Extracting document content
Adding source metadata
Creating embeddings
Storing documents in Qdrant

Workflow file:

workflows/knowledge-ingestion.json
Customer Support

Responsible for:

Receiving customer requests
Retrieving relevant knowledge
Detecting conflicts
Generating evidence-based answers
Applying decision rules and guardrails
Escalating high-risk or unsupported cases
Returning a structured API response

Workflow file:

workflows/customer-support.json
Validation

The system was tested against multiple scenarios, including:

Scenario	Expected Behavior
Standard shipping question	Resolve
Unknown company information	Escalate
Account compromise	Escalate
Duplicate charge	Escalate
Support-hours conflict	Detect conflict and escalate
Normal policy question	Resolve using retrieved evidence

The workflow was also tested through Postman using the customer-support webhook.

Current Version

Version: 1.0

The current version represents a validated baseline focused on:

RAG-based support
Evidence-grounded responses
Retrieval confidence
Knowledge conflict detection
Decision rules
Safety guardrails
Human escalation
Structured API responses
Known Limitations

The current implementation does not yet provide:

Authentication for the public webhook
Production-scale load testing
Comprehensive automated evaluation datasets
Full observability and monitoring
Direct integration with live order or payment systems
Persistent customer conversation history
Production deployment infrastructure

These are considered future improvements rather than requirements for the current validated version.

Project Goal

The primary goal is to demonstrate how an AI customer-support workflow can combine generative AI with retrieval, deterministic rules, confidence evaluation, and human oversight.

The project prioritizes reliability and controlled automation rather than allowing the language model to make unrestricted support decisions.
