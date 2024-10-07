>🐈 `rtvi-ai` client repos have moved to [`pipecat-ai`](https://github.com/pipecat-ai)
- [rtvi-client-android](https://github.com/pipecat-ai/rtvi-client-android)
- [rtvi-client-android-daily](https://github.com/pipecat-ai/rtvi-client-android-daily)
- [rtvi-client-ios](https://github.com/pipecat-ai/rtvi-client-ios)
- [rtvi-client-ios-daily](https://github.com/pipecat-ai/rtvi-client-ios-daily)
- [rtvi-client-react-native-daily](https://github.com/pipecat-ai/rtvi-client-react-native-daily)
- [rtvi-client-web](https://github.com/pipecat-ai/rtvi-client-web)

# RTVI-AI: an open standard for Real-Time Voice [and Video] Inference

![rtvi cloud service sketch](images/rtvi-top-sketch.jpg)

This GitHub org contains:

  - open source SDK code
  - documentation for standard endpoint shapes, event messages, and data structures

Our goal here is to make it easy to build AI **voice-to-voice** and **real-time video** applications.

  - Applications developers should be able to write code that can use any inference service.
  - Inference services should be able to leverage open source for the complicated, client-side developer tooling needed for real-time multimedia.
  - Any developer should be able to trivially stand up real-time AI infrastructure for small-scale use, testing, or prototyping.

Jump straight to resources:
  - client SDKs
    - [Web and React](https://github.com/rtvi-ai/rtvi-client-web)
    - [iOS](https://github.com/rtvi-ai/rtvi-client-ios-daily)
    - [Android](https://github.com/rtvi-ai/rtvi-client-android-daily)
    - [React Native](https://github.com/rtvi-ai/rtvi-client-react-native-daily)
  - [API reference documentation](https://docs.rtvi.ai/introduction)
  - Web demo [source code](https://github.com/rtvi-ai/rtvi-web-demo) and [live demo](https://demo.dailybots.ai/)
  - Infrastructure
    - [Backend/infrastructure examples](https://github.com/rtvi-ai/rtvi-infra-examples)
    - Pipecat server-side RTVI implementation [complete source code](https://github.com/pipecat-ai/pipecat/blob/main/src/pipecat/processors/frameworks/rtvi.py)


# What client-side code looks like

Here's a voice-to-voice AI "hello world" in Javascript.

```javascript
//
// Start a multi-turn voice-to-voice session in a web app
//

import { VoiceClient } from "@realtime-ai";

function myTrackHandler(track, participant, voiceClient) {
    if (participant.local || track.kind !== 'audio') {
      return;
    }
    let audioElement = document.createElement('audio');
    audioElement.srcObject = new MediaStream([track]);
    document.body.appendChild(audioElement);
    audioElement.play();
}

const voiceClient = new VoiceClient({
    baseUrl,
    enableMic: true,
    callbacks: {
      trackStarted: myTrackHandler,
    }
});

voiceClient.start();
```

The `baseUrl` parameter determines what inference service the client uses. This simple client uses the inference provider's defaults for:

  - which AI model or models are used for audio input, language processing, and audio output
  - system prompt and context management
  - configuration of the workflow "pipeline"
  - what features are automatically enabled
  

# The real-time AI stack

Conceptually, a voice-to-voice AI application looks like this.

![rtvi voice-to-voice loop](images/rtvi-voice-loop.jpg)

This is a conceptual diagram rather than an architecture diagram. The voice loop in the cloud might use one, three, or many models to perform audio input processing, language-level processing, and audio output. Real-world voice AI applications often use a collection of models, combined with application code that orchestrates how the models are used.

Here's a diagram for a voice AI application that collects information from a patient prior to a healthcare visit. This workflow follows a flexible conversational script and makes function calls out to an external system.

![rtvi tool use loop](images/rtvi-tool-use.jpg)

Good standards and SDKs make easy things easy, and hard things possible. A good SDK design is flexible enough to support future capabilities and the evolution of related standards. Finally, we live in a cross-platform world; the Web, iOS, Android, Linux, macOS, and Windows all need first-class support.

Here are the client-side "functional layers" of the RTVI stack.

![rtvi tool use loop](images/rtvi-client-stack.jpg)

On the cloud side, implementors have a lot of flexibility. Cloud implementations need to send and receive RTVI events and data structures. But the details of how that happens don't matter to client code!

One way to think about this is that the purpose of a standard like RTVI is to promote interoperability (and lower friction, and increase developer productivity) by specifically constraining client-side implementations but not server-side architectures.

Having said that, reference implementations are helpful. The [Pipecat open source library](https://github.com/pipecat-ai/pipecat) implements RTVI events and data structures.

Here's one way to build RTVI infrastructure using Pipecat or a similar real-time AI library.

![rtvi pipecat infrastructure example](images/rtvi-pipecat.jpg)

# Architecture — events and services

RTVI is a set of abstractions and a messages format for communicating between inference servers and clients.

The RTVI architecture defines:
  - A small number of events that allow clients and servers to communicate about audio and video streams, session state, metrics, and errors.
  - A generic "services" mechanism
    - Inference servers expose "services."
    - Services are abstract containers for configuration and actions. In a typical implementation, a service will map to an element in a real-time orchestration pipeline. But servers can define and implement services however they want to. RTVI defines an interface for service configuration and actions, but does not define any specific services.

## Events

RTVI defines the following standard event messages. Client implementations can expose these via event handlers or callbacks.

  - transport-state-changed
  - connected
  - disconnected
  - client-ready
  - bot-ready
  - config
  - bot-disconnected
  - generic-message
  - error
  - track-started
  - track-stopped
  - metrics
  - user-started-speaking
  - user-stopped-speaking
  - bot-started-speaking
  - bot-stopped-speaking
  - user-transcription
  - bot-transcription

See the [RTVI JavaScript SDK documentation](https://docs.rtvi.ai/api-reference/messages-and-events) for a client-side view of events and callbacks.

## Services

An inference server will usually expose a set of services that a client can configure and trigger actions on.

For example, a typical voice-to-voice inference server will define `stt`, `llm`, and `tts` services.

Each of these services will have options that can be configured from the client or via REST APIs. For example, the `llm` service will allow a specific `model` to be specified.

Here is an example services configuration.

```
[
  {
    "service": "vad",
    "options": [{ "name": "params", "value": { "stop_secs": 3.0 } }]
  },
  {
    "service": "tts",
    "options": [
      { "name": "voice", "value": "79a125e8-cd45-4c13-8a67-188112f4dd22" }
    ]
  },
  {
    "service": "llm",
    "options": [
      {
        "name": "model",
        "value": "meta-llama/Meta-Llama-3.1-8B-Instruct-Turbo"
      },
      {
        "name": "initial_messages",
        "value": [
          {
            "role": "system",
            "content": `You are a assistant called ExampleBot. You can ask me anything.
              Keep responses brief and legible.
              Your responses will converted to audio. Please do not include any special characters in your response other than '!' or '?'.
              Start by briefly introducing yourself.`
          }
        ]
      },
    ]
  }
]
```

Services may expose actions, as well.

For example, a `tts` service will typically offer a `say` action.

It's important to note that the RTVI standard defines the services and actions mechanisms, but not specific services and actions.

The Open Source RTVI SDKs provide generic methods like [`updateConfig()`](https://docs.rtvi.ai/api-reference/client-methods#updateconfig) and [`action()`](https://docs.rtvi.ai/api-reference/client-methods#action).

Calling a `tts:say` action with the reference RTVI JavaScript SDK looks like this:

```
response = await voiceClient.action({
  service: "tts",
  action: "say",
  arguments: [{ name: "text", value: "Say 'Hello world!'" }],
});
```

However, RTVI clients can certainly be written or extended to provide better developer ergonomics for specific platforms or use cases! The reference RTVI JavaScript SDK offers a [helper libraries](https://docs.rtvi.ai/api-reference/helpers/introduction) mechanism that can provide more convenient and type-safe interfaces for service configuration and actions.

## Taking a step back — building blocks

The RTVI standard tries to be as flexible as possible, while also being shaped to support today's core real-time AI use cases:

  - voice-to-voice inference loops
  - real-time video generation
  - real-time computer vision
  - multi-model and multi-model orchestration

An RTVI implementation that can be used in production will generally need to provide a substantial subset of the following building blocks:

  - low-latency audio and video codecs and transport
  - text input and output
  - image input and output
  - tts -> llm -> stt pipeline configuration
  - llm context management
  - effective phrase endpointing
  - low-latency, llm context-aware, interruption handling 
  - function calling
  - builtin tool extensions
  
# Contributing

If you'd like to contribute to RTVI, join the [Pipecat Discord](https://discord.com/invite/pipecat) or submit PRs to the various repos here.

We welcome all contributions, including additional RTVI client SDKs, transport providers, and additions to the Pipecat server-side implementation of the RTVI standard.




