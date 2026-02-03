# NetSuite SuiteScript 2.x N/llm Module

## Overview

The **N/llm module** enables generative AI capabilities in NetSuite SuiteScript 2.1, allowing developers to interact with Large Language Models (LLMs) through Oracle Cloud Infrastructure (OCI) Generative AI service.

**Module ID:** `N/llm`
**Since:** NetSuite 2024.1
**SuiteScript Version:** 2.1 only
**Supported Script Types:** Server-side scripts only

## Key Features

| Feature | Description |
|---------|-------------|
| **Text Generation** | Send prompts to LLMs via `generateText()` |
| **Prompt Studio Integration** | Execute managed prompts with `evaluatePrompt()` |
| **RAG Support** | Provide source documents for context-aware responses with citations |
| **Embeddings** | Convert text to vector embeddings for semantic search |
| **Streaming** | Receive real-time partial responses |
| **Structured Output** | Get JSON-formatted responses with schema validation |

## Loading the Module

```javascript
/**
 * @NApiVersion 2.1
 */
define(['N/llm'], function(llm) {
    // Your code here
});

// Or using require (for debugger/testing):
require(['N/llm'], function(llm) {
    // Your code here
});
```

## Methods

### llm.generateText(options)

Sends a prompt to the LLM and receives a response.

**Alias:** `llm.chat(options)`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options.prompt` | string | Yes | The prompt to send to the LLM |
| `options.modelFamily` | string | No | LLM to use (default: Cohere Command A) |
| `options.preamble` | string | No | Initial context/system message |
| `options.chatHistory` | llm.ChatMessage[] | No | Previous conversation messages |
| `options.documents` | llm.Document[] | No | Source documents for RAG |
| `options.responseFormat` | Object | No | JSON schema for structured responses |
| `options.safetyMode` | string | No | Safety mode (default: STRICT) |
| `options.timeout` | number | No | Timeout in ms (default: 30000) |
| `options.modelParameters` | Object | No | Model-specific parameters |
| `options.ociConfig` | Object | No | OCI configuration for unlimited usage |

**Model Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `maxTokens` | number | Maximum tokens to generate |
| `temperature` | number | Randomness (lower = factual, higher = creative) |
| `topK` | number | Tokens considered per generation step |
| `topP` | number | Probability threshold for token consideration |
| `frequencyPenalty` | number | Penalty for frequently appearing tokens |
| `presencePenalty` | number | Penalty for previously used tokens |

**Returns:** `llm.Response`
**Governance:** 100 units

```javascript
const response = llm.generateText({
    prompt: "Summarize the key points of this sales order.",
    modelFamily: llm.ModelFamily.COHERE_COMMAND,
    modelParameters: {
        maxTokens: 1000,
        temperature: 0.2,
        topK: 3,
        topP: 0.7
    }
});

log.debug('Response', response.text);
log.debug('Tokens used', response.usage.totalTokens);
```

---

### llm.generateTextStreamed(options)

Streams LLM responses as they're generated instead of waiting for completion.

**Alias:** `llm.chatStreamed(options)`

**Parameters:** Same as `generateText()`

**Returns:** `llm.StreamedResponse`
**Governance:** 100 units

```javascript
const streamedResponse = llm.generateTextStreamed({
    prompt: "Write a detailed product description."
});

// Access accumulated text
const fullText = streamedResponse.text;
```

---

### llm.evaluatePrompt(options)

Executes a prompt stored in Prompt Studio with variable substitution.

**Alias:** `llm.executePrompt(options)`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options.promptId` | string | Yes | Internal ID of the prompt in Prompt Studio |
| `options.variables` | Object | No | Key-value pairs for prompt variables |
| `options.timeout` | number | No | Timeout in milliseconds |
| `options.ociConfig` | Object | No | OCI configuration for unlimited usage |

**Returns:** `llm.Response`
**Governance:** 100 units

