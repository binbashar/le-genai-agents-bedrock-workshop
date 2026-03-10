# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a CDK scaffolding workshop for rapid prototyping of AI-powered applications using Amazon Bedrock Knowledge Bases and Agents. The project is a PNPM monorepo with two main packages: `cdk_infra` (backend AWS infrastructure) and `reactjs_ui` (React demonstration frontend).

## Architecture

### Use Cases
- **Chatbot**: AI-powered chatbot with knowledge base and action group integration
- **Text2SQL**: Natural language to SQL conversion using Bedrock Agent with Athena integration

### Key Components
- **Backend (`packages/cdk_infra`)**: AWS CDK infrastructure with Bedrock Agents, Knowledge Bases, Lambda functions, API Gateway, DynamoDB, and Cognito
- **Frontend (`packages/reactjs_ui`)**: React demonstration UI with Cognito authentication and Bedrock integration

## Common Commands

### Development Setup
```bash
# Install all dependencies
pnpm install

# Initialize Python virtual environment for CDK
pnpm run cdk_infra:init-python-venv

# Generate OpenAPI schemas
pnpm run cdK_infra:generate-openapi-schemas

# Generate prompts
pnpm run cdk_infra:generate-prompt
```

### Infrastructure Management
```bash
# Deploy all CDK stacks
pnpm run cdk_infra:deploy

# Destroy all CDK stacks  
pnpm run cdk_infra:destroy

# CDK commands (synth, diff, etc.)
pnpm run cdk [command]
```

### Frontend Development
```bash
# Start Vite development server
pnpm run vite

# Build frontend
pnpm run vite build

# Preview build
pnpm run vite preview
```

### Full Build
```bash
# Complete build process (license, python venv, schemas, synth, frontend)
pnpm run build
```

### Testing
```bash
# Run CDK tests
pnpm --filter cdk_infra test
```

## Development Notes

### Cursor Rules Integration
This project uses Cursor rules that enforce a Plan/Act workflow:
- Start in PLAN mode by default - gather information but don't make changes
- Switch to ACT mode only when user explicitly approves the plan
- Always communicate current mode in responses
- All code and explanations must be in English

### Key Directories
- `packages/cdk_infra/src/backend/agents/` - Bedrock Agent action groups
- `packages/cdk_infra/src/stacks/` - CDK stack definitions
- `packages/cdk_infra/src/prompt/` - Agent instructions and orchestration templates
- `packages/reactjs_ui/src/components/` - Reusable React components
- `packages/reactjs_ui/src/views/` - Page components

### AWS Services Used
Backend: Bedrock, OpenSearch Serverless, S3, API Gateway, Lambda, DynamoDB, CloudWatch, IAM, Cognito, Glue, Athena

### Prerequisites
- AWS CLI configured with appropriate permissions
- CDK bootstrapped in target region (recommended: us-west-2)
- Node.js ≥20.18.1, PNPM 9.15.0, Python ≥3.12, Docker
- Bedrock model access: Claude (Sonnet 3.5, 4), Titan Embeddings v2, Nova Lite
- Always use binbash profile. Do not use other AWS profiles. Do not export the AWS_PROFILE if you are using it.

### Git Hygiene
- `packages/reactjs_ui/src/aws-exports.js` contains real Cognito IDs — gitignored, never commit
- `.mcp.json` is local dev config — gitignored, never commit
- `.env.example` is the safe template (empty values) — committed as reference