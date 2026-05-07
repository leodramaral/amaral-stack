---
title: "Integrando Groq ao OpenAI SDK"
date: '2026-05-06T21:20:33-04:00'
author: "Leandro Amaral"
tags: ["Groq", "OpenAI", "NodeJS", "Backend", "IA"]
description: "Como resolvi gargalos de hardware usando a compatibilidade do Groq com a OpenAI SDK para estudar LLMs."
draft: true
---

# Integrando Groq ao OpenAI SDK

Recentemente estive trabalhando com o SDK da OpenAI para implementar um chatbot rodando uma modelo LLM local, o problema é meu computador de desenvolvimento não tem potência para esse fim, foi quando conheci o serviço [**Groq**](https://groq.com/).

### O que é o Groq?

O Groq não é apenas mais um provedor de nuvem. Enquanto os gigantes do setor utilizam GPUs, eles desenvolveram uma arquitetura de hardware própria chamada **LPU (Language Processing Unit)**. O foco aqui é velocidade bruta e baixa latência.

O que resolveu meu problema foi descobrir que, além de possuir sua própria SDK, o Groq oferece **total compatibilidade com a OpenAI SDK**. Isso me permitiu manter todo o código que eu já tinha escrito, apenas trocando o "motor" da aplicação. O Groq possui um tier free que é bastante útil para esse tipo de estudo e validação.

---

### Cadastro e API Key

---

### Implementação Básica: OpenAI SDK com "Cérebro" Groq

Para quem já utiliza a biblioteca da OpenAI em JavaScript/TypeScript, a transição é transparente. O segredo está em apontar o `baseURL` para o endpoint de compatibilidade do Groq.

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: 'SUA_CHAVE_GROQ',
  baseURL: 'https://api.groq.com/openai/v1', // Integração via compatibilidade
});

async function main() {
  const systemPrompt = 'Você é um atendente virtual de e-commerce, direto e educado.';
  const userMessage = 'Meu pedido 550e8400 está atrasado. Consegue ver o status?';

  const response = await openai.chat.completions.create({
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userMessage }
    ],
    model: 'llama3-8b-8192',
  });

  console.log(response.choices[0].message.content);
  /**
   * SAÍDA ESPERADA:
   * "Sinto pelo atraso. Vou consultar o status do pedido 550e8400 agora."
   */
}

main();
```

### Usando Function Tools
Aqui entra o Function Calling: você descreve uma função real do seu sistema e o modelo decide quando chamá-la. Isso melhora o direcionamento porque o modelo passa a buscar dados objetivos (ex.: status do pedido) em vez de tentar "inventar" respostas.

```javascript
// Reaproveita o cliente OpenAI do exemplo anterior.

async function getOrderStatus({ orderId }) {
  // Simula consulta real no seu banco/ERP.
  return { orderId, status: 'em separacao', eta: '2 dias' };
}

async function runAgent() {
  const userMessage = 'Meu pedido 550e8400 está atrasado. Consegue ver o status?';

  const messages = [
    { role: 'user', content: userMessage }
  ];

  const tools = [{
    type: 'function',
    function: {
      name: 'get_order_status',
      description: 'Consulta o status de um pedido no banco de dados',
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
    model: 'llama3-70b-8192',
    messages,
    tools,
  });

  const toolCall = response.choices[0].message.tool_calls[0];
  console.log(toolCall.function);
  /**
   * SAÍDA ESPERADA:
   * { name: 'get_order_status', arguments: '{"orderId":"550e8400"}' }
   */

  const toolArgs = JSON.parse(toolCall.function.arguments);
  const toolResult = await getOrderStatus(toolArgs);

  const finalResponse = await openai.chat.completions.create({
    model: 'llama3-70b-8192',
    messages: [
      ...messages,
      response.choices[0].message,
      {
        role: 'tool',
        tool_call_id: toolCall.id,
        name: 'get_order_status',
        content: JSON.stringify(toolResult),
      },
    ],
  });

  console.log(finalResponse.choices[0].message.content);
  /**
   * SAÍDA ESPERADA:
   * "Sinto pelo atraso. Seu pedido 550e8400 está em separacao e a previsao de entrega é de 2 dias."
   */
}

runAgent();
```

### Conclusão: Uma Ótima Escolha para Estudos
O Groq me permitiu estudar conceitos como Function Calling (Tools) e Streamings (que quero abordar em breve em outro post) com a mesma experiência de uma API paga de alto nível, mas com um tempo de resposta baixíssimo e sem onerar meu hardware local.

Pontos de Atenção (Limitações):

- IDs de Modelos: O Groq possui uma biblioteca de modelos que devem ser utilizados. Confira [aqui](https://console.groq.com/docs/models) a lista.

- Rate Limits: O plano gratuito é excelente, mas possui limites de requisições por minuto (RPM) que devem ser monitorados no seu dashboard. Consulte também a [tabela](https://console.groq.com/docs/rate-limits) de rate limits

- Recursos Exclusivos: Funcionalidades específicas da OpenAI, como geração de imagens com DALL-E ou Fine-tuning nativo, não são acessíveis por este endpoint. Mais detalhes sobre a compatabilidade [aqui](https://console.groq.com/docs/openai)

Para quem está começando no mundo das LLMs e não quer (ou não pode) investir em hardware pesado agora, o Groq é, sem dúvida, o melhor ponto de partida para transformar ideias em código funcional.