```javascript
const response = llm.evaluatePrompt({
    promptId: 'custprompt_summarize_order',
    variables: {
        orderNumber: 'SO-12345',
        customerName: 'Acme Corp'
    }
});
```

---

### llm.evaluatePromptStreamed(options)

Streams responses from Prompt Studio prompts.

**Alias:** `llm.executePromptStreamed(options)`

**Parameters:** Same as `evaluatePrompt()`

**Returns:** `llm.StreamedResponse`
**Governance:** 100 units

---

### llm.embed(options)

Converts text to vector embeddings for semantic search, recommendations, classification, or clustering.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options.input` | string or string[] | Yes | Text(s) to convert to embeddings |
| `options.embedModelFamily` | string | No | Embedding model to use |
| `options.truncate` | string | No | Truncation method (llm.Truncate enum) |
| `options.timeout` | number | No | Timeout in milliseconds |
| `options.ociConfig` | Object | No | OCI configuration for unlimited usage |

**Returns:** `llm.EmbedResponse`
**Governance:** 100 units

```javascript
const embedResponse = llm.embed({
    input: ["Red running shoes", "Blue sneakers for jogging"],
    embedModelFamily: llm.EmbedModelFamily.COHERE_EMBED
});

// Access embeddings array
const vectors = embedResponse.embeddings;
```

---

### llm.getRemainingFreeUsage()

Returns remaining free API requests in the current month.

**Returns:** `number`
**Governance:** 0 units

```javascript
const remaining = llm.getRemainingFreeUsage();
log.debug('Remaining free usage', remaining);
```

---

### llm.getRemainingFreeEmbedUsage()

Returns remaining free embeddings requests in the current month.

**Returns:** `number`
**Governance:** 0 units

---

### llm.createChatMessage(options)

Creates a chat message object for conversation history.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options.role` | string | Yes | Message author role (use llm.ChatRole enum) |
| `options.text` | string | Yes | Message content |

**Returns:** `llm.ChatMessage`

```javascript
const userMessage = llm.createChatMessage({
    role: llm.ChatRole.USER,
    text: "What are the shipping options?"
});

const assistantMessage = llm.createChatMessage({
    role: llm.ChatRole.ASSISTANT,
    text: "We offer standard and express shipping."
});

// Use in chat history
const response = llm.generateText({
    prompt: "Which is faster?",
    chatHistory: [userMessage, assistantMessage]
});
```

---

### llm.createDocument(options)

Creates a document object for RAG (Retrieval-Augmented Generation).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `options.id` | string | Yes | Unique document identifier |
| `options.data` | string | Yes | Document content |

**Returns:** `llm.Document`

```javascript
const doc1 = llm.createDocument({
    id: 'policy_returns',
    data: 'Our return policy allows returns within 30 days of purchase.'
});

const doc2 = llm.createDocument({
    id: 'policy_shipping',
    data: 'Standard shipping takes 5-7 business days.'
});

const response = llm.generateText({
    prompt: "What is the return policy?",
    documents: [doc1, doc2]
});

// Response includes citations
response.citations.forEach(citation => {
    log.debug('Citation', citation.text + ' from: ' + citation.documentIds);
});
```

## Objects

### llm.Response

| Property | Type | Description |
|----------|------|-------------|
| `text` | string | The LLM's response text |
| `model` | string | Model used for the response |
| `chatHistory` | llm.ChatMessage[] | List of chat messages |
| `citations` | llm.Citation[] | Citations from source documents |
| `documents` | llm.Document[] | Source documents used |
| `usage` | llm.Usage | Token usage statistics |

### llm.StreamedResponse

| Property | Type | Description |
|----------|------|-------------|
| `text` | string | Accumulated streamed text |
| `model` | string | Model used |
| `chatHistory` | llm.ChatMessage[] | Chat messages |
| `citations` | llm.Citation[] | Citations |
| `documents` | llm.Document[] | Source documents |

### llm.EmbedResponse

| Property | Type | Description |
|----------|------|-------------|
| `embeddings` | number[][] | Vector embeddings |
| `inputs` | string[] | Input texts processed |
| `model` | string | Model used |

