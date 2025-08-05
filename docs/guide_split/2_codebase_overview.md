# 2. Codebase Overview

The Lexi codebase (`Drift_mini.py`) is a monolithic application that integrates several powerful libraries and components:

*   **Flet**: Used for building the cross-platform user interface. Flet allows developers to create desktop, web, and mobile applications using Python.
*   **Hugging Face Transformers**: Utilized for loading and managing large language models (LLMs) and sentence transformers for various AI tasks, including text generation and embeddings.
*   **Langchain**: Provides a framework for developing applications powered by language models, facilitating conversational AI and document processing.
*   **Llama.cpp**: A C/C++ port of Facebook's LLaMA model, used here for efficient local inference of GGUF models.
*   **PyGments**: Used for syntax highlighting of code snippets within the UI.
*   **PyPDFLoader**: For loading and processing PDF documents.

The application structure includes components for model selection, PDF processing, chat interaction, and UI rendering. The core logic resides within the `Drift_mini.py` file, managing both the backend AI operations and the frontend UI presentation.
