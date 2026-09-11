# Security Policy

## Overview

Security is an important consideration for the Customer Support Assistant.

The project uses AI, vector search, automation workflows, and external integrations. The repository is therefore designed to keep credentials, private configuration, and customer data separate from the public source code.

This document describes the security practices and limitations of the current project.

---

## Supported Version

The current validated version is:

```text
1.0

Security-related issues affecting the current version are welcome and should be reported responsibly.

Reporting a Security Vulnerability

If you discover a security vulnerability in this project, please do not disclose sensitive technical details publicly through GitHub Issues or other public channels.

Instead, contact the project author privately through the contact method associated with the repository or GitHub profile.

When reporting a vulnerability, include:

A clear description of the issue
The affected component or workflow
Steps required to reproduce the issue
Potential security impact
Any relevant logs or screenshots that do not contain secrets or personal data
A suggested mitigation, if available

Please do not include:

API keys
Passwords
Access tokens
Telegram bot tokens
Telegram chat IDs
Private customer information
Authentication credentials
Other sensitive information
Secrets and Credentials

The repository must never contain real credentials.

Examples of sensitive information include:

Google Gemini API keys
Qdrant credentials
Telegram bot tokens
Telegram chat IDs
n8n credentials
Authentication tokens
Webhook secrets
Database credentials

The repository provides:

.env.example

as a configuration reference.

Real environment files such as:

.env

are excluded through .gitignore.

Credentials for external services should be configured locally through n8n or the appropriate environment configuration.

Workflow Security

The exported n8n workflows are sanitized before being committed to the repository.

The public workflow files must not contain:

API keys
Access tokens
Telegram chat IDs
n8n credential identifiers
Private webhook identifiers
Private instance information
Production configuration

Before publishing an updated workflow export, inspect the JSON for credentials and private configuration.

Sensitive Customer Information

The system is designed for customer-support automation and must not be used to collect unnecessary sensitive information.

The AI assistant must never request or store:

Passwords
Authentication codes
Complete payment card numbers
CVV codes
Other sensitive payment credentials

Customers should be directed to appropriate official recovery or support procedures when authentication or account-security issues occur.

Financial and Security Requests

The AI assistant must not independently make high-risk operational decisions.

The following cases should be escalated to human support:

Suspected fraud
Unauthorized transactions
Duplicate charges
Payment disputes
Account compromise
Unauthorized account access
Security incidents
Refund approval requests
Policy exceptions
Other cases requiring human judgment

The AI assistant must not approve, promise, or initiate refunds.

Knowledge Base Security

The knowledge base is treated as an approved source of information for the AI assistant.

Only reviewed and approved documents should be added to:

knowledge/

Knowledge documents should not contain:

Passwords
API keys
Authentication tokens
Customer personal information
Payment credentials
Private internal secrets
Production access information

Changes to the knowledge base should be reviewed before running the ingestion workflow.

Vector Database Security

Qdrant stores vector representations of the approved knowledge base.

The local development instance is exposed on:

http://localhost:6333

The Qdrant database should not be exposed publicly without appropriate authentication, network controls, and access restrictions.

Production deployments should use appropriate security controls for:

Network access
Authentication
Encryption
Data retention
Backup protection
Access permissions
Webhook Security

The current project exposes the customer-support workflow through an HTTP webhook.

The validated project implementation does not yet provide production-grade webhook authentication.

Therefore, the current webhook should be considered a development and demonstration interface rather than a hardened public production endpoint.

Production deployments should implement appropriate controls such as:

Authentication
Authorization
Request validation
Rate limiting
Input size limits
Abuse protection
HTTPS
Request logging and monitoring
Telegram Security

Telegram is used for human-support escalation notifications.

Telegram credentials and chat identifiers must never be committed to the repository.

Production deployments should ensure that escalation messages do not expose unnecessary customer-sensitive information.

Docker Security

The project uses Docker for local infrastructure.

Production deployments should avoid unnecessary exposure of container ports and should apply appropriate container and network security controls.

The shared Docker network used by the development environment is:

ai-support-net

Local development services should not be exposed to the public internet unless required and properly secured.

AI Safety Considerations

The system uses deterministic rules and guardrails in addition to generative AI.

The language model should not be treated as an authoritative source outside the retrieved knowledge base.

The system is designed around the following principles:

Retrieve evidence before generating an answer.
Restrict answers to approved knowledge.
Detect insufficient knowledge.
Detect known knowledge conflicts.
Apply deterministic escalation rules.
Escalate sensitive or high-risk requests.
Avoid requesting sensitive credentials.
Keep humans involved in decisions requiring human judgment.
Data Handling

The current project is intended for controlled development and demonstration.

Do not use real customer personal information, payment information, authentication credentials, or production-sensitive data in local testing.

Use synthetic or non-sensitive test data when validating the workflows.

Production Security Limitations

The current version does not yet provide:

Production-grade authentication
Role-based access control
Comprehensive audit logging
Rate limiting
Advanced monitoring
Automated security scanning
Production secrets management
Full data-retention controls
Formal threat modeling
Large-scale security testing

These areas should be addressed before using the system in a production environment.

Responsible Disclosure

Security researchers and contributors are encouraged to report vulnerabilities privately and responsibly.

Please allow reasonable time for investigation and remediation before publicly disclosing a security vulnerability.

Security Status

Current Version: 1.0

Security Classification: Development / Portfolio Project

This project demonstrates security-aware AI automation practices but should not be considered production-hardened security infrastructure without additional controls and testing.

Author

Raghad Alibrahim

Informatics Engineering | AI & Cybersecurity | Automation