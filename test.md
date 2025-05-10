*   **Integrated Development Environment (IDE):** PyCharm Professional was likely used for its robust Python and Django support, debugging capabilities, and integration with version control. Alternatives like VS Code with Python extensions are also common.
*   **Programming Languages:**
    *   Python (Version 3.8+ recommended for modern Django and AI libraries) for backend logic.
    *   JavaScript (ES6+) for frontend interactivity and client-side API calls.
    *   HTML5 and CSS3 for structuring and styling web pages.
*   **Frameworks & Libraries (Backend):**
    *   Django (e.g., Version 3.2 or 4.x) as the core web framework.
    *   Django REST framework (DRF) - *Potentially used for some APIs, though the provided code primarily uses `JsonResponse` directly. If DRF was used, it should be mentioned here.*
    *   `google-generativeai` (Python SDK for Gemini API access from the backend).
    *   `vaderSentiment` (Python library for sentiment analysis).
    *   `thefuzz` (Python library for fuzzy string matching).
*   **Libraries (Frontend):**
    *   `@google/generative-ai` (JavaScript SDK for client-side Gemini API access).
    *   `marked.js` (For parsing Markdown to HTML).
    *   `highlight.js` (For syntax highlighting of code blocks).
    *   *jQuery (Potentially, though not explicitly shown in all scripts. Often used with Django for simpler AJAX).*
*   **Database:** SQLite for local development, as configured in Django's `settings.py`.
*   **Version Control:** Git, with a repository hosted on a platform like GitHub, GitLab, or Bitbucket, for source code management and collaboration.
*   **Web Server (Development):** Django's built-in development server (`manage.py runserver`).
*   **Package Management:**
    *   `pip` (Python package installer) with a `requirements.txt` file to manage Python dependencies.
    *   `npm` or `yarn` (If frontend dependencies were managed formally, though not explicitly indicated by the script includes using CDNs).
*   **Operating System:** Development could occur on Windows, macOS, or Linux.
