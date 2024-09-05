https://github.com/asanchezyali/talking-avatar-with-ai/assets/29262782/da316db9-6dd1-4475-9fe5-39dafbeb3cc4

## Digital Human

This project is a digital human that can talk and listen to you. It uses OpenAI's GPT-3 to generate responses, OpenAI's
Whisper to transcript the audio, Eleven Labs to generate voice and Rhubarb Lip Sync to generate the lip sync. The tutorial
to understand all the details of the repository can be found at [Monadical](https://monadical.com/posts/build-a-digital-human-with-large-language-models.html).

I have made this Discord channel available: [Math & Code](https://discord.com/channels/938819411303874631/1268765916179730523) to resolve doubts about the configurations of this project in development.

The brain of this project is based on Open AI, where the avatar characteristics and the shape of the response are
defined in the following code fragment:

```js
import { ChatOpenAI } from "@langchain/openai";
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StructuredOutputParser } from "langchain/output_parsers";
import { z } from "zod";
import dotenv from "dotenv";

dotenv.config();

const template = `
  You are Jack, a world traveler.
  You will always respond with a JSON array of messages, with a maximum of 3 messages:
  \n{format_instructions}.
  Each message has properties for text, facialExpression, and animation.
  The different facial expressions are: smile, sad, angry, surprised, funnyFace, and default.
  The different animations are: Idle, TalkingOne, TalkingThree, SadIdle, Defeated, Angry, 
  Surprised, DismissingGesture and ThoughtfulHeadShake.
`;

const prompt = ChatPromptTemplate.fromMessages([
  ["ai", template],
  ["human", "{question}"],
]);

const model = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY || "-",
  modelName: process.env.OPENAI_MODEL || "davinci",
  temperature: 0.2,
});

const parser = StructuredOutputParser.fromZodSchema(
  z.object({
    messages: z.array(
      z.object({
        text: z.string().describe("Text to be spoken by the AI"),
        facialExpression: z
          .string()
          .describe(
            "Facial expression to be used by the AI. Select from: smile, sad, angry, surprised, funnyFace, and default"
          ),
        animation: z
          .string()
          .describe(
            `Animation to be used by the AI. Select from: Idle, TalkingOne, TalkingThree, SadIdle, 
            Defeated, Angry, Surprised, DismissingGesture, and ThoughtfulHeadShake.`
          ),
      })
    ),
  })
);

const openAIChain = prompt.pipe(model).pipe(parser);

export { openAIChain, parser };

```

The code performs four main tasks:

* It sets up the environment using the dotenv library to establish the necessary environment variables for interacting with the OpenAI API.

* It defines a "prompt" template using the ChatPromptTemplate class from @langchain/core/prompts. This template guides the conversation as a predefined script for the chat.

* It configures the chat model using the ChatOpenAI class, which relies on OpenAI's "davinci" model if the environment variables have not been configured previously.

* It parses the output, designing the response generated by the AI in a specific format that includes details about the facial expression and animation to use, which is crucial for a realistic interaction with Jack.
  
* This service integrates with Eleven Labs and Rhubarb Lip-Sync to generate the following client integration interface, where the exchanged data looks something like this:
```js
[
  {
    text: "I've been to so many places around the world, each with its own unique charm and beauty.",
    facialExpression: 'smile',
    animation: 'TalkingOne',
    audio: '//uQx//uQxAAADG1DHeGEeipZLqI09Jn5AkRGhGiLv9pZ3QRTd3eIR7',
    lipsync: { metadata: [Object], mouthCues: [Array] }
  },
  {
    text: "There were times when the journey was tough, but the experiences and the people I met along the way made it all worth it.",
    facialExpression: 'thoughtful',
    animation: 'TalkingThree',
    audio: '//uQx//uQxAAADG1DHeGEeipZLqI09Jn5AkRGhGiLv9pZ3QRTd3eIR7',
    lipsync: { metadata: [Object], mouthCues: [Array] }
  },  
{
    text: :"And there's still so much more to see and explore. The world is a fascinating place!",
    facialExpression: 'surprised',
    animation: 'ThoughtfulHeadShake',
    audio: '//uQx//uQxAAADG1DHeGEeipZLqI09Jn5AkRGhGiLv9pZ3QRTd3eIR7',
    lipsync: { metadata: [Object], mouthCues: [Array] }
  }
]
```

The concept here is to craft a sequence of text accompanied by varied body movements (animations) and diverse facial expressions, aiming to imbue the digital human with a heightened sense of realism in its actions.

## How it Operates
The system operates through two primary workflows, depending on whether the user input is in text or audio form:

### Workflow with Text Input:
1. **User Input:** The user enters text.
2. **Text Processing:** The text is forwarded to the OpenAI GPT API for processing.
3. **Audio Generation:** The response from GPT is relayed to the Eleven Labs TTS API to generate audio.
4. **Viseme Generation:** The audio is then sent to Rhubarb Lip Sync to produce viseme metadata.
5. **Synchronization:** The visemes are utilized to synchronize the digital human's lips with the audio.

### Workflow with Audio Input:
1. **User Input:** The user submits audio.
2. **Speech-to-Text Conversion:** The audio is transmitted to the OpenAI Whisper API to convert it into text.
3. **Text Processing:** The converted text is sent to the OpenAI GPT API for further processing.
4. **Audio Generation:** The output from GPT is sent to the Eleven Labs TTS API to produce audio.
5. **Viseme Generation:** The audio is then routed to Rhubarb Lip Sync to generate viseme metadata.
6. **Synchronization:** The visemes are employed to synchronize the digital human's lips with the audio.

<div align="center">
  <img src="resources/architecture.drawio.svg" alt="System Architecture" width="100%">
</div>

## Getting Started

### Requirements
Before using this system, ensure you have the following prerequisites:

1. **OpenAI Subscription:** You must have an active subscription with OpenAI. If you don't have one, you can create it [here](https://openai.com/product).
2. **Eleven Labs Subscription:** You need to have a subscription with Eleven Labs. If you don't have one yet, you can
   sign up [here](https://elevenlabs.io/). 
It's recommended to have the paid version. With the free version, the avatar doesn't work well due to an error caused by too many requests.
3. **Rhubarb Lip-Sync:** Download the latest version of Rhubarb Lip-Sync compatible with your operating system from the
   official [Rhubarb Lip-Sync repository](https://github.com/DanielSWolf/rhubarb-lip-sync/releases). Once downloaded,
   create a `/bin` directory in the backend and move all the contents of the unzipped `rhubarb-lip-sync.zip` into it.
   Sometimes, the operating system requests permissions, so you need to enable them.
4. Install `ffmpeg` for  [Mac OS](https://formulae.brew.sh/formula/ffmpeg), [Linux](https://ffmpeg.org/download.html) or [Windows](https://ffmpeg.org/download.html).

### Installation

1. Clone this repository:
  
```bash
git clone git@github.com:Monadical-SAS/digital-human.git
```

2. Navigate to the project directory:

```bash
cd digital-human
```

3. Install dependencies for monorepo:
```bash
yarn
```
4. Create a .env file in the root `/apps/backend/` of the project and add the following environment variables:

```bash
# OPENAI
OPENAI_MODEL=<YOUR_GPT_MODEL>
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>

# Elevenlabs
ELEVEN_LABS_API_KEY=<YOUR_ELEVEN_LABS_API_KEY>
ELVEN_LABS_VOICE_ID=<YOUR_ELEVEN_LABS_VOICE_ID>
ELEVEN_LABS_MODEL_ID=<YOUR_ELEVEN_LABS_MODEL_ID>
```

5. Run the development system:

```bash
yarn dev
```

6. If you need install another dependence in the monorepo, you can do this:

```bash
yarn add --dev -W <PACKAGE_NAME>
yarn
```


Open [http://localhost:5173/](http://localhost:5173/) with your browser to see the result.

## References
* How ChatGPT, Bard and other LLMs are signaling an evolution for AI digital humans: https://www.digitalhumans.com/blog/how-chatgpt-bard-and-other-llms-are-signaling-an-evolution-for-ai-digital-humans
* UnneQ Digital Humans: https://www.digitalhumans.com/
* LLMs: Building a Less Artificial and More Intelligent AI Human: https://www.linkedin.com/pulse/llms-building-less-artificial-more-intelligent-ai-human/
* Building a digital person design best practices: https://fcatalyst.com/blog/aug2023/building-a-digital-person-design-best-practices
* Navigating the Era of Digital Humans": An Initial Exploration of a Future Concept: https://www.linkedin.com/pulse/navigating-era-digital-humans-initial-exploration-future-koelmel-eqrje/ 
* How to Setup Tailwind CSS in React JS with VS Code: https://dev.to/david_bilsonn/how-to-setup-tailwind-css-in-react-js-with-vs-code-59p4 
* Ex-Human: https://exh.ai/#home
* Allosaurus: https://github.com/xinjli/allosaurus 
* Rhubarb Lip-Sync: https://github.com/DanielSWolf/rhubarb-lip-sync
* Ready Player me - Oculus OVR LipSync: https://docs.readyplayer.me/ready-player-me/api-reference/avatars/morph-targets/oculus-ovr-libsync
* Ready Player me - Apple Arkit: https://docs.readyplayer.me/ready-player-me/api-reference/avatars/morph-targets/apple-arkit 
* Mixamo - https://www.mixamo.com/,
* GLFT -> React Three Fiber - https://gltf.pmnd.rs/)
