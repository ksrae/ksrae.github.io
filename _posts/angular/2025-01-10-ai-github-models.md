---
title: "How to prototyping with the Github AI models"
date: 2025-01-10 22:31:00 +0900
comments: true
categories: javascript
tags: [nodejs, githubmodels, gpt]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Github-AI-models%EB%A1%9C-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85-%EB%A7%8C%EB%93%A4%EA%B8%B0)
<br/>

GitHub Models offer a streamlined approach to managing and deploying AI and machine learning models directly within the GitHub ecosystem. Developers can leverage GitHub's model repository and API to manage their own models or utilize pre-existing ones. These models facilitate a wide range of AI tasks, benefiting from GitHub's robust version control and collaboration features.

This exploration shifts the focus from creating AI models to prototyping with readily available AI models hosted on GitHub Models. Specifically, GitHub Models provides access to cutting-edge models like GPT-4o and Llama 3.3, enabling rapid experimentation. The following sections outline how to invoke these AI models and process their responses through code.

## Accessing the Marketplace

The [GitHub Marketplace](https://github.com/marketplace/models) provides a user interface for interacting with text-based AI models. While the marketplace offers a basic chat-like interface for initial testing, a deeper dive into the development aspects is warranted. To proceed, select a model to prototype.

## Obtaining an Access Token

A GitHub Access Token is required to interact with the models programmatically. You can either generate a new token or use an existing one. Clicking the "Get API Key" button in the marketplace provides guidance on token generation and code setup.

The following examples primarily focus on the Javascript SDK, utilizing the OpenAI SDK as a baseline.

While Azure AI provides a paid alternative, the free tier is sufficient for initial testing. Consequently, Azure AI will not be covered in this guide.

## Registering the Token

Prior to running any code, the access token must be registered in your environment. For prototyping purposes, this registration is temporary and will be lost upon system reboot. Therefore, the token must be registered each time you start a new session.

Refer to the following instructions:

```
export GITHUB_TOKEN="<your-github-token-goes-here>"

If you're in powershell:
$Env:GITHUB_TOKEN="<your-github-token-goes-here>"

If you're using Windows command prompt:
set GITHUB_TOKEN=<your-github-token-goes-here>
```

## Setting up Node and package.json

Install Node.js, then create a project folder and initialize a `package.json` file. The required dependencies vary based on the specific AI model. Consult the model's documentation for the necessary additions.

For GPT-4o, the `package.json` file should look like this:

```json
{
  "type": "module",
  "dependencies": {
    "openai": "latest"
  }
}
```

## Implementing the Chatbot Code

Implement the chatbot functionality as demonstrated in the "Run a basic code sample" guide.

```jsx
import OpenAI from "openai";

const token = process.env["GITHUB_TOKEN"];
const endpoint = "https://models.inference.ai.azure.com";
const modelName = "gpt-4o";

export async function main() {

  const client = new OpenAI({ baseURL: endpoint, apiKey: token });

  const response = await client.chat.completions.create({
    messages: [
        { role:"system", content: "You are a helpful assistant." },
        { role:"user", content: "What is the capital of France?" }
      ],
      temperature: 1.0,
      top_p: 1.0,
      max_tokens: 1000,
      model: modelName
    });

  console.log(response.choices[0].message.content);
}

main().catch((err) => {
  console.error("The sample encountered an error:", err);
});
```

Executing this code should produce the expected response from the AI model.

## Parameterizing the Query

The preceding code uses a static query. To enable dynamic querying, modify the code to accept user input. The following example demonstrates how to pass the query as a command-line argument.

Now, execute the script with the command `node index.js "your question"`. This will pass "your question" to the AI model and display the response. If you encounter issues with argument parsing, use `node index.js -- "your question"` to explicitly specify the argument.

```jsx
import OpenAI from "openai/index.mjs";

// Get the user query from the command-line arguments (args[2])
const userQuery = process.argv[2];

const token = process.env["GITHUB_TOKEN"];
const endpoint = "https://models.inference.ai.azure.com";
const modelName = "gpt-4o";

export async function main() {

  const client = new OpenAI({ baseURL: endpoint, apiKey: token });

  const response = await client.chat.completions.create({
    messages: [
        { role: "system", content: "You are a helpful assistant." },
        { role: "user", content: userQuery } // Use the user query passed from args
      ],
      temperature: 1.0,
      top_p: 1.0,
      max_tokens: 1000,
      model: modelName
    });

  console.log(response.choices[0].message.content);
}

main().catch((err) => {
  console.error("The sample encountered an error:", err);
});
```

## Further Exploration

Beyond text generation, these models support a variety of functionalities, including image creation. Experiment with different models like Llama 3.3 and Mistral to determine the AI best suited for your specific needs.