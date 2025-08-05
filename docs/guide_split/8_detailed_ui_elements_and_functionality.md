# 8. Detailed UI Elements and Functionality

Lexi's user interface is designed to be intuitive and functional, guiding the user through model interaction, PDF processing, and conversational AI. This section details the key UI components, their purpose, and relevant code snippets.

### 8.1. Initial Loading and Download Screen

Upon launching Lexi, the application first checks for and downloads necessary AI models. This process is represented by a dynamic loading screen, ensuring the user is aware of the background operations.

**Purpose:** To provide visual feedback during the initial setup, model download, and loading phases, preventing the application from appearing unresponsive. It also informs the user about the progress of large file downloads.

**Key Components & Functionality:**

*   **Progress Bar (`ft.ProgressBar`)**: Displays the overall download and loading progress. The `HfProgress` class customizes this to show Hugging Face model download progress.
    ```python
    # Snippet from HfProgress class in Drift_mini.py
    class HfProgress(tqdm):
        # ... (init and other methods)
        def display(self, msg=None, pos=None):
            # Calculate percentage
            if self.total and self.n is not None and self.total > 0:
                percentage = 100.0 * self.n / self.total
            else:
                percentage = 0.0
                
            # Construct status message
            if self.n is not None and self.total is not None:
                status_msg = f"Downloading: {self.n / 1024 / 1024:.1f}MB / {self.total / 1024 / 1024:.1f}MB ({percentage:.1f}%)"
            else:
                status_msg = "Downloading: calculating size..."
                
            # Update UI
            if hasattr(self, 'screen') and self.screen and not self.screen.is_cancelled and self.screen.active:
                try:
                    self.screen.update_progress(
                        40 + (percentage * 0.6),  # Scale to 40-100% range (embedding is first 40%)
                        status_msg
                    )
                except Exception as e:
                    print(f"Error updating progress UI: {e}")
            # ...
    ```

*   **Status Text (`ft.Text`)**: Provides textual updates on the current operation (e.g., "Initializing...", "Downloading models...", "Loading LLM...").
    ```python
        # Snippet from main function in Drift_mini.py
        status_text = ft.Text("Initializing...", size=16, color="white")
        # ...
        page.add(ft.Container(
            content=ft.Column([
                ft.Image(
                    src=f"data:image/png;base64,{icon_base64}",
                    width=1200,
                    height=1200,
                    fit=ft.ImageFit.COVER
                ),
                ft.Container(height=16),
                status_text,
                ft.Container(height=16),
                progress_bar
            ]),
            alignment=ft.alignment.center,
            expand=True
        ))
    ```

*   **Error Messages**: If model download or initialization fails, a clear error message is displayed to the user.
        ```python
        # Snippet from main function in Drift_mini.py
        if not success:
            page.add(ft.Container(
                content=ft.Text("❌ Model download failed. Please check your connection.", color="white"),
                alignment=ft.alignment.center,
                expand=True
            ))
            page.update()
            return
        # ...
        if not success:
            page.add(ft.Container(
                content=ft.Text("❌ Failed to initialize. Check models or logs.", color="white"),
                alignment=ft.alignment.center,
                expand=True
            ))
            page.update()
            return
        ```

### 8.2. Main Chat Interface

Once initialized, the user is presented with the main chat interface, which facilitates interaction with the Lexi AI.

**Key Components & Functionality:**

