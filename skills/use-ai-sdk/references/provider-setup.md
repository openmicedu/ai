---
title: Provider Setup
description: Reference for using AI SDK provider packages directly (without the Vercel AI Gateway).
---

# Provider Setup

Use each provider's dedicated package directly with the provider's own API key. This avoids routing requests through the Vercel AI Gateway.

## Installation

Install the provider package(s) you need with the project's package manager:

```bash
pnpm add @ai-sdk/openai      # OpenAI
pnpm add @ai-sdk/anthropic   # Anthropic
pnpm add @ai-sdk/google      # Google
```

## Authentication

Each provider reads its API key from a provider-specific environment variable:

```env filename=".env.local"
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here
GOOGLE_GENERATIVE_AI_API_KEY=your_google_key_here
```

Check the provider's docs at `node_modules/@ai-sdk/<provider>/docs/` for the exact environment variable name and any extra configuration.

## Usage

Import the provider from its package and pass a model to the AI SDK function:

```ts
import { generateText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const { text } = await generateText({
  model: anthropic('claude-sonnet-4.5'),
  prompt: 'What is love?',
});
```

```ts
import { openai } from '@ai-sdk/openai';
model: openai('gpt-4.1');

import { google } from '@ai-sdk/google';
model: google('gemini-2.5-pro');
```

To customize configuration (custom base URL, headers, etc.), create a provider instance:

```ts
import { createAnthropic } from '@ai-sdk/anthropic';

const anthropic = createAnthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});
```

## Find Available Models

**Important**: Always fetch the current model list before writing code. Never use model IDs from memory - they may be outdated.

The Vercel AI Gateway model listing is a convenient catalog for discovering current model IDs across providers, even when you call the providers directly. Note that the gateway lists IDs in `provider/model` form (e.g. `anthropic/claude-sonnet-4.5`); when calling a provider package directly, drop the `provider/` prefix and pass just the model portion (e.g. `anthropic('claude-sonnet-4.5')`).

```bash
# Anthropic models
curl -s https://ai-gateway.vercel.sh/v1/models | jq -r '[.data[] | select(.id | startswith("anthropic/")) | .id] | reverse | .[]'

# OpenAI models
curl -s https://ai-gateway.vercel.sh/v1/models | jq -r '[.data[] | select(.id | startswith("openai/")) | .id] | reverse | .[]'

# Google models
curl -s https://ai-gateway.vercel.sh/v1/models | jq -r '[.data[] | select(.id | startswith("google/")) | .id] | reverse | .[]'
```

You can also consult the provider's own documentation/model listing for the authoritative set of IDs.

When multiple versions of a model exist, use the one with the highest version number (e.g., prefer `claude-sonnet-4.6` over `claude-sonnet-4.5` over `claude-sonnet-4`).