### llm.ChatMessage

| Property | Type | Description |
|----------|------|-------------|
| `role` | string | Author role (USER, ASSISTANT, SYSTEM) |
| `text` | string | Message content |

### llm.Citation

| Property | Type | Description |
|----------|------|-------------|
| `documentIds` | string[] | IDs of source documents |
| `start` | number | Starting position of cited text |
| `end` | number | Ending position of cited text |
| `text` | string | The cited text |

### llm.Document

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Document identifier |
| `data` | string | Document content |

### llm.Usage

| Property | Type | Description |
|----------|------|-------------|
| `promptTokens` | number | Tokens in request |
| `completionTokens` | number | Tokens in response |
| `totalTokens` | number | Total tokens used |

## Enums

### llm.ModelFamily

Specifies which LLM to use for text generation.

| Value | Model | RAG Support | Preamble Support |
|-------|-------|-------------|------------------|
| `COHERE_COMMAND` | cohere.command-a-03-2025 | Yes | Yes |
| `COHERE_COMMAND_LATEST` | cohere.command-a-03-2025 | Yes | Yes |

```javascript
llm.ModelFamily.COHERE_COMMAND
llm.ModelFamily.COHERE_COMMAND_LATEST
```

### llm.EmbedModelFamily

Specifies which model to use for embeddings.

| Value | Model |
|-------|-------|
| `COHERE_EMBED` | cohere.embed-v4.0 |
| `COHERE_EMBED_LATEST` | cohere.embed-v4.0 |

### llm.ChatRole

Defines message author roles.

| Value | Description |
|-------|-------------|
| `USER` | User message |
| `ASSISTANT` | Assistant/LLM response |
| `SYSTEM` | System instruction |

### llm.SafetyMode

Controls content safety filtering.

| Value | Description |
|-------|-------------|
| `STRICT` | Maximum content filtering (default) |
| `CONTEXTUAL` | Context-aware filtering |
| `NONE` | No filtering |

### llm.Truncate

Specifies truncation method when embedding input exceeds 512 tokens.

| Value | Description |
|-------|-------------|
| `START` | Truncate from the beginning |
| `END` | Truncate from the end |
| `NONE` | Throw error if exceeds limit |

## Complete Examples

### Basic Text Generation

```javascript
/**
 * @NApiVersion 2.1
 * @NScriptType Suitelet
 */
define(['N/llm', 'N/log'], function(llm, log) {

    function onRequest(context) {
        const response = llm.generateText({
            prompt: "Hello World!",
            modelParameters: {
                maxTokens: 1000,
                temperature: 0.2,
                topK: 3,
                topP: 0.7,
                frequencyPenalty: 0.4,
                presencePenalty: 0
            }
        });

        log.debug('LLM Response', response.text);
        log.debug('Remaining free usage', llm.getRemainingFreeUsage());

        context.response.write(response.text);
    }

    return { onRequest: onRequest };
});
```

### RAG with Source Documents

```javascript
/**
 * @NApiVersion 2.1
 * @NScriptType Suitelet
 */
define(['N/llm', 'N/query', 'N/log'], function(llm, query, log) {

    function onRequest(context) {
        // Step 1: Query NetSuite data
        const results = query.runSuiteQL({
            query: "SELECT name, description FROM item WHERE itemtype = 'InvtPart' FETCH FIRST 10 ROWS ONLY"
        }).asMappedResults();

        // Step 2: Create documents from data
        const documents = results.map((item, index) => {
            return llm.createDocument({
                id: 'item_' + index,
                data: 'Product: ' + item.name + '. Description: ' + item.description
            });
        });

        // Step 3: Generate text with documents
        const response = llm.generateText({
            prompt: "Which products are best for outdoor activities?",
            documents: documents,
            preamble: "You are a helpful product expert. Use only the provided documents to answer questions."
        });

        // Step 4: Process citations
        log.debug('Response', response.text);
        response.citations.forEach(function(citation) {
            log.debug('Citation', 'Text: ' + citation.text + ', Sources: ' + citation.documentIds.join(', '));
        });

        context.response.write(response.text);
    }

    return { onRequest: onRequest };
});
```

