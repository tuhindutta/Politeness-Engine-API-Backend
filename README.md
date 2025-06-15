# Politeness-Engine-API-Backend

## Overview
The backend of Speech Refiner is a Flask-based REST API designed to process audio files uploaded from the frontend application. It performs two main tasks: transcribing the speech from audio and refining the transcribed text using a Large Language Model (LLM). The backend is designed for modularity, scalability, and secure integration with production-ready platforms.

## Architecture Overview
The backend system consists of three primary files:
- main.py
This is the entry point of the Flask application. It defines the `/upload` route, handles audio file uploads, manages file storage, and connects the processing pipeline using utility classes.
- utils.py
Contains the core logic split into two major classes:
  - Transcription: Utilizes the Google Speech Recognition API (via the `speech_recognition` Python package) to convert spoken content in an audio file into text.
  - LLM: Sends the transcribed text to a Groq-hosted large language model API along with a prompt, and receives a refined or enhanced version of the transcription.
- requirements.txt
Lists all the Python dependencies required to run the backend. This includes packages for Flask-based API handling, audio processing, LLM communication, and server deployment.

## Endpoint Specification
- POST `/upload`
This is the primary and only endpoint exposed by the backend. It accepts an audio file from the client, processes it through the transcription and LLM pipeline, and returns both the raw transcription and the refined output.
  - Request Format
    - Method: POST
    - Content-Type: multipart/form-data
    - Form field:
      - `audio`: The uploaded audio file (formats like WAV or FLAC are recommended for best compatibility).
  - Processing Flow
    1. The uploaded audio file is received and securely saved to a temporary `uploads/` directory.
    2. It is then read into memory using `scipy.io.wavfile`.
    3. The `Transcription` class processes the audio data to generate the raw text.
    4. The transcription is truncated to the first 100 words to control LLM token usage.
    5. The truncated text is passed to the `LLM` class, which forwards it to the **Groq LLM** API.
    6. The LLM returns a refined or enhanced response based on the given prompt.
    7. The temporary file is deleted, and a JSON response is sent back to the frontend.
  - Successful Response (`200 OK`)
    The response contains the following fields:
    - input: The transcribed text, truncated to the first 100 words.
    - output: The LLM-generated or enhanced output.
  - Error Responses
    - If the audio field is missing in the request, a `400 Bad Request` is returned with a descriptive error message.
    - If an unexpected exception occurs (e.g., file format issues, API failures), a `500 Internal Server Error` is returned along with the exception message for debugging.

