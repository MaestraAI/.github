# MaestraAI

MaestraAI gives musicians AI-generated feedback on their practice recordings. Upload a
score, record yourself playing against it, and get back an analysis of pitch accuracy
and harmonic context -- turned into human-readable feedback -- to guide your next
practice session.

This is a multi-repo project. Each repository contains detailed READMEs with design decisions (message broker choices, deployment strategies, etc.) and development instructions, and our [Notion space](https://app.notion.com/p/Public-API-39d768a1b93c8154ad73cf3677cface4?source=copy_link) contains architectural design documents, detailing design choices and decisions throughout the development process.

## Architecture Overview

1. An end user uploads a musical score (MusicXML) to the **[maestra-ai-api](https://github.com/maestra-ai/maestra-ai-api)**.
2. The API stores the file and publishes an event;
3. The **Score Analysis Service** receives the message, performs harmonic analysis, calculates the expected pitch (in Hz) of every note, stores the analysis, and publishes a completion message to a queue. 
5. The client streams a practice recording, send to the the API service in ~15-30 second chunks.
6. The **Pitch Analysis Service** receives the recording and compares each chunk against the score's expected pitches, sending a completed raw analysis to an LLM
7. The LLM turns the analysis into human-readable feedback and returns it to the caller
8. The client polls the API for status and receives the finished feedback.

## Repositories

| Repo | Purpose |
|------|---------|
| [maestra-ai-api](https://github.com/maestra-ai/maestra-ai-api) | Public API -- the entrypoint for uploads, the four MVP endpoints, and the only thing clients talk to directly |
| [maestra-ai-infra](https://github.com/maestra-ai/maestra-ai-infra) | Terraform for infrastructure (RabbitMQ topology today; more to come) -- structured as reusable modules composed per environment |
| maestra-ai-score-analysis | Harmonic analysis of uploaded scores *(planned)* |
| maestra-ai-pitch-analysis | Pitch analysis of practice recordings + LLM feedback generation *(planned)* |
| maestra-ai-ui | Front end client for the end user to interact with our application *(planned)* |

## Stack

Python + FastAPI for the API layer, RabbitMQ for messaging between services (with a
documented path to swap for AWS SQS if this ever needed to scale), DynamoDB and S3
(MinIO locally) for task tracking and storage, and Terraform for infrastructure as code.
Full rationale for each stack choice lives each individaul repo's README file. 
As the entrypoing for the server-side application, **[maestra-ai-api](https://github.com/maestra-ai/maestra-ai-api)** contains the most detailed information about Stack choices. 
For full details, visit our [Notion documentation](https://app.notion.com/p/Public-API-39d768a1b93c8154ad73cf3677cface4?source=copy_link)

## Getting started

Start with [maestra-ai-api](https://github.com/maestra-ai/maestra-ai-api)'s README --
it covers running the full stack locally in Docker, applying the RabbitMQ topology via
`maestra-ai-infra`, and debugging the API in VS Code.