### Chatbot with Conversation History

```javascript
/**
 * @NApiVersion 2.1
 * @NScriptType Suitelet
 */
define(['N/llm', 'N/log'], function(llm, log) {

    function onRequest(context) {
        // Build conversation history
        const chatHistory = [
            llm.createChatMessage({
                role: llm.ChatRole.USER,
                text: "What products do you sell?"
            }),
            llm.createChatMessage({
                role: llm.ChatRole.ASSISTANT,
                text: "We sell electronics, clothing, and home goods."
            }),
            llm.createChatMessage({
                role: llm.ChatRole.USER,
                text: "Tell me more about electronics."
            }),
            llm.createChatMessage({
                role: llm.ChatRole.ASSISTANT,
                text: "Our electronics include laptops, tablets, and smartphones."
            })
        ];

        // Continue the conversation
        const response = llm.generateText({
            prompt: "Which laptop do you recommend?",
            chatHistory: chatHistory,
            preamble: "You are a helpful sales assistant."
        });

        log.debug('Response', response.text);
        context.response.write(response.text);
    }

    return { onRequest: onRequest };
});
```

### Structured JSON Output

```javascript
/**
 * @NApiVersion 2.1
 * @NScriptType Suitelet
 */
define(['N/llm', 'N/log'], function(llm, log) {

    function onRequest(context) {
        const response = llm.generateText({
            prompt: "Extract customer info: John Smith, email john@example.com, phone 555-1234",
            responseFormat: {
                type: "object",
                required: ["name", "email", "phone"],
                properties: {
                    name: { type: "string" },
                    email: { type: "string" },
                    phone: { type: "string" }
                }
            }
        });

        const customer = JSON.parse(response.text);
        log.debug('Customer Name', customer.name);
        log.debug('Customer Email', customer.email);
        log.debug('Customer Phone', customer.phone);

        context.response.write(JSON.stringify(customer, null, 2));
    }

    return { onRequest: onRequest };
});
```

### Embeddings for Similarity Search

```javascript
/**
 * @NApiVersion 2.1
 * @NScriptType Suitelet
 */
define(['N/llm', 'N/log'], function(llm, log) {

    function cosineSimilarity(vec1, vec2) {
        let dotProduct = 0;
        let mag1 = 0;
        let mag2 = 0;
        for (let i = 0; i < vec1.length; i++) {
            dotProduct += vec1[i] * vec2[i];
            mag1 += vec1[i] * vec1[i];
            mag2 += vec2[i] * vec2[i];
        }
        return dotProduct / (Math.sqrt(mag1) * Math.sqrt(mag2));
    }

    function onRequest(context) {
        // Generate embeddings for products
        const products = [
            "Red running shoes for athletes",
            "Blue casual sneakers",
            "Black leather dress shoes",
            "White tennis shoes"
        ];

        const embedResponse = llm.embed({
            input: products,
            embedModelFamily: llm.EmbedModelFamily.COHERE_EMBED
        });

        // Search query
        const queryEmbed = llm.embed({
            input: ["shoes for running"],
            embedModelFamily: llm.EmbedModelFamily.COHERE_EMBED
        });

        // Find most similar product
        let maxSimilarity = -1;
        let bestMatch = -1;

        for (let i = 0; i < embedResponse.embeddings.length; i++) {
            const similarity = cosineSimilarity(queryEmbed.embeddings[0], embedResponse.embeddings[i]);
            if (similarity > maxSimilarity) {
                maxSimilarity = similarity;
                bestMatch = i;
            }
        }

        log.debug('Best match', products[bestMatch] + ' (similarity: ' + maxSimilarity + ')');
        context.response.write('Best match: ' + products[bestMatch]);
    }

    return { onRequest: onRequest };
});
```

