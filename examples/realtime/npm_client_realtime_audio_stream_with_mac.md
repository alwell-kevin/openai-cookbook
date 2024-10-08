## OpenAI Realtime API Integration for Node.js

This example will guide you through the process of integrating [the OpenAI Realtime platform](https://platform.openai.com/docs/api-reference/realtime) into a Node.js application that runs locally on mac (or other device) for development. It serves as a gateway for developers looking to interact with OpenAI's GPT models in real-time, using audio input and output to create a conversational assistant.

In this tutorial, developers will learn how to use the `@openai/realtime-api-beta` client to connect to OpenAI's API, stream audio input, and receive responses in real time. This integration provides a way to implement dynamic interactions that utilize GPT's natural language processing capabilities in audio form, making it well suited for conversational AI, live assistant applications, and other real-time use cases. If you want an example using a raw websocket, that exists in this same [realtime directory](./ws_realtime_audio_stream_with_mac.md). 

This code sample is also a great way to create a development environment that allows developers to test their own use cases locally, facilitating rapid prototyping and iteration.

### Machine Dependencies

To successfully run this application, the following machine dependencies are required:

(This guide was developed on a mac for mac users. The machine dependency is mostly related to audio input/output utilities resident on mac. Other components are reusable.)

1. **Node.js** (version 14 or higher): Required to run the JavaScript code.
2. **npm** (Node Package Manager): To install the necessary Node.js packages (`@openai/realtime-api-beta`, `mic`, `dotenv`, `speaker`).
3. **Sox**: The application uses the `mic` package, which relies on the Sox command line utility (`rec`) to capture audio. Install via Homebrew on macOS:
   ```sh
   brew install sox
   ```
4. **Microphone**: A working microphone is required for capturing live audio.
5. **Speakers or Headphones**: Needed to play back the audio response from the OpenAI API.
6. **Environment Variables**: You need to set the `OPENAI_API_KEY` in your environment to authenticate with OpenAI's API. You can add it to a `.env` file:
   ```
   OPENAI_API_KEY=your_openai_api_key_here
   ```
7. **Homebrew** (for macOS users): To install system dependencies like Sox.
8. **Network Connection**: A stable internet connection is required for communication with the OpenAI API.

### Summary

This tutorial is valuable for developers who want to:
- Understand how to use [OpenAI’s Realtime API](https://platform.openai.com/docs/api-reference/realtime-client-events) for conversational AI applications.
- Gain hands-on experience with audio streaming and real-time processing in Node.js using [the latest npm module](https://github.com/openai/openai-realtime-api-beta/tree/main) for ease and simplicity.
- Create an interactive experience that takes full advantage of GPT's capabilities for spoken dialogue.

By providing a practical example of connecting and interacting with OpenAI, this tutorial serves as a foundational guide for building real-time AI-driven solutions. This example is designed to assist developers in quickly setting up a voice-based AI assistant using Node.js.

### Step-by-Step Guide for Implementing this Code

1. **Install Node.js and npm**
   - Make sure you have Node.js (version 14 or higher) and npm installed on your machine.

2. **Clone or Create the Project**
   - Create a new project directory and navigate into it:
   ```sh
   mkdir openai-realtime-api
   cd openai-realtime-api
   ```

3. **Install Required Dependencies**
   - Install the necessary packages using npm:
   ```sh
   npm init -y
   npm install @openai/realtime-api-beta mic dotenv speaker
   ```

4. **Install Sox**
   - Sox is required for capturing microphone input. Install it using Homebrew (for macOS):
   ```sh
   brew install sox
   ```

5. **Create a `.env` File**
   - Create a `.env` file in the root of your project directory and add your OpenAI API key:
   ```
   OPENAI_API_KEY=your_openai_api_key_here
   ```

6. **Set Up the Code**
   - Copy the provided JavaScript code below into a file named `app.mjs` in your project directory.

7. **Run the Application**
   - Run the application with Node.js:
   ```sh
   node app.mjs
   ```

8. **Verify Dependencies**
   - Ensure Sox is available in your system path. If you encounter an error related to `rec` command not found, add the following line to your code to adjust the path:
   ```javascript
   process.env.PATH = `${process.env.PATH}:/usr/local/bin`;
   ```

### Example Code Using the OpenAI Realtime API Client

Below is an example of using the OpenAI Realtime API client to create a real-time voice-based assistant. This code captures audio from the microphone, sends it to OpenAI's API, and plays the response.

```javascript
import { RealtimeClient } from '@openai/realtime-api-beta';
import mic from 'mic';
import { Readable } from 'stream';
import Speaker from 'speaker';
import dotenv from 'dotenv';

dotenv.config();

const API_KEY = process.env.OPENAI_API_KEY;

if (!API_KEY) {
  console.error('Please set your OPENAI_API_KEY in your environment variables.');
  process.exit(1);
}

const client = new RealtimeClient({
  apiKey: API_KEY,
  model: 'gpt-4o-realtime-preview-2024-10-01',
});

let micInstance;
let speaker;

async function main() {
  try {
    console.log('Attempting to connect...');
    await client.connect();
    startAudioStream();
    console.log('Connection established successfully.');
  } catch (error) {
    console.error('Error connecting to OpenAI Realtime API:', error);
    console.log('Connection attempt failed. Retrying in 5 seconds...');
    setTimeout(main, 5000);
  }
}

main();

client.on('conversation.item.completed', ({ item }) => {
  console.log('Conversation item completed:', item);
  
  if (item.type === 'message' && item.role === 'assistant' && item.formatted && item.formatted.audio) {
    console.log('Playing audio response...');
    playAudio(item.formatted.audio);
  } else {
    console.log('No audio content in this item.');
  }
});

// BEGIN MANAGE AUDIO INTERFACES

function startAudioStream() {
  try {
    micInstance = mic({
      rate: '24000',
      channels: '1',
      debug: false,
      exitOnSilence: 6,
      fileType: 'raw',
      encoding: 'signed-integer',
    });

    const micInputStream = micInstance.getAudioStream();

    micInputStream.on('error', (error) => {
      console.error('Microphone error:', error);
    });

    micInstance.start();
    console.log('Microphone started streaming.');

    let audioBuffer = Buffer.alloc(0);
    const chunkSize = 4800; // 0.2 seconds of audio at 24kHz

    micInputStream.on('data', (data) => {
      audioBuffer = Buffer.concat([audioBuffer, data]);

      while (audioBuffer.length >= chunkSize) {
        const chunk = audioBuffer.slice(0, chunkSize);
        audioBuffer = audioBuffer.slice(chunkSize);

        const int16Array = new Int16Array(chunk.buffer, chunk.byteOffset, chunk.length / 2);

        try {
          client.appendInputAudio(int16Array);
        } catch (error) {
          console.error('Error sending audio data:', error);
        }
      }
    });

    micInputStream.on('silence', () => {
      console.log('Silence detected, creating response...');
      try {
        client.createResponse();
      } catch (error) {
        console.error('Error creating response:', error);
      }
    });
  } catch (error) {
    console.error('Error starting audio stream:', error);
  }
}

function playAudio(audioData) {
  try {
    if (!speaker) {
      speaker = new Speaker({
        channels: 1,
        bitDepth: 16,
        sampleRate: 24000,
      });
    }

    // Convert Int16Array to Buffer
    const buffer = Buffer.from(audioData.buffer);

    // Create a readable stream from the buffer
    const readableStream = new Readable({
      read() {
        this.push(buffer);
        this.push(null);
      },
    });

    // Pipe the stream to the speaker
    readableStream.pipe(speaker);
    console.log('Audio sent to speaker for playback. Buffer length:', buffer.length);

    // Handle the 'close' event to recreate the speaker for the next playback
    speaker.on('close', () => {
      console.log('Speaker closed. Recreating for next playback.');
      speaker = null;
    });
  } catch (error) {
    console.error('Error playing audio:', error);
  }
}

// END MANAGE AUDIO INTERFACES
```

By following these steps, developers new to integrating real-time APIs can easily set up a voice-based assistant that interacts with OpenAI's models, providing an engaging example of real-time conversational AI.