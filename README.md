# Flux IA API - Documentação Oficial

> Sistema de conversação com IA via Server-Sent Events (SSE), oferecendo múltiplos modelos especializados em raciocínio, pesquisa e combinações avançadas. **Seu diferencial reside na baixa censura, permitindo interações mais amplas e diretas.**

A Flux IA API permite a integração de capacidades avançadas de inteligência artificial em suas aplicações, com foco em flexibilidade e abrangência nas respostas. Esta documentação detalha os endpoints, modelos disponíveis, formato de resposta via SSE e tratamento de erros.

---

## Informações Gerais

*   **Base URL:** `https://hubsdev.com`
*   **Formato de Dados:** Server-Sent Events (SSE) – cada linha é um JSON puro
*   **Autenticação:** Chave de API via parâmetro `key`
*   **Timeout Padrão:** 120 segundos
*   **Consumo:** 1.0 crédito por requisição válida

---

## Modelos Disponíveis

| Modelo (parâmetro `model`) | Descrição |
| :--- | :--- |
| `flux-thinking` | Raciocínio avançado, análise lógica, resolução de problemas complexos. Ideal para programação, matemática, planejamento e pesquisas elaboradas. |
| `flux-search` | Acesso otimizado a busca de informações. Respostas atualizadas e consultas rápidas com alta precisão. |
| `flux-search-max` | Versão premium do Search – pesquisas mais profundas, respostas completas e análise de múltiplas fontes. |
| `flux-thinking-max` | Versão mais poderosa da linha Thinking. Alta complexidade, geração de conteúdo avançado e raciocínio aprofundado. |
| `flux-thinking-search` | Combina raciocínio avançado com pesquisa em tempo real. Analisa informações encontradas para respostas precisas e contextualizadas. |
| `flux-thinking-search-max` | Modelo mais completo – máximo de raciocínio + pesquisas avançadas. Respostas extremamente detalhadas e acesso expandido. |

> **Nota:** O parâmetro `model` deve ser exatamente como listado acima (lowercase com hífens).

---

## Endpoint: Enviar requisição para Flux.

Inicia uma sessão de chat com streaming de eventos em tempo real.

**Endpoint:** `GET /api/ia/flux-chat?key={KEY}&prompt={MENSAGEM}&chat_id={ID}&model={MODELO}`

### Parâmetros

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `key` | `string` | Sim | Chave de API válida |
| `prompt` | `string` | Sim | Mensagem enviada para a IA (mínimo 2 caracteres, máximo 2000) |
| `chat_id` | `string` | Sim | Identificador único da conversa (máximo 6 caracteres, usado para manter histórico) |
| `model` | `string` | Sim | Modelo da IA (conforme tabela de modelos disponíveis) |

---

## Resposta (SSE)

A resposta é um fluxo de **linhas JSON puras** (cada linha é um objeto JSON, sem prefixo `data:`). Os eventos são enviados sequencialmente.

### Tipos de Evento

| Evento | Descrição | Exemplo |
| :--- | :--- | :--- |
| `flux-thinking-start` | Início do raciocínio interno | `{"type":"flux-thinking-start","result":"Analisando..."}` |
| `flux-thinking` | Fragmento do raciocínio (delta) | `{"type":"flux-thinking","result":"a pergunta"}` |
| `flux-thinking-end` | Fim do raciocínio | `{"type":"flux-thinking-end","result":null}` |
| `flux-text-start` | Início da resposta textual | `{"type":"flux-text-start","result":"Olá"}` |
| `flux-text` | Fragmento da resposta (delta) | `{"type":"flux-text","result":" mundo!"}` |
| `flux-text-end` | Fim da resposta textual | `{"type":"flux-text-end","result":null}` |
| `flux-suggestions-start` | Sugestões de perguntas adicionais | `{"type":"flux-suggestions-start","result":["Pergunta 1","Pergunta 2"]}` |
| `flux-suggestions` | Mesmo conteúdo do start (para compatibilidade) | `{"type":"flux-suggestions","result":["..."]}` |
| `error` | Erro durante o streaming | `{"type":"error","result":{"message":"Timeout"}}` |

> **Importante:** Os eventos `-end` não carregam conteúdo acumulado, apenas sinalizam o fim. As sugestões não possuem evento `-end`.

### Exemplo de Fluxo de Resposta

```json
{"type":"flux-thinking-start","result":"Vou processar sua pergunta..."}
{"type":"flux-thinking","result":" Primeiro, analiso o contexto"}
{"type":"flux-thinking-end","result":null}
{"type":"flux-text-start","result":"A resposta para sua pergunta é:"}
{"type":"flux-text","result":" explicação detalhada aqui."}
{"type":"flux-text-end","result":null}
{"type":"flux-suggestions-start","result":["Pergunta relacionada 1","Pergunta relacionada 2"]}
{"type":"flux-suggestions","result":["Pergunta relacionada 1","Pergunta relacionada 2"]}
```

---

## Códigos de Erro (HTTP)

| HTTP | Código interno | Descrição |
| :--- | :--- | :--- |
| 400 | `PROMPT_REQUIRED` | Parâmetro `prompt` não informado ou não é string |
| 400 | `PROMPT_TOO_SHORT` | Prompt com menos de 2 caracteres |
| 400 | `PROMPT_TOO_LONG` | Prompt com mais de 2000 caracteres |
| 400 | `CHAT_ID_REQUIRED` | Parâmetro `chat_id` não informado |
| 400 | `CHAT_ID_TOO_LONG` | `chat_id` com mais de 6 caracteres |
| 400 | `MODEL_REQUIRED` | Parâmetro `model` não informado |
| 400 | `INVALID_MODEL` | Modelo não suportado (ver tabela de modelos) |
| 500 | `ERRO_INTERNO` | Erro interno no processamento da requisição |

### Erros Durante o Stream (evento `error`)

Durante o streaming, podem ocorrer erros reportados no próprio canal SSE:

```json
{"type":"error","result":{"message":"timeout of 120000ms exceeded"}}
```

---

## Regras de Negócio

*   **Histórico de Conversa:** O sistema mantém automaticamente o histórico por `chat_id`. Cada novo `prompt` adiciona a mensagem do usuário e a resposta do assistente ao histórico daquela conversa.
*   **Consumo:** Cada requisição bem-sucedida (início do stream) consome **0.05 crédito** da chave de API, independente do tamanho da resposta.
*   **Timeout:** A conexão com o provedor de IA tem limite de **120 segundos**. Caso ultrapasse, um evento `error` é enviado e o stream é fechado.
*   **Persistência:** O histórico é armazenado e reutilizado em chamadas posteriores com o mesmo `chat_id`.

---

## Suporte e Atualizações

*   Novos modelos poderão ser adicionados à tabela acima sem aviso prévio.
*   Esta documentação está sujeita a melhorias conforme evolução da API.
*   Para reportar problemas, utilize os canais oficiais de suporte da plataforma.

---

*Flux IA – Inteligência que flui com você.*
