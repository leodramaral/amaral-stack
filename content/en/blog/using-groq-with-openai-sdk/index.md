---
title: "Using Groq with the OpenAI SDK"
date: '2026-05-06T21:20:33-04:00'
author: "Leandro Amaral"
tags: ["Groq", "OpenAI", "NodeJS", "Backend", "AI"]
description: "How I solved hardware bottlenecks using Groq's compatibility with the OpenAI SDK to study LLMs."
---

# Using Groq with the OpenAI SDK

Recently, I was working with the OpenAI SDK to implement a chatbot running a local LLM model, but my development computer lacks the power for it. That's when I discovered the [**Groq**](https://groq.com/) service.

### What is Groq?

Groq is not just another cloud provider. While industry giants use GPUs, they have developed their own hardware architecture called **LPU (Language Processing Unit)**. The focus here is on raw speed and low latency.

What solved my problem was discovering that, in addition to having its own SDK, Groq offers **full compatibility with the OpenAI SDK**. This allowed me to keep all the code I had already written, just changing the application's "engine." Groq has a free tier that is quite useful for this type of study and validation.

---

### Registration and API Key
Creating a key to use a model is free.

- Access the [Groq website](https://console.groq.com/home) and register;
- NOTE: As of this post's publication, there is a bug that prevents creating accounts with Outlook or Hotmail;
- A default 'Organization' and 'Project' will be created;

{{< figure src="groq-api-keys-page.png" alt="Groq page where api keys are listed" >}}

- Create your API key by clicking the dedicated button;

### Enabling a Model
After creating the key, you must enable a model in the panel to use it.

- Access the page [to enable one or more models for use](https://console.groq.com/settings/project/limits);
- In the "Allowed Models" section, select "Edit" where you can choose one or more models for use;


{{< figure src="groq-allowed-models.png" alt="Groq page where models are enabled" >}}

---

### Basic Implementation: OpenAI SDK with a Groq "Brain"

For those who already use the OpenAI library in JavaScript/TypeScript, the transition is seamless. The secret is to point the `baseURL` to Groq's compatibility endpoint.

```typescript
import OpenAI from 'openai';

const MODEL = 'meta-llama/llama-4-scout-17b-16e-instruct';

const openai = new OpenAI({
  apiKey: 'YOUR_GROQ_API',
  baseURL: 'https://api.groq.com/openai/v1', // Integration via compatibility
});

async function main() {
  const systemPrompt = 'You are a direct and polite e-commerce virtual assistant.';
  const userMessage = 'My order 550e8400 is late. Can you check the status?';

  const response = await openai.chat.completions.create({
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userMessage }
    ],
    model: MODEL,
  });

  console.log(response.choices[0].message.content);
  /**
   * EXPECTED OUTPUT:
   * "Of course, I can check the status of your order! Please wait a moment while I retrieve the information from our systems.
      Yes, I was able to view your order. I can inform you that the current status is 'Processing' and the estimated delivery is an additional 2 days.
      Would you like me to send you an update email or call you to discuss other options? Thank you for your patience!"
   */
}

main();
```

### Using Function Tools
This is where Function Calling comes in: you describe a real function from your system, and the model decides when to call it. This improves direction because the model starts seeking objective data (e.g., order status) instead of trying to "invent" answers.

```typescript
// Reuses the OpenAI client from the previous example.
const MODEL = 'meta-llama/llama-4-scout-17b-16e-instruct';

async function getOrderStatus({ orderId }: { orderId: string }): Promise<{ orderId: string, status: string, eta: string }> {
        // Simulates a real query to your database.
        return { orderId, status: 'in separation', eta: '2 days' };
}

async function runAgent() {
  const userMessage = 'My order 550e8400 is late. Can you check the status?';

  const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
    { role: 'user', content: userMessage }
  ];

  const tools: OpenAI.Chat.Completions.ChatCompletionTool[] = [{
    type: 'function',
    function: {
      name: 'get_order_status',
      description: 'Queries the status of an order in the database',
      parameters: {
        type: 'object',
        properties: {
          orderId: { type: 'string' }
        },
        required: ['orderId'],
      },
    },
  }];

  const response = await openai.chat.completions.create({
    model: MODEL,
    messages,
    tools,
  });

  const { message } = response.choices[0];

  const toolCall = message.tool_calls[0];
  console.log(toolCall.function);
  /**
   * EXPECTED OUTPUT:
   * { name: 'get_order_status', arguments: '{"orderId":"550e8400"}' }
   */

  const toolArgs = JSON.parse(toolCall.function.arguments);
  const toolResult = await getOrderStatus(toolArgs);

  const finalResponse = await openai.chat.completions.create({
    model: MODEL,
    messages: [
      ...messages,
      {
        role: 'assistant',
        tool_calls: message.tool_calls,
      },
      {
        role: 'tool',
        tool_call_id: toolCall.id,
        name: toolCall.function.name,
        content: JSON.stringify(toolResult),
      },
    ],
  });

  console.log(finalResponse.choices[0].message.content);
  /**
   * EXPECTED OUTPUT:
   * "Sorry for the delay. Your order 550e8400 is in separation and the estimated delivery is 2 days."
   */
}

runAgent();
```

### Conclusion: A Great Choice for Studies
Groq allowed me to study concepts like Function Calling (Tools) and Streaming (which I plan to cover in another post soon) with the same experience as a high-level paid API, but with a very low response time and without burdening my local hardware.

Points of Attention (Limitations):

- **Model IDs**: Groq has a [library](https://console.groq.com/docs/models) of models that must be used.

- **Rate Limits**: The free plan is excellent but has requests per minute (RPM) limits that should be monitored on your dashboard. Check the rate limits [table](https://console.groq.com/docs/rate-limits).

- **Exclusive Features**: Some OpenAI features [are not supported](https://console.groq.com/docs/openai#currently-unsupported-openai-features).

For those starting in the world of LLMs who don't want (or can't) invest in heavy hardware right now, Groq is, without a doubt, the best starting point to turn ideas into functional code.
