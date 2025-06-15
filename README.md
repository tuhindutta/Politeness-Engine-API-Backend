# 🌐 Politeness-Engine-API-Backend

## 📌 Overview
The backend of Speech Refiner is a Flask-based REST API designed to process audio files uploaded from the frontend application. It performs two main tasks: transcribing the speech from audio and refining the transcribed text using a Large Language Model (LLM). The backend is designed for modularity, scalability, and secure integration with production-ready platforms.

## 🧱 Architecture Overview
The backend system consists of three primary files:
- main.py
This is the entry point of the Flask application. It defines the `/upload` route, handles audio file uploads, manages file storage, and connects the processing pipeline using utility classes.
- utils.py
Contains the core logic split into two major classes:
  - 🗣️ Transcription: Utilizes the Google Speech Recognition API (via the `speech_recognition` Python package) to convert spoken content in an audio file into text.
  - 💬 LLM: Sends the transcribed text to a Groq-hosted large language model API along with a prompt, and receives a refined or enhanced version of the transcription.
- requirements.txt
Lists all the Python dependencies required to run the backend. This includes packages for Flask-based API handling, audio processing, LLM communication, and server deployment.

## 🔗 Endpoint Specification
- POST `/upload`
This is the primary and only endpoint exposed by the backend. It accepts an audio file from the client, processes it through the transcription and LLM pipeline, and returns both the raw transcription and the refined output.
  - 📝 Request Format
    - Method: POST
    - Content-Type: multipart/form-data
    - Form field:
      - `audio`: The uploaded audio file (formats like WAV or FLAC are recommended for best compatibility).
  - 🔄 Processing Flow
    1. The uploaded audio file is received and securely saved to a temporary `uploads/` directory.
    2. It is then read into memory using `scipy.io.wavfile`.
    3. The `Transcription` class processes the audio data to generate the raw text.
    4. The transcription is truncated to the first 100 words to control LLM token usage.
    5. The truncated text is passed to the `LLM` class, which forwards it to the **Groq LLM** API.
    6. The LLM returns a refined or enhanced response based on the given prompt.
    7. The temporary file is deleted, and a JSON response is sent back to the frontend.
  - ✅ Successful Response (`200 OK`)
    The response contains the following fields:
    - input: The transcribed text, truncated to the first 100 words.
    - output: The LLM-generated or enhanced output.
  - ❌ Error Responses
    - If the audio field is missing in the request, a `400 Bad Request` is returned with a descriptive error message.
    - If an unexpected exception occurs (e.g., file format issues, API failures), a `500 Internal Server Error` is returned along with the exception message for debugging.

## 🚦 Rate Limiting
To ensure fair usage and prevent abuse, the backend implements rate limiting based on the client's IP address. This is handled using the `Flask-Limiter` package.
- Limit Per IP:
  - Maximum 10 requests per minute
  - Maximum 150 requests per day
- Rate Limit Storage:
A Redis instance is used to persist request counts and rate limiting state. This is configured via the `REDIS_URI` environment variable. If no external Redis URI is provided, the app defaults to using in-memory storage.
- Redis Hosting:
Redis is deployed separately using [Upstash](https://upstash.com/), a serverless Redis provider, and the URI is securely injected via environment variables.

## 🧠 LLM API Integration
The backend interacts with a Groq-hosted Large Language Model (LLM) to enhance or summarize the transcribed text.
- Provider: Groq
- API Access: The API key is provided securely through an environment variable named `GROQ_API_KEY`.
- Usage: The transcription is sent to the Groq API through the `LLM.query_llm()` method along with a prompt defined in the codebase. The response is processed and returned to the frontend.
- Security: The API key is never exposed on the client side and is kept securely within the backend environment configuration.

## 🚀 Deployment Overview
- Hosting Platform:
The backend is deployed on [Render](https://render.com/), a cloud platform for hosting web services. The Flask app is served using *Gunicorn*, a production-grade WSGI HTTP server.
- Environment Variables:
Render allows injecting secure environment variables into the deployed application. The following are configured:
  - `GROQ_API_KEY` – The LLM API key for authentication with the Groq API.
  - `REDIS_URI` – The Redis connection string for rate limiting, pointing to an Upstash instance.
- Audio Upload Handling:
The backend temporarily stores incoming audio files in the `uploads/` folder, which is auto-created at runtime if it doesn’t exist. Files are deleted immediately after processing to ensure security and avoid disk clutter.

## 📦 Dependency List
The backend depends on the following Python libraries:
```txt
﻿Flask==3.1.1
flask-cors==6.0.0
Flask-Limiter==3.11.0
numpy==2.0.2
redis>=3.0
requests==2.32.3
scipy==1.13.1
soundfile==0.13.1
SpeechRecognition==3.14.3
gunicorn
```

## Final Notes
- The backend is secure, stateless, and compatible with cross-platform frontend interfaces (like Electron, web clients, or mobile apps).
- Designed to be production-ready, with rate limits, environment-based secrets, and external service integration fully isolated from the frontend.
- All audio processing and cleanup is handled server-side, ensuring minimal client-side dependency and strong privacy controls.