### User Event: Clean Up Record Fields

```javascript
/**
 * @NApiVersion 2.1
 * @NScriptType UserEventScript
 */
define(['N/llm', 'N/record', 'N/log'], function(llm, record, log) {

    function afterSubmit(context) {
        if (context.type !== context.UserEventType.CREATE && context.type !== context.UserEventType.EDIT) {
            return;
        }

        const rec = record.load({
            type: context.newRecord.type,
            id: context.newRecord.id
        });

        const description = rec.getValue({ fieldId: 'description' });

        if (!description) {
            return;
        }

        // Clean up and improve the description
        const response = llm.generateText({
            prompt: "Clean up and professionally rewrite this product description. Keep it concise: " + description,
            modelParameters: {
                maxTokens: 500,
                temperature: 0.3
            }
        });

        rec.setValue({
            fieldId: 'description',
            value: response.text
        });

        rec.save();

        log.debug('Description cleaned', 'Record ID: ' + context.newRecord.id);
    }

    return { afterSubmit: afterSubmit };
});
```

## Error Handling

Common error codes:

| Error Code | Description |
|------------|-------------|
| `SSS_MISSING_REQD_ARGUMENT` | Missing required `prompt` parameter |
| `MUTUALLY_EXCLUSIVE_ARGUMENTS` | Both `presencePenalty` and `frequencyPenalty` > 0 with COHERE_COMMAND |
| `INVALID_MODEL_FAMILY_VALUE` | Invalid `modelFamily` value |
| `DOCUMENT_IDS_MUST_BE_UNIQUE` | Duplicate document IDs provided |
| `INAPPROPRIATE_CONTENT_DETECTED` | Response blocked by safety mode |
| `MAXIMUM_PARALLEL_REQUESTS_LIMIT_EXCEEDED` | More than 5 parallel requests |
| `RESPONSE_FORMAT_HAS_INVALID_JSON_SCHEMA` | Invalid JSON schema in responseFormat |

```javascript
try {
    const response = llm.generateText({
        prompt: userInput
    });
    // Handle response
} catch (e) {
    log.error('LLM Error', e.name + ': ' + e.message);
    if (e.name === 'INAPPROPRIATE_CONTENT_DETECTED') {
        // Handle content safety issue
    }
}
```

## Best Practices

1. **Validate AI Outputs**: LLMs may generate inaccurate content. Always validate responses before using them in business logic.

2. **Monitor Usage**: Use `getRemainingFreeUsage()` to track quota and implement fallback strategies.

3. **Chunk Data Appropriately**: For RAG, balance document size - too little context yields poor answers, too much risks token limits.

4. **Handle Timeouts**: Set appropriate timeout values and handle timeout errors gracefully.

5. **Use Preambles**: Provide clear system instructions via preamble for consistent behavior.

6. **Prompt Engineering**: Quality depends heavily on prompt design. Include clear instructions, examples, and context.

7. **Governance Awareness**: Each LLM call costs 100 governance units. Plan script design accordingly.

8. **Security**: Avoid sending sensitive PII or confidential data to LLMs.

## Availability

- **Prerequisites**: Server SuiteScript feature must be enabled
- **Regional Availability**: Only available in certain NetSuite regions (check Generative AI Availability)
- **EU Support**: Fully supported in European Union with all NetSuite system-supported languages

## Usage Modes

| Mode | Description | Best For |
|------|-------------|----------|
| **Free Tier** | Limited monthly quota | Testing, development, low-volume |
| **On-Demand** | Pay-as-you-go with OCI account | Medium volume, SuiteApps |
| **Dedicated AI Cluster** | Reserved OCI capacity | High-volume production |

## References

- [N/llm Module Documentation](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_9123730083.html)
- [SuiteScript 2.x Generative AI APIs](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_6193337927.html)
- [N/llm Module Script Samples](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_1015110627.html)
- [llm.generateText(options)](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_1014032554.html)
- [llm.ModelFamily](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_1014101247.html)
