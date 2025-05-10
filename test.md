**4.1. Proposed Technology Stack**
    **4.1.1. Backend:**
    *   **Language:** Python (Version 3.9+)
    *   **Framework:** Django (Version 4.x or latest stable) - Chosen for its robustness, scalability, "batteries-included" nature (ORM, admin, security), and strong community support, making it ideal for a complex, data-driven application.
    *   **Asynchronous Communication:** Server-Sent Events (SSE) for real-time updates from the AI agent.
    *   **Sentiment Analysis Library:** VADER (`vaderSentiment`).
    *   **Fuzzy Search Library:** `thefuzz`.
    **4.1.2. Frontend:**
    *   **Languages:** HTML5, CSS3, JavaScript (ES6+)
    *   **Key Libraries (Client-Side):**
        *   `@google/generative-ai` (JavaScript SDK for direct Gemini API interactions from the client for certain features).
        *   `marked.js` (for parsing Markdown responses from AI).
        *   `highlight.js` (for syntax highlighting code blocks in AI responses).
        *   *No heavy frontend framework (like React/Angular/Vue) is initially proposed to maintain simplicity, relying on vanilla JavaScript and potentially jQuery for DOM manipulation and AJAX.*
    **4.1.3. Database:**
    *   **Development:** SQLite (for ease of setup).
    *   **Production (Proposed):** PostgreSQL or MySQL (for better performance, concurrency, and scalability). Django's ORM allows for easy switching.
    **4.1.4. AI/ML Services:**
    *   **Large Language Models:** Google Generative AI – specifically, models from the Gemini family (e.g., `gemini-1.5-flash-latest` for speed/cost balance, `gemini-1.5-pro-latest` for more complex reasoning if needed, and a vision-capable model for image analysis).
    **4.1.5. Development and Collaboration Tools:**
    *   **Version Control:** Git (with a remote repository on GitHub/GitLab).
    *   **IDE:** PyCharm Professional or VS Code with appropriate Python/Django extensions.
    *   **Deployment Platform (Initial):** PythonAnywhere (for ease of deployment of Python/Django applications).

**4.2. System Architecture (Conceptual)**
A multi-tier architecture is envisioned:
*   **Presentation Tier (Client):** Web browser rendering HTML, CSS, JS. Handles user input and displays data. Initiates some AI interactions directly (with secure backend proxy for API keys).
*   **Application Tier (Server):** Django application handling business logic, request processing, form validation, database interaction via ORM, template rendering, and orchestrating backend AI interactions (including the AI agent logic and SSE streaming).
*   **Data Tier:** Relational database (SQLite/PostgreSQL/MySQL) storing all application data.
*   **External Services Tier:** Google Generative AI services accessed via APIs.

**4.3. Development Methodology**
An iterative and incremental development approach, borrowing principles from Agile methodologies, will be adopted:
1.  **Core Foundation:** Build the basic Django project structure, user authentication, profile models, and book/category models.
2.  **Basic Features:** Implement core CRUD functionalities for books, basic recommendations, and the review system.
3.  **Iterative AI Integration:** Develop AI features one by one (e.g., Category Chat -> Image Finder -> AI Agent), with each feature undergoing cycles of prompt engineering, backend integration, frontend implementation, and refinement.
4.  **Regular Internal Review:** Frequent internal reviews (between team members) to assess progress, identify roadblocks, and ensure components integrate correctly.
5.  **Focus on User Experience:** UI/UX considerations will be integrated throughout each development iteration.
