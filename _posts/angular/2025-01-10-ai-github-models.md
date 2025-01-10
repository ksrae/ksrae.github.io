---
title: "How to prototyping with the Github AI models"
date: 2025-01-10 22:31:00 +0900
comments: true
categories: javascript
tags: [nodejs, githubmodels, gpt]
---

[Github Models](https://docs.github.com/en/github-models) 는 GitHub의 AI 및 머신 러닝 모델을 쉽게 관리하고 배포할 수 있도록 돕는 기능입니다. <br/>
GitHub에서 제공하는 모델 저장소나 API를 통해 개발자는 자신만의 머신 러닝 모델을 관리하거나 다른 사람이 만든 모델을 활용할 수 있습니다. <br/>
이러한 모델들은 다양한 인공지능 작업에 사용될 수 있으며, GitHub의 강력한 버전 관리 기능을 통해 모델의 버전 관리와 협업을 할 수 있습니다.<br/><br/>

이번 시간에는 이러한 Github Models를 가지고 AI를 만드는게 아니라 이미 제공되고 있는 다양한 ai 모델을 가지고 prototyping을 해보려고 합니다.<br/>
특히 Github Models 에는 gpt4oO나 llama3.3 등 최신 모델들을 prototyping 할 수 있도록 제공해주고 있습니다.<br/>
따라서 우리는 이러한 모델을 가지고 어떻게 ai를 호출하고 응답을 받는지를 테스트할 수 있는 간단한 코드를 작성해보려고 합니다.<br/>


## MarketPlace 방문하기
[Marketplace](https://github.com/marketplace/models)를 방문하면 이미 Text-ai 형태의 창을 볼 수 있습니다.<br/>
간단하게는 여기서 질문과 답변을 테스트 해볼 수 있습니다만, 우리는 개발을 목적으로 하므로 여기에서 더 깊이 들어가 봅시다.<br/>
prototyping 하고자 하는 모델을 하나 선택합니다.<br/>



## Token 발급하기
Github에 로그인하여 Access Token을 발급 받거나 이미 발급 받은 키가 반드시 필요합니다.<br/>
Get API Key 버튼을 클릭하면 발급부터 코드 작성까지 안내를 받을 수 있습니다.<br/><br/>

여기에서는 Language: javascript SDK는 OpenAI SDK를 기준으로 설명하겠습니다.<br/><br/>

또한 Azure AI 유료버전을 활용할 수도 있으나 무료 버전으로도 충분히 테스트 해볼 수 있으니 여기에서는 Azure AI는 다루지 않겠습니다.<br/>


## Token 등록하기
코드를 작성하기 전에 반드시 내 컴퓨터에 token을 등록해야 합니다.<br/>
Prototyping 이므로 재부팅하면 초기화 되므로 매번 token를 등록해야 합니다.<br/><br/>

가이드의 아래 항목을 참조하세요.

```
export GITHUB_TOKEN="<your-github-token-goes-here>"

If you're in powershell:
$Env:GITHUB_TOKEN="<your-github-token-goes-here>"

If you're using Windows command prompt:
set GITHUB_TOKEN=<your-github-token-goes-here>
```

## Node 설치와 packge.json 작성
Node를 설치하고 설치가 되었으면 프로젝트 폴더를 만들어 package.json을 작성합니다.<br/>
AI 마다 요구하는 dependencies가 다르므로 가이드 문서를 확인하고, 알맞은 항목을 추가해야 합니다.<br/><br/>

gpt4o인 경우에는 다음과 같이 작성합니다.

```
{
  "type": "module",
  "dependencies": {
    "openai": "latest"
  }
}
```

## Chatbot 코드 작성

가이드의 "Run a basic code sample" 과 같이 코드를 작성합니다.

```js
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

실행해보면 제대로 응답이 오는 것을 볼 수 있습니다.


## 파라미터로 질문 변경하기

위의 코드는 질문이 고정되어 있어 불편하므로, 사용자로부터 질문을 입력받아 처리하도록 수정합니다. 아래 코드는 명령줄 인수로 질문을 받아오는 예제입니다. <br/><br/>

이제 node index.js "질문" 이렇게 전달하면 질문에 대한 응답을 받아볼 수 있습니다.<br/>
만일 파라미터를 인식할 수 없는 에러가 발생하는 경우 node index.js -- "질문" 이렇게 하여 명확하게 지정할 수도 있습니다.<br/>


```js
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

## 기타 활용
이외에도 이미지 생성 요청 등 다양한 기능을 활용할 수 있습니다. <br/>
gpt-4o 외에도 llama3.3, mistral 등의 제공되는 다양한 모델을 테스트하여 자신에게 맞는 AI를 찾아보는 용도로 활용하기에 좋습니다.