*   **App Bar (`ft.AppBar` or `ft.Container` acting as one)**: Located at the top, it contains the application title, status indicators, and action buttons.
    *   **Lexi Icon and Title**: Displays the Lexi logo (Base64 encoded) and the application name.
    *   **Model Status Indicator (`ft.Text`)**: Shows which model is currently loaded (e.g., "Optimal Model Selected").
    *   **Model Selection Button (`ft.IconButton`)**: Triggers a dialog for the user to select a different AI model.
        ```python
        # Snippet from app_bar creation in Drift_mini.py
        app_bar = ft.Container(
            content=ft.Row([
                ft.IconButton(
                    icon=ft.icons.MENU,
                    icon_color="#FFFFFF",
                    on_click=lambda e: page.go("/settings"), # Example navigation
                ),
                ft.Container(width=12),
                ft.Column([
                    ft.Text(
                        "Lexi Drift",
                        color="#FFFFFF",
                        size=20,
                        weight=ft.FontWeight.BOLD,
                    ),
                    ft.Text(
                        "Optimal Model Selected", # This text changes based on model status
                        color="#BDBDBD",
                        size=12,
                    ),
                ]),
                ft.Container(expand=True), 
                ft.TextButton(
                    content=ft.Row([
                        ft.Icon(ft.icons.SETTINGS, color="#FFFFFF"),
                        ft.Text("Models", color="#FFFFFF"),
                    ]),
                    on_click=lambda e: open_model_dialog(),
                ),
                ft.IconButton(
                    icon=ft.icons.INFO_OUTLINE,
                    icon_color="#FFFFFF",
                    on_click=lambda e: open_info_dialog(),
                ),
                ft.IconButton(
                    icon=ft.icons.FILE_UPLOAD,
                    icon_color="#FFFFFF",
                    on_click=lambda e: file_picker.pick_files(allow_multiple=True),
                ),
            ]),
            padding=ft.padding.only(left=10, right=10, top=5, bottom=5),
            bgcolor="#202020",
            width=page.width,
        )
        ```
    *   **Info Button (`ft.IconButton`)**: Displays information about the application or current model.
    *   **File Upload Button (`ft.IconButton`)**: Initiates the file picker to allow users to upload documents (e.g., PDFs) for processing.
        ```python
        # Snippet from file picker setup in Drift_mini.py
        file_picker = ft.FilePicker(
            on_result=lambda e: on_file_picked(e, page, chat_column, input_field, send_button, typing_indicator, info_card)
        )
        page.overlay.append(file_picker)
        ```

