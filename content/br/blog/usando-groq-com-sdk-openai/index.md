---
title: "Usando Groq com o SDK da OpenAI"
date: '2026-05-06T21:20:33-04:00'
author: "Leandro Amaral"
tags: ["Groq", "OpenAI", "NodeJS", "Backend", "IA"]
description: "Como resolvi gargalos de hardware usando a compatibilidade do Groq com a OpenAI SDK para estudar LLMs."
---

# Usando Groq com o SDK da OpenAI

Recentemente estive trabalhando com o SDK da OpenAI para implementar um chatbot rodando uma modelo LLM local, o problema é meu computador de desenvolvimento não tem potência para esse fim, foi quando conheci o serviço [**Groq**](https://groq.com/).

### O que é o Groq?

O Groq não é apenas mais um provedor de nuvem. Enquanto os gigantes do setor utilizam GPUs, eles desenvolveram uma arquitetura de hardware própria chamada **LPU (Language Processing Unit)**. O foco aqui é velocidade bruta e baixa latência.

O que resolveu meu problema foi descobrir que, além de possuir sua própria SDK, o Groq oferece **total compatibilidade com a OpenAI SDK**. Isso me permitiu manter todo o código que eu já tinha escrito, apenas trocando o "motor" da aplicação. O Groq possui um tier free que é bastante útil para esse tipo de estudo e validação.

---

### Cadastro e API Key
Criar uma chave para usar um modelo é gratuito

- Acesse o [site do Groq](https://console.groq.com/home) e faça seu cadastro;
- OBS: até a publicação desse post, existe um bug que não permite criação de conta Outlook ou Hotmail;
- Será criado uma 'Organização' e 'Projeto' padrão;

{{< figure src="groq-api-keys-page.png" alt="Página do Groq onde são listadas as api keys" >}}

- Crie a sua api key, acionando o botão dedicado;

### Habilitando um modelo
Após a criação da chave, você deve habilitar um modelo no painel para poder utilizá-lo

- Acesse a página [para habilitar um ou mais modelos para uso](https://console.groq.com/settings/project/limits);
- Na seção "Allowed Models" selecione "Edit" onde você poderá escolher um ou mais modelos para uso;


{{< figure src="groq-allowed-models.png" alt="Página do Groq onde os modelos são habilitados" >}}

---

### Implementação Básica: OpenAI SDK com "Cérebro" Groq

Para quem já utiliza a biblioteca da OpenAI em JavaScript/TypeScript, a transição é transparente. O segredo está em apontar o `baseURL` para o endpoint de compatibilidade do Groq.

```typescript
import OpenAI from 'openai';

const MODEL = 'meta-llama/llama-4-scout-17b-16e-instruct';
const openai = new OpenAI({
  apiKey: 'YOUR_GROQ_API',
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
    model: MODEL,
  });

  console.log(response.choices[0].message.content);
  /**
   * SAÍDA ESPERADA:
   * "Claro, posso verificar o status do seu pedido! Por favor, aguarde um minuto enquanto eu busco a informação em nossos sistemas.
      Sim, consegui visualizar o seu pedido. Posso te informar que o status atualemente é "Em processamento" e que a previsão de entrega é 2 dias adicionais.
      Quer que eu te envie um email de atualização ou que eu te chame para discutir outras opções? Obrigado por sua paciência!"
   */
}

main();

```

### Usando Function Tools
Aqui entra o Function Calling: você descreve uma função real do seu sistema e o modelo decide quando chamá-la. Isso melhora o direcionamento porque o modelo passa a buscar dados objetivos (ex.: status do pedido) em vez de tentar "inventar" respostas.

```javascript
// Reaproveita o cliente OpenAI do exemplo anterior.

const MODEL = 'meta-llama/llama-4-scout-17b-16e-instruct';

async function getOrderStatus({ orderId }: { orderId: string }): Promise<{ orderId: string, status: string, eta: string }> {
        // Simula consulta real no seu banco.
        return { orderId, status: 'em separacao', eta: '2 dias' };
}

async function runAgent() {
  const userMessage = 'Meu pedido 550e8400 está atrasado. Consegue ver o status?';

  const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
    { role: 'user', content: userMessage }
  ];

  const tools: OpenAI.Chat.Completions.ChatCompletionTool[] = [{
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
    model: MODEL,
    messages,
    tools,
  });

  const { message } = response.choices[0];

  const toolCall = message.tool_calls[0];
  console.log(toolCall.function);
  /**
   * SAÍDA ESPERADA:
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
   * SAÍDA ESPERADA:
   * "Sinto pelo atraso. Seu pedido 550e8400 está em separacao e a previsao de entrega é de 2 dias."
   */
}

runAgent();

```

### Conclusão: Uma Ótima Escolha para Estudos
O Groq me permitiu estudar conceitos como Function Calling (Tools) e Streamings (que quero abordar em breve em outro post) com a mesma experiência de uma API paga de alto nível, mas com um tempo de resposta baixíssimo e sem onerar meu hardware local.

Pontos de Atenção (Limitações):

- IDs de Modelos: O Groq possui uma [biblioteca](https://console.groq.com/docs/models) de modelos que devem ser utilizados.

- Rate Limits: O plano gratuito é excelente, mas possui limites de requisições por minuto (RPM) que devem ser monitorados no seu dashboard. Consulte a [tabela](https://console.groq.com/docs/rate-limits) de rate limits.

- Recursos Exclusivos: algumas funcionalidades da OpenAI [não são suportadas](https://console.groq.com/docs/openai#currently-unsupported-openai-features).

Para quem está começando no mundo das LLMs e não quer (ou não pode) investir em hardware pesado agora, o Groq é, sem dúvida, o melhor ponto de partida para transformar ideias em código funcional.