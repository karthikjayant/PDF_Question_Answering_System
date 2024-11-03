# PDF Question-Answering Backend Service

This project provides a backend solution for real-time question-answering on PDF documents. Users can upload PDFs and interactively ask questions about the content through a WebSocket, receiving answers based on the document. Built with FastAPI, PyMuPDF, and NLP models, the service includes Redis-based rate limiting to efficiently manage requests, making it a robust tool for dynamic document analysis.

## Overview

This backend service lets users upload PDF files, ask questions in real-time through WebSocket connections, and receive context-aware answers. Session-based context handling supports follow-up questions, while rate limiting controls usage to prevent abuse.

## Features

- **PDF Upload**: Allows users to upload PDFs, extract their text, and store relevant metadata.
- **Real-Time Q&A with WebSocket**: Enables interactive question-answering with session-based context.
- **Rate Limiting**: Ensures efficient handling by limiting the number of requests per user.

## Technologies Used

- **FastAPI**: Backend framework for managing HTTP requests and WebSocket connections.
- **Redis**: Manages rate limiting and session tracking.
- **PyMuPDF**: Extracts text from PDF documents.
- **LangChain and LlamaIndex**: Handles natural language processing for accurate responses based on the PDF content.

## Setup Instructions

### Prerequisites

- **Python 3.8 or newer**
- **Redis server** (for managing rate limits)

### Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/username/pdf-qa-service.git
    cd pdf-qa-service
    ```

2. **Install dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

3. **Start the Redis server**:
    ```bash
    redis-server
    ```

4. **Run the FastAPI application**:
    ```bash
    uvicorn app.main:app --reload
    ```

### Accessing API Documentation

Once the server is running, you can view interactive API documentation at `http://127.0.0.1:8000/docs`.

### API Endpoints

1. **POST /upload-pdf/**: Uploads a PDF document.
    - **Request**: PDF file
    - **Response**: JSON with file metadata.

2. **WebSocket /ws/question/**: Connects to the WebSocket for real-time Q&A.
    - **Message Format**: `"{document_id}|{question}"`
    - **Response**: Returns answers based on PDF content, with rate limiting.

## Architectural Overview

The backend utilizes FastAPI to handle both HTTP and WebSocket requests. PDF text is extracted using PyMuPDF and stored in an SQLite database. LangChain and LlamaIndex power the NLP for question-answering, enabling context-aware responses.

## Testing Script

A testing script covers the PDF upload, WebSocket Q&A, and rate limiting features.

### **tests/test_main.py**

```python
from fastapi.testclient import TestClient
from app.main import app
import pytest

client = TestClient(app)

def test_upload_pdf():
    with open("sample.pdf", "rb") as pdf_file:
        response = client.post("/upload-pdf/", files={"file": pdf_file})
    assert response.status_code == 200
    assert "filename" in response.json()

def test_websocket_question():
    with client.websocket_connect("/ws/question/") as websocket:
        websocket.send_text("1|What is the travel date?")
        data = websocket.receive_text()
        assert "date" in data  # Assuming the PDF contains a date field

def test_rate_limiting():
    with client.websocket_connect("/ws/question/") as websocket:
        for _ in range(12):  # Exceed the rate limit of 10 messages/min
            websocket.send_text("1|What is the PNR number?")
        response = websocket.receive_text()
        assert "Rate limit exceeded" in response