*   **Chat History Area (`ft.Column`)**: This is the main display area where conversational turns (user queries and Lexi's responses) are rendered.
    *   **User Messages**: Displayed with a distinct background and alignment (e.g., right-aligned).
    *   **Lexi Messages**: Displayed with a distinct background and alignment (e.g., left-aligned), often accompanied by the Lexi avatar.
    *   **Code Highlighting**: Lexi can generate code, which is then highlighted using Pygments for readability.
        ```python
        # Snippet from message rendering in Drift_mini.py
        # ... inside a function that adds messages to chat_column
        if is_user:
            bubble = ft.Container(
                content=ft.Text(message, color="#FFFFFF"),
                bgcolor="#4A4A4A",
                border_radius=10,
                padding=10,
            )
            msg_widget = ft.Row([ft.Container(expand=True), bubble], alignment=ft.MainAxisAlignment.END)
        else:
            # ... Lexi avatar logic ...
            if message.startswith("```") and message.endswith("```"):
                # Code block, apply highlighting
                lang_match = re.match(r"```(\w+)?\n", message)
                lang = lang_match.group(1) if lang_match else "text"
                code_content = message[lang_match.end():-3].strip()
                highlighted_spans = highlight_code_multilang(code_content, lang)
                bubble_content = ft.Column([
                    ft.Text("Lexi:", color="#FFFFFF", weight=ft.FontWeight.BOLD),
                    ft.Container(height=5),
                    ft.Column(highlighted_spans, spacing=0)
                ])
            else:
                bubble_content = ft.Text(message, color="#FFFFFF")

            bubble = ft.Container(
                content=bubble_content,
                bgcolor="#333333",
                border_radius=10,
                padding=10,
            )
            msg_widget = ft.Row([avatar, bubble], alignment=ft.MainAxisAlignment.START)
        chat_column.controls.append(msg_widget)
        page.update()
        ```

*   **Typing Indicator (`ft.Container` with `ft.Text`)**: Appears when Lexi is generating a response, providing visual cues that the AI is processing.
    ```python
    # Snippet from typing indicator in Drift_mini.py
    typing_indicator = ft.Container(
        content=ft.Row([
            ft.ProgressRing(width=16, height=16, stroke_width=2),
            ft.Text("Lexi is typing...", color="#BDBDBD"),
        ]),
        visible=False, # Initially hidden
    )
    ```

*   **PDF Processing Indicator**: When a PDF is uploaded and processed, a message indicates the status (e.g., "Processing PDF...").

### 8.3. Input Area

At the bottom of the main chat interface, the input area allows users to type their queries and send them to Lexi.

**Key Components & Functionality:**

*   **Text Input Field (`ft.TextField`)**: Where the user types their message. It supports multiline input and has a hint text.
    ```python
    # Snippet from input_field creation in Drift_mini.py
    input_field = ft.TextField(
        hint_text="Type your message...",
        hint_style=ft.TextStyle(color="#BDBDBD"),
        multiline=True,
        min_lines=1,
        max_lines=5,
        expand=True,
        border_radius=15,
        filled=True,
        bgcolor="#333333",
        border_color="#4A4A4A",
        text_style=ft.TextStyle(color="#FFFFFF"),
        on_submit=lambda e: send_message(e, page, chat_column, input_field, send_button, typing_indicator, info_card),
    )
    ```

*   **Send Button (`ft.IconButton`)**: Sends the message typed in the input field to Lexi. It is typically enabled when there is text in the input field.
    ```python
    # Snippet from send_button creation in Drift_mini.py
    send_button = ft.IconButton(
        icon=ft.icons.SEND,
        icon_color="#FFFFFF",
        on_click=lambda e: send_message(e, page, chat_column, input_field, send_button, typing_indicator, info_card),
        disabled=True, # Initially disabled
    )
    ```

### 8.4. Model Selection Dialog

This dialog allows users to switch between different AI models available in Lexi. It's typically triggered by a button in the app bar.

**Key Components & Functionality:**

*   **Dialog (`ft.AlertDialog`)**: A modal overlay that presents model options.
*   **Model List (`ft.Column` of `ft.Row`s)**: Each row represents a model, showing its name, description, and a button to select it.
    ```python
    # Snippet from open_model_dialog function in Drift_mini.py
    def open_model_dialog():
        model_options = [
            ("Lexi Lite", "Smaller, faster model for quick responses.", "phi-1_5-gguf-4bit"),
            ("Lexi Core", "Balanced model for general tasks.", "phi-2-gguf-4bit"),
            ("Lexi Edge", "Advanced model for complex queries.", "phi-3-mini-4k-instruct"),
            ("Lexi Ultra", "Largest model for maximum capability.", "deepseek-llm-7b-chat-gguf-q4")
        ]

        model_controls = []
        for name, description, model_folder in model_options:
            model_controls.append(
                ft.Row([
                    ft.Column([
                        ft.Text(name, weight=ft.FontWeight.W_500, size=14),
                        ft.Text(description, size=12, color="#BDBDBD"),
                    ]),
                    ft.Container(expand=True),
                    ft.TextButton(
                        "Select",
                        on_click=partial(select_model, model_folder, name),
                    ),
                ])
            )

        dialog = ft.AlertDialog(
            modal=True,
            title=ft.Text("Available Models"),
            content=ft.Column(
                model_controls,
                scroll="always",
                height=300,
            ),
            actions=[
                ft.TextButton("Close", on_click=lambda _: close_dialog()),
            ],
            actions_alignment=ft.MainAxisAlignment.END,
        )
        page.dialog = dialog
        dialog.open = True
        page.update()
    ```
*   **Disclaimer**: Warns users about potential crashes when switching models.
*   **Close Button**: Closes the dialog.

### 8.5. Information Dialog

This dialog provides general information about Lexi, potentially including version details, credits, or links to documentation.

**Key Components & Functionality:**

*   **Dialog (`ft.AlertDialog`)**: A modal overlay for displaying information.
*   **Content (`ft.Text`)**: Displays the informational text.
*   **Close Button**: Closes the dialog.

### 8.6. PDF Upload and Processing

Lexi allows users to upload PDF documents, which are then processed to extract text and create embeddings for conversational context.

**Key Components & Functionality:**

*   **File Picker (`ft.FilePicker`)**: A non-visual control that interacts with the operating system to allow users to select files. Its `on_result` event handler processes the selected files.
    ```python
    # Snippet from on_file_picked function in Drift_mini.py
    async def on_file_picked(e: ft.FilePickerResultEvent, page, chat_column, input_field, send_button, typing_indicator, info_card):
        if e.files:
            for file in e.files:
                # Display message that PDF is being processed
                add_message(page, chat_column, f"Processing PDF: {file.name}...", is_user=True)
                page.update()

                # Simulate processing time or actual processing
                await asyncio.sleep(1) 

                # Actual PDF processing logic (simplified for snippet)
                pdf_path = file.path
                if pdf_path:
                    # ... (PDF loading, splitting, embedding logic)
                    add_message(page, chat_column, f"PDF '{file.name}' processed and ready for questions!", is_user=False)
                else:
                    add_message(page, chat_column, f"Failed to load PDF '{file.name}'.", is_user=False)
                page.update()
    ```
*   **Status Messages in Chat**: Messages are added to the chat history to inform the user about the PDF processing status (e.g., "Processing PDF: ...", "PDF processed...").

When making changes or adding new features, always consider the existing structure and design principles.


