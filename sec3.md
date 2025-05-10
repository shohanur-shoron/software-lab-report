**3. System Design**

This chapter elaborates on the comprehensive design of the BookVerse AI system. It covers the overall system architecture, the detailed database schema, the design of individual functional modules, the Application Programming Interfaces (APIs) facilitating communication, considerations for User Interface (UI) and User Experience (UX), and key security measures implemented.

**3.1. System Architecture**

    3.1.1. High-Level Architectural Overview
    BookVerse AI is designed as a monolithic web application with distinct components for frontend, backend, and AI service interaction. It primarily follows a **Model-View-Template (MVT)** architecture, inherent to the Django framework, which is a variation of the Model-View-Controller (MVC) pattern.
    *   **Client-Side (Browser):** The user interacts with the system through a web browser. The frontend is built using HTML, CSS, and JavaScript. JavaScript is extensively used for dynamic UI updates, asynchronous communication with the backend (AJAX), and handling real-time interactions with AI services (e.g., streaming responses from Gemini).
    *   **Web Server/Application Server (Django):** Django handles incoming HTTP requests, routes them to appropriate view functions, processes business logic, interacts with the database via its ORM, and renders HTML templates to be sent back to the client. For AI-driven agentic features, it also orchestrates communication with external LLM APIs and manages Server-Sent Events (SSE) for streaming responses.
    *   **Database:** A relational database (SQLite for development, with potential for PostgreSQL/MySQL in production) stores all persistent data, including user profiles, book information, categories, reviews, reading statuses, and AI interaction history where relevant.
    *   **External AI Services (Google Generative AI - Gemini):** The system makes API calls to Google's Gemini models for:
        *   Natural language understanding and generation for book category recommendations.
        *   Image analysis for identifying books from cover images.
        *   Instruction parsing and response generation for the agentic AI managing user lists.
    The architecture emphasizes a separation of concerns, with Django managing the core application logic and data, while the frontend handles presentation and user interaction, and AI services provide specialized intelligence.

    3.1.2. Component Diagram and Interactions
    *(This would ideally be a visual diagram. Below is a textual description that should guide its creation.)*

    **Key Components:**
    1.  **User Interface (Frontend):**
        *   HTML/CSS/JS files rendering in the browser.
        *   `scripts.js`, `scripts-image.js`, `scripts-specific-book.js`, `search-suggestions.js`: Handle UI logic, event listeners, AJAX calls, and client-side Gemini interaction setup.
        *   `gemini2.js`, `gemini-bookverse2.js`, `gemini-image.js`: Client-side wrappers for direct communication with Gemini API for specific chat/image tasks (note: API key handling here needs careful consideration for security in production).
    2.  **Django Web Application (Backend):**
        *   **URL Dispatcher (`urls.py`):** Routes incoming HTTP requests.
        *   **Views (`views.py` in `users`, `book`, `mainpages`, `agent_chat` apps):** Contain the business logic.
            *   Handle user authentication and profile operations.
            *   Manage book, category, and author data.
            *   Process forms.
            *   Implement search and recommendation logic.
            *   Orchestrate AI agent interactions (parsing Gemini instructions, executing actions, summarizing results).
            *   Manage SSE streams.
        *   **Models (`models.py` in `users`, `book` apps):** Define the database schema and provide an ORM interface.
        *   **Templates (`*.html` files):** HTML files with Django Template Language for dynamic content generation.
        *   **Forms (`forms.py` in `book` app):** Define and validate forms for data input.
        *   **Utilities (`utills.py` in `users` app):** Helper functions.
        *   **Internal APIs (`api.py` in `users` app):** Provide JSON responses for AJAX calls (e.g., username availability).
    3.  **Database (e.g., SQLite):**
        *   Stores all persistent application data.
    4.  **Google Generative AI Service (External):**
        *   Gemini Models (e.g., `gemini-1.5-flash-latest`, `gemini-1.5-pro-latest`).
        *   Accessed via API calls from both frontend JavaScript (for direct chat features) and Django backend (for agentic AI processing).

    **Interaction Flow Example (Agentic AI Chat):**
    1.  User types a command in the `ai_agent.html` chat interface.
    2.  JavaScript (`agent_chat/initiate_nlp_processing` AJAX call) sends the query to the Django backend.
    3.  Django's `initiate_nlp_processing` view stores the query in the session and returns a `stream_id`.
    4.  JavaScript then opens an SSE connection to `stream_nlp_results/<stream_id>`.
    5.  Django's `_stream_nlp_processing` function (generator):
        a.  Retrieves the query.
        b.  Sends an API call to Gemini (via `get_gemini_instructions_internal`) to parse the command into JSON.
        c.  Yields status updates via SSE.
        d.  Calls `execute_single_action` to perform database operations based on parsed JSON.
        e.  Yields status/warning updates via SSE.
        f.  Calls `summarize_results_with_gemini` to get a natural language summary from Gemini.
        g.  Yields the final summary and "finished" event via SSE.
    6.  JavaScript on the client-side receives SSE events and updates the chat UI dynamically.

    3.1.3. Technology Stack Selection and Rationale
        *   **3.1.3.1. Backend: Python & Django**
            *   **Python:** Chosen for its readability, extensive libraries (especially for AI/ML), and large developer community.
            *   **Django:** Selected for its "batteries-included" nature, providing an ORM, admin interface, robust security features (CSRF, XSS), and a structured MVT pattern suitable for rapid development of complex, data-driven applications like BookVerse AI. Its scalability makes it a good choice for future growth.
        *   **3.1.3.2. Database: SQLite (Development) / PostgreSQL or MySQL (Potential Production)**
            *   **SQLite:** Used during development for its simplicity and ease of setup (file-based, no separate server needed).
            *   **PostgreSQL/MySQL:** Recommended for production due to better concurrency, scalability, data integrity features, and robustness compared to SQLite. Django's ORM allows for relatively easy migration between these databases.
        *   **3.1.3.3. Frontend: HTML, CSS, JavaScript**
            *   **HTML:** Standard markup language for structuring web pages.
            *   **CSS:** Used for styling and presentation, ensuring a visually appealing and responsive user interface.
            *   **JavaScript:** Essential for client-side interactivity, dynamic content updates, handling asynchronous operations (AJAX, SSE, Gemini API calls from client), and managing the UI of chat interfaces. No specific frontend framework (like React, Vue, Angular) was chosen for this iteration to keep complexity manageable, relying on vanilla JavaScript and jQuery (if used, not explicitly stated but common with Django).
        *   **3.1.3.4. AI/ML: Google Generative AI (Gemini) & VADER Sentiment**
            *   **Google Generative AI (Gemini Models):** Chosen for their advanced natural language understanding, generation, and multimodal capabilities. The specific models (`gemini-1.5-flash-latest`, `gemini-1.5-pro-latest`, `gemini-2.0-flash`) offer a balance of performance and cost, suitable for tasks like conversational category recommendation, image-to-text for book covers, and instruction parsing for the AI agent. The ease of use of its API was also a factor.
            *   **VADER Sentiment (`vaderSentiment` Python library):** Selected for its effectiveness in analyzing sentiment in text, particularly user-generated content like reviews. It's lexicon-based, fast, and doesn't require training data for general sentiment analysis, making it easy to integrate for analyzing book comments.
        *   **3.1.3.5. Other Libraries/Tools:**
            *   **`thefuzz` (Python library):** Used for implementing fuzzy string matching in the search functionality (`search_books_sqlite_fuzzy`), allowing for more tolerant and user-friendly search results that can handle typos or partial matches.
            *   **`highlight.js` (JavaScript library):** Used on the frontend to provide syntax highlighting for code blocks within AI-generated responses, improving readability for technical content.
            *   **`marked.js` (JavaScript library):** Used on the frontend to parse Markdown formatted text (often returned by LLMs) into HTML for rich display in chat interfaces.

**3.2. Database Design**

    3.2.1. Entity-Relationship Diagram (ERD)
    *(This would be a visual diagram. The description below outlines the entities and their key relationships. An actual ERD would show cardinalities like one-to-one, one-to-many, many-to-many explicitly.)*

    **Entities:**
    *   User (Django's `auth.User`)
    *   Profile (`users.Profile`)
    *   Category (`book.Category`)
    *   Author (`book.Author`)
    *   Book (`book.Book`)
    *   Favorite (`book.Favorite`)
    *   ReadingStatus (`book.ReadingStatus`)
    *   Event (`book.Event`)
    *   MyEvent (`book.MyEvent`)
    *   Comment (`book.Comment`)
    *   RecentlyViewedBook (`book.RecentlyViewedBook`)

    **Key Relationships:**
    *   `User` has a one-to-one relationship with `Profile`.
    *   `Profile` has a many-to-many relationship with `Category` (for user interests).
    *   `User` (as `added_by`) has a one-to-many relationship with `Category`.
    *   `User` (as `addedBy`) has a one-to-many relationship with `Author`.
    *   `Author` has a many-to-many relationship with `User` (for followers).
    *   `Book` has a many-to-one relationship with `Category`.
    *   `Book` has a many-to-one relationship with `Author` (authors).
    *   `Book` has a many-to-one relationship with `User` (as publisher).
    *   `Book` has a many-to-many relationship with `User` (for likes).
    *   `Favorite` acts as a junction table for a many-to-many relationship between `User` and `Book` (explicit favorites list).
    *   `ReadingStatus` acts as a junction table linking `User` and `Book` with additional status information.
    *   `Comment` has a many-to-one relationship with `Book` and `User`.
    *   `MyEvent` acts as a junction table for a many-to-many relationship between `User` and `Event`.
    *   `RecentlyViewedBook` acts as a junction table linking `User` and `Book` with a timestamp.

    3.2.2. Detailed Schema for Each Model
    *(Based on `models.py` files provided)*

        *   **`django.contrib.auth.models.User` (Built-in Django Model):**
            *   `username`: CharField (unique)
            *   `password`: CharField (hashed)
            *   `email`: EmailField (optional, can be unique)
            *   `first_name`: CharField
            *   `last_name`: CharField
            *   `is_staff`, `is_active`, `is_superuser`: BooleanFields
            *   `date_joined`, `last_login`: DateTimeFields

        *   **`users.models.Profile`:**
            *   `user`: OneToOneField to `User` (CASCADE delete, null, blank)
            *   `image`: ImageField (upload_to='proPic/original', null, blank)
            *   `phone`: CharField (max_length=20)
            *   `gender`: CharField (max_length=20, null, blank)
            *   `is_user`: BooleanField (default=False)
            *   `is_reviewer`: BooleanField (default=False)
            *   `interests`: ManyToManyField to `book.Category` (related_name='interests', blank, null)
            *   `total_reviews`: IntegerField (default=0)
            *   `total_views`: IntegerField (default=0)
            *   `__str__`: Returns `self.user.first_name`

        *   **`book.models.Category`:**
            *   `name`: CharField (max_length=100)
            *   `description`: TextField (null, blank)
            *   `added_by`: ForeignKey to `User` (CASCADE delete, null, blank)
            *   `is_active`: BooleanField (default=False)
            *   `__str__`: Returns f"{self.name} added by {self.added_by}"

        *   **`book.models.Author`:**
            *   `name`: CharField (max_length=100)
            *   `image`: ImageField (upload_to='author_images/', blank, null)
            *   `followers`: ManyToManyField to `User` (related_name='followers', blank)
            *   `addedBy`: ForeignKey to `User` (related_name='addedBy', CASCADE delete, blank, null)
            *   `__str__`: Returns f"{self.name} added by {self.addedBy}"

        *   **`book.models.Book`:**
            *   `name`: CharField (max_length=100)
            *   `image`: ImageField (upload_to='book_images/', blank, null)
            *   `description`: TextField
            *   `category`: ForeignKey to `Category` (CASCADE delete)
            *   `price`: PositiveIntegerField (default=0)
            *   `authors`: ForeignKey to `Author` (CASCADE delete, null, blank)
            *   `publisher`: ForeignKey to `User` (CASCADE delete, null, blank)
            *   `total_views`: PositiveIntegerField (default=0)
            *   `rating`: PositiveIntegerField (default=0) *(Note: This seems to be a distinct field from comment ratings; consider how it's populated or if it's an aggregate)*
            *   `suggestions`: TextField (blank, null)
            *   `link`: URLField (blank, null)
            *   `link_clicked`: PositiveIntegerField (default=0)
            *   `likes_counter`: PositiveIntegerField (default=0)
            *   `likes`: ManyToManyField to `User` (related_name='likes', blank)
            *   `published_time`: DateField (auto_now_add=True)
            *   `pages`: CharField (max_length=80, null, blank) *(Consider IntegerField if always numeric)*
            *   `language`: CharField (max_length=100, blank, null)
            *   `chapters`: CharField (max_length=50, null, blank) *(Consider IntegerField)*
            *   `favorites_chapters`: CharField (max_length=50, null, blank)
            *   `favorites_quotes`: TextField (blank, null)
            *   `series`: CharField (max_length=100, blank, null)
            *   `best_character`: CharField (max_length=100, blank, null)
            *   `awards`: TextField (blank, null)
            *   `format`: CharField (max_length=50, choices=[('Hardcover', 'Hardcover'), ...], blank, null)
            *   `get_absolute_url`: Method returning reverse('library:book_detail', ...)
            *   `__str__`: Returns f'{self.name} by {self.authors.name} book'

        *   **`book.models.Favorite`:** (Explicit user favorites list)
            *   `book`: ForeignKey to `Book` (CASCADE delete)
            *   `user`: ForeignKey to `User` (CASCADE delete)
            *   `timestamp`: DateTimeField (auto_now_add=True)
            *   `__str__`: Returns `self.book.name`

        *   **`book.models.ReadingStatus`:**
            *   `book`: ForeignKey to `Book` (CASCADE delete)
            *   `user`: ForeignKey to `User` (CASCADE delete)
            *   `status`: CharField (max_length=20, choices=[('to_read', 'To Read'), ...])
            *   `current_page`: PositiveIntegerField (default=0)
            *   `last_updated`: DateTimeField (auto_now=True)

        *   **`book.models.Event`:**
            *   `name`: CharField (max_length=100)
            *   `description`: TextField (null, blank)
            *   `timestamp`: DateTimeField *(Note: In the code, this is just `DateTimeField()`, might need `default=timezone.now` or `auto_now_add=True` depending on intent)*
            *   `image`: ImageField (upload_to='event_images/', blank, null)
            *   `ongoing`: BooleanField (default=False)
            *   `is_valid`: BooleanField (default=False)
            *   `__str__`: Returns f'{self.name} - {self.is_valid}'

        *   **`book.models.MyEvent`:** (User's participation in events)
            *   `user`: ForeignKey to `User` (CASCADE delete)
            *   `event`: ForeignKey to `Event` (CASCADE delete)
            *   `timestamp`: DateTimeField (auto_now_add=True)
            *   `__str__`: Returns f"{self.user} {self.event.name}"

        *   **`book.models.Comment`:**
            *   `book`: ForeignKey to `Book` (CASCADE delete, related_name='comments')
            *   `user`: ForeignKey to `User` (CASCADE delete, related_name='comments')
            *   `text`: TextField
            *   `textSentimentScore`: FloatField (null, blank)
            *   `sentimentCategory`: CharField (max_length=50, null, blank)
            *   `rating`: IntegerField (validators=[MinValueValidator(1), MaxValueValidator(5)])
            *   `created_at`: DateTimeField (auto_now_add=True)
            *   `updated_at`: DateTimeField (auto_now=True)
            *   `Meta`: `ordering = ['-created_at']`
            *   `__str__`: Returns f'Comment by {self.user.username} on {self.book.name}'

        *   **`book.models.RecentlyViewedBook`:**
            *   `user`: ForeignKey to `User` (CASCADE delete, related_name='recently_viewed')
            *   `book`: ForeignKey to `Book` (CASCADE delete)
            *   `timestamp`: DateTimeField (auto_now=True)
            *   `Meta`: `ordering = ['-timestamp']`, `unique_together = ('user', 'book')`
            *   `__str__`: Returns f"{self.user.username} viewed {self.book.name} at {self.timestamp}"

    3.2.3. Data Integrity and Relationships
    *   **Foreign Keys:** Used extensively to maintain referential integrity between tables (e.g., a `Book` must belong to a valid `Category`). `on_delete=models.CASCADE` is commonly used, meaning if a referenced object (e.g., a `User`) is deleted, related objects (e.g., their `Profile`, `Comments`) are also deleted.
    *   **OneToOneField:** Ensures a strict one-to-one mapping (e.g., `User` to `Profile`).
    *   **ManyToManyField:** Models many-to-many relationships (e.g., `Profile.interests` to `Category`, `Book.likes` to `User`). Django automatically creates intermediary tables for these unless a `through` model is specified (like `Favorite` conceptually is for user-book favorites).
    *   **`null=True, blank=True`:** Used for optional fields. `null=True` allows the database to store NULL values, while `blank=True` allows forms to accept empty values.
    *   **`unique_together`:** Ensures combinations of fields are unique (e.g., a user cannot have the same book multiple times in `RecentlyViewedBook`).
    *   **Validators:** Used on fields like `Comment.rating` to enforce constraints (e.g., rating between 1 and 5).
    *   **Choices:** Provide a predefined set of options for fields like `Book.format` and `ReadingStatus.status`, enhancing data consistency.

**3.3. Module Design**
    The system is modularized based on Django applications (`users`, `book`, `mainpages`, `agent_chat`) and functionalities.

    *   **3.3.1. User Authentication and Profile Management Module (Primarily `users` app, with views in `users/views.py`)**
        *   **3.3.1.1. Registration and Login Flow:**
            *   Registration (`create_account` view): Collects first name, last name, email, phone, gender, password. Generates a unique username (`generate_unique_username`). Creates a `User` object and an associated `Profile` object. Handles password confirmation and checks for existing phone numbers.
            *   Login (`login_user` view): Allows login via username/email OR phone number. Uses `authenticate` and `login` from `django.contrib.auth`.
        *   **3.3.1.2. Profile Creation and Updates:**
            *   Profile is created automatically upon user registration.
            *   `update_account` view allows users to modify their first name, last name, email, phone, and gender.
            *   `upload_image` and `change_image` views handle profile picture uploads, saving to `Profile.image`.
            *   `add_your_interest` and `change_interest` views, along with `add_interest` and `del_interest` API views, manage the `Profile.interests` ManyToManyField with `Category`.
        *   **3.3.1.3. Username Availability and Management:**
            *   `is_username_available` API (`users/api.py`) checks if a username is taken (used during registration or username change).
            *   `update_username` (post-registration step) and `change_username` views allow users to set/modify their username, with checks for uniqueness.
    *   **3.3.2. Book and Content Management Module (Primarily `book` app, views in `book/views.py`)**
        *   **3.3.2.1. Book CRUD Operations:**
            *   `create_book` view handles book creation. It checks for existing authors/categories or creates new ones if they don't exist, associating them with the current user (`request.user`) as `added_by` or `publisher`. Handles image uploads for book covers.
            *   (Update/Delete operations for books are not explicitly detailed in provided views but would typically be handled by admin interface or dedicated views for authorized users/reviewers).
        *   **3.3.2.2. Category and Author Management:**
            *   Categories and Authors can be created implicitly during book creation if they don't exist.
            *   The `book.forms` (`CategoryForm`, `AuthorForm`) suggest that dedicated views for managing these might exist or are planned, likely for admin/reviewer roles.
        *   **3.3.2.3. Image Upload and Handling:** Uses Django's `ImageField` for profile pictures, author images, book covers, and event images, with specified `upload_to` paths.
    *   **3.3.3. AI-Powered Recommendation Module (Distributed across `mainpages`, `book` apps, and frontend JS for Gemini)**
        *   **3.3.3.1. Interest Discovery Sub-module (LLM-based Chat for Categories):**
            *   **Frontend (`gemini-bookverse2.js`):** Handles user interaction in a chat interface.
            *   **Prompt Engineering:** Uses a detailed system prompt (defined in `gemini-bookverse2.js`) to guide the Gemini model. The prompt instructs the LLM to analyze user's natural language description of interests and recommend 1-4 relevant categories from a fixed list, explaining the rationale. It also handles explicit requests for book examples within a category.
            *   **Interaction Flow:** User types interests -> JS sends this to Gemini with the system prompt -> Gemini responds with category suggestions (or book examples if requested) -> JS displays the response.
            *   **API Design:** Client-side JavaScript directly calls the Gemini API.
        *   **3.3.3.2. Book Recommendation Logic (Backend):**
            *   `mainpages.views.get_interested_books`: Retrieves books whose category matches the logged-in user's `Profile.interests`.
            *   `mainpages.views.get_not_interested_books`: Retrieves books whose category does NOT match user interests (for the "discover" page).
            *   Homepage (`mainHomePage`) displays `get_interested_books` by default.
        *   **3.3.3.3. Image-to-Book Information Sub-module (LLM Vision):**
            *   **Frontend (`gemini-image.js`, `scripts-image.js`):** Allows users to upload an image or capture one from their camera.
            *   **Prompt Engineering:** A fixed prompt ("tell me about this book in details") is sent along with the image to the Gemini vision model.
            *   **Interaction Flow:** User provides image -> JS sends image and prompt to Gemini -> Gemini responds with text about the book -> JS displays the response.
            *   **API Design:** Client-side JavaScript directly calls the Gemini API with image data.
    *   **3.3.4. Agentic AI Management Module (LLM-based Account/List Control - `agent_chat` app)**
        *   **3.3.4.1. System Prompt Design (`agent_chat.views.get_full_prompt`):** A highly structured prompt instructs the Gemini model to parse user's natural language queries into JSON objects (or an array of objects for multiple requests) representing actions on 'favourite_list' or 'reading_status'. Defines specific operations (show_all, count, add, remove, show_summary, show_specific_status, set_status) and required parameters.
        *   **3.3.4.2. Instruction Parsing (`agent_chat.views.get_gemini_instructions_internal`):** Sends the user query combined with the system prompt to Gemini. Expects a JSON response as per the prompt's specifications. Handles potential cleanup of Gemini's raw response.
        *   **3.3.4.3. Action Execution Logic (`agent_chat.views.execute_single_action`):** Takes a parsed JSON instruction.
            *   Identifies the action (`favourite_list`, `reading_status`) and operation.
            *   Finds books using `Book.objects.get(name__iexact=...)` or `filter(name__icontains=...)` for fuzzy matching when book names are parameters.
            *   Performs database operations:
                *   `Favorite` model: `get_or_create`, `delete`, `count`.
                *   `ReadingStatus` model: `update_or_create`, `delete`, `count` by status.
            *   Returns a status dictionary (success, error, warning) with a message and relevant data.
        *   **3.3.4.4. Result Summarization and Streaming (`agent_chat.views.summarize_results_with_gemini`, `_stream_nlp_processing`):**
            *   `_stream_nlp_processing` orchestrates the flow: gets instructions, executes actions, and then calls `summarize_results_with_gemini`.
            *   `summarize_results_with_gemini` takes the list of action results and the original query, sends them to another Gemini model (summary_model) with a prompt to generate a user-friendly, natural language summary of what was done. This summary can include HTML formatting (like image tags for book covers).
            *   Uses Server-Sent Events (SSE) to stream status updates ("Received request", "Processing...", "Issue with action...", "Wrapping up...") and the final summary to the client.
        *   **3.3.4.5. Supported Actions (as defined in system prompt):**
            *   **Favorites:** Show all, count, add by book name, remove by book name.
            *   **Reading Status:** Show summary (counts per status), show specific status list (to_read, reading, completed), set status for a book, remove status for a book.
    *   **3.3.5. Review and Interaction Module (Primarily `book` app views and models)**
        *   **3.3.5.1. Commenting System (`book.views.add_comment_view`, `book.models.Comment`):**
            *   Users can add comments and ratings (1-5) to books.
            *   `add_comment_view` expects JSON data (book_id, comment_text, rating), creates a `Comment` object.
        *   **3.3.5.2. Sentiment Analysis of Comments (`book.views.analyze_comment_sentiment_vader`):**
            *   Called within `add_comment_view`.
            *   Uses the VADER library to calculate a compound sentiment score and categorize it (e.g., "Positive", "Negative", "Neutral"). This data is stored in `Comment.textSentimentScore` and `Comment.sentimentCategory`.
            *   The aggregated sentiment score per book is calculated in `book_detail_view`.
        *   **3.3.5.3. Book Liking/Unliking (`book.views.like_book_view`, `Book.likes` M2M field, `Book.likes_counter`):**
            *   `like_book_view` handles adding/removing (implicitly by `add()`) a user to the `Book.likes` ManyToManyField. It also updates `Book.likes_counter`.
        *   **3.3.5.4. Adding/Removing from Favorites (`book.views.toggle_favourite_view`, `book.models.Favorite`):**
            *   This refers to the explicit "Favorite" model, distinct from "likes."
            *   `toggle_favourite_view` uses `Favorite.objects.get_or_create()` and `delete()` to add or remove a book from the user's `Favorite` list. This is separate from the agentic AI management but acts on the same underlying data.
    *   **3.3.6. Reading Status and List Management Module (Backend: `book/views.py`, `mainpages/views2.py`; Frontend: for AI agent)**
        *   **3.3.6.1. Tracking Statuses (`book.models.ReadingStatus`):** Stores `book`, `user`, `status` ('to_read', 'reading', 'completed'), `current_page`, `last_updated`.
        *   **3.3.6.2. Updating Reading Progress (`mainpages.views2.update_reading_progress_api`):**
            *   Allows users (likely via UI elements not detailed in provided JS, but API exists) to update `current_page` for books in 'reading' status.
            *   Automatically transitions status to 'completed' if `current_page` reaches total pages.
        *   **3.3.6.3. Displaying User-Specific Lists (various views in `mainpages/views.py` and `mainpages/views2.py`):**
            *   `your_favorites` (from `Favorite` model), `recently_seen` (from `RecentlyViewedBook`), `liked_books` (from `Book.likes`), `your_comments` (from `Comment`).
            *   `want_to_read_books` (`status='to_read'`), `currently_reading_books` (`status='reading'`), `finish_reading_books` (`status='completed'`).
            *   User profile (`user_profile` view) aggregates and displays these lists.
    *   **3.3.7. Search and Discovery Module (Primarily `mainpages` app)**
        *   **3.3.7.1. Fuzzy Search Implementation (`mainpages.views2.search_books_sqlite_fuzzy`):**
            *   Takes a query string.
            *   Performs initial `icontains` lookups on book name, author name, description, category name, series.
            *   Further refines results using `fuzz.token_set_ratio` from `thefuzz` library to score candidates against the query.
            *   Returns books scoring above `SIMILARITY_THRESHOLD`.
            *   Used in `mainHomePage` when `request.method == 'POST'`.
        *   **3.3.7.2. Category-based Browsing (`mainpages.views.specific_category`, `category_page`):**
            *   `category_page` lists all categories.
            *   `specific_category` shows all books belonging to a selected category.
        *   **3.3.7.3. Discover Page Logic (`mainpages.views.discoverBooksPage`, `get_not_interested_books`):** Shows books whose categories are *not* in the user's profile interests.
        *   **3.3.7.4. Search Suggestions (`mainpages.views.search_items`, `search-suggestions.js`):**
            *   `search_items` backend view returns a flat JSON list of all book names, category names, and author names.
            *   `search-suggestions.js` frontend script fetches this data, filters it based on user input in the search box, and displays matching suggestions dynamically.

**3.4. API Design (for AJAX and internal communication)**
    The system uses several internal APIs for asynchronous operations and dynamic content updates, primarily returning JSON responses.

    *   **User Management APIs (`users/api.py` and implicit in `users/views.py`):**
        *   `is_username_available(request, username)`: `GET`. Checks if a username exists.
            *   Response: `{'available': True/False}`
        *   `add_interest(request, interest)`: `GET` (Could be `POST` for adding data). Adds a category to user's interests.
            *   Response: `{'success': True}` or `{'error': 'message'}, status=404/500`
        *   `del_interest(request, interest)`: `GET` (Could be `POST`/`DELETE`). Removes a category from user's interests.
            *   Response: `{'success': True}` or `{'error': 'message'}, status=404/500`

    *   **Book Interaction APIs (`book/views.py`):**
        *   `add_comment_view(request)`: `POST`. Expects JSON `{'book_id', 'comment_text', 'rating'}`.
            *   Response: `{'status': 'success/error', 'message': '...'}`
        *   `like_book_view(request)`: `POST`. Expects JSON `{'book_id'}`.
            *   Response: `{'status': 'success/error', 'new_count': count}`
        *   `toggle_favourite_view(request)`: `POST`. Expects JSON `{'book_id'}`.
            *   Response: `{'status': 'success/error', 'message': '...', 'is_favourite': True/False}`
        *   `add_to_reading_list_view(request)`: `POST`. Expects JSON `{'book_id'}` (sets status to 'to_read').
            *   Response: `{'status': 'success/error', 'message': '...', 'reading_status': 'to_read'}`
        *   `get_authors(request)`: `GET`.
            *   Response: JSON list of author names.
        *   `get_category(request)`: `GET`.
            *   Response: JSON list of category names.

    *   **Reading Progress API (`mainpages/views2.py`):**
        *   `update_reading_progress_api(request)`: `POST`. Expects JSON `{'book_id', 'current_page'}`.
            *   Response: `{'status': 'success/error', 'message': '...', 'book_id', 'current_page', 'total_pages', 'reading_status'}`

    *   **AI Agent APIs (`agent_chat/views.py`):**
        *   `initiate_nlp_processing(request)`: `POST`. Expects `query` (form data or JSON).
            *   Response: `{'status': 'success/error', 'stream_id': 'uuid'}`
        *   `stream_nlp_results(request, stream_id)`: `GET`. Server-Sent Event stream.
            *   Events: `status`, `warning`, `error`, `finished`. Data is JSON.

    *   **Search Suggestions API (`mainpages/views.py`):**
        *   `search_items(request)`: `GET`.
            *   Response: Flat JSON list of strings (book/category/author names).

**3.5. User Interface (UI) and User Experience (UX) Design**

    3.5.1. Wireframes/Mockups for Key Pages
    *(This section would typically include actual diagrams/sketches. Below is a textual description of intended layouts based on common web patterns and the implied functionality.)*
        *   **Homepage (`homePage/home.html`):**
            *   Layout: Header (logo, search bar, profile icon/login), main content area (book carousels/grids), possibly a sidebar for categories or filters.
            *   Content: Displays recommended books (initially based on interests, then search results). Each book card shows cover, title, author, possibly rating/likes. "Add to favorites" / "Add to reading list" quick actions.
        *   **Book Detail Page (`homePage/card-detailed-view.html`):**
            *   Layout: Prominent book cover, detailed information section (description, author, category, price, pages, etc.), ratings/sentiment summary, comment section, action buttons (like, favorite, add to reading status).
            *   UX: Clear presentation of all relevant book data. Easy to read reviews and add new ones. "Related books" or "Books by same author" sections could enhance discovery.
        *   **User Profile Page (`homePage/userProfile.html`):**
            *   Layout: User's profile picture, name, contact info (phone), gender. Tabs or sections for: Interests, Favorite Books, Reading Lists (To Read, Reading, Completed), Recently Viewed, My Comments, Followed Authors. "Update Account" / "Change Password" links.
            *   UX: Provides a comprehensive overview of the user's activity and preferences. Easy navigation between different lists.
        *   **AI Chat Interfaces:**
            *   **Book Category Chat (`homePage/gemini.html`):** Standard chat bubble interface. Input field for user to describe interests. Bot responses display category suggestions or book examples. Handled by `scripts.js` and `gemini-bookverse2.js`.
            *   **Book Finder (Image) (`homePage/gemeni-image.html`):** Interface for image upload (file input) or camera capture (video feed, capture button). Preview area for selected/captured image. Result area displays Gemini's analysis of the book. Handled by `scripts-image.js` and `gemini-image.js`.
            *   **Specific Book Chat (`homePage/gemini_specific_book.html`):** Similar to general chat, but pre-filled with an initial prompt about a specific book. User can then ask follow-up questions. Handled by `scripts-specific-book.js` and `gemini2.js`.
            *   **Agentic AI Chat (`homePage/ai_agent.html`):** Chat interface where users type commands to manage their lists. Bot responses stream in, confirming actions or providing information. Handled by SSE and specific JS for `agent_chat`.
            *   UX for Chat: Clear distinction between user and bot messages. Loading indicators during AI processing. Streamed responses for better perceived performance. Error handling for AI failures or misunderstandings. Markdown parsing for rich text responses.
        *   **Registration/Login Pages (`account/signup.html`, `account/login.html`):** Standard form-based interfaces. Clear error messages for validation issues.
        *   **Update Account Pages (e.g., `account/updateUsername.html`, `updateAccount/personalDetails.html`):** Form-based interfaces for specific profile updates.

    3.5.2. Navigation Structure
    *   **Main Navigation (Header):**
        *   Logo (links to Homepage).
        *   Search Bar (with dynamic suggestions).
        *   Links: Home, Discover, Categories, Events, AI Book Finder, AI Agent.
        *   User Profile Icon (dropdown: My Profile, Your Favorites, Reading Lists, Settings, Logout) or Login/Sign Up buttons.
    *   **Side Panel (`sideMenu` in `scripts-specific-book.js`, likely applicable more broadly):** Activated by profile icon, offers quick links to user-specific pages like "Your Comments," "Your Favorites," "Recently Seen," "Liked Books," "Currently Reading," "Want to Read," "Finished Reading."
    *   **Footer:** Copyright, About Us, Contact links (standard).
    *   **Breadcrumbs:** Could be used on deeper pages (e.g., Category -> Book Detail) for easier navigation.

    3.5.3. User Interaction Flows
        *   **New User Registration & Onboarding:**
            1.  User fills registration form (`create_account`).
            2.  Redirected to `update_username` page.
            3.  Redirected to `upload_image` page.
            4.  Redirected to `add_your_interest` page to select initial interests.
            5.  Lands on Homepage with personalized recommendations.
        *   **Book Discovery (Category Chat):**
            1.  User navigates to AI Category Chat.
            2.  Types interests (e.g., "I like sci-fi with spaceships").
            3.  AI (Gemini via `gemini-bookverse2.js`) suggests categories (e.g., "Science Fiction").
            4.  User can ask for book examples in a suggested category.
            5.  User can click on category names to browse books.
        *   **Book Discovery (Image Search):**
            1.  User navigates to AI Book Finder (Image).
            2.  Uploads or captures book cover.
            3.  AI (Gemini Vision via `gemini-image.js`) analyzes and provides book details.
        *   **Managing Lists (Agentic AI Chat):**
            1.  User navigates to AI Agent Chat.
            2.  Types command (e.g., "Add Dune to my to-read list and show my favorites").
            3.  Backend agent (`agent_chat` app) processes:
                a.  Gemini parses command to JSON.
                b.  Django executes actions (updates `ReadingStatus` for Dune, queries `Favorite` list).
                c.  Gemini summarizes results.
            4.  User sees streamed status and final summary.
        *   **Interacting with a Book (Book Detail Page):**
            1.  User views book details.
            2.  Can click "Like" (AJAX to `like_book_view`).
            3.  Can click "Add to Favorite" (AJAX to `toggle_favourite_view`).
            4.  Can select "Add to Reading List" (e.g., 'To Read', AJAX to `add_to_reading_list_view`).
            5.  Can write a comment and give a rating (AJAX to `add_comment_view`).

    3.5.4. Accessibility Considerations (Briefly)
    While not explicitly detailed in code structure, good design practices would include:
    *   Semantic HTML for screen readers.
    *   Sufficient color contrast.
    *   Keyboard navigability for all interactive elements.
    *   ARIA attributes where appropriate for dynamic content and custom controls.
    *   Alternative text for images.

**3.6. Security Considerations**

    3.6.1. Django Security Features (Built-in)
    *   **Cross-Site Scripting (XSS) Protection:** Django templates automatically escape variables, mitigating XSS unless explicitly marked as `|safe`.
    *   **Cross-Site Request Forgery (CSRF) Protection:** Django's CSRF middleware and `{% csrf_token %}` template tag protect against CSRF attacks on POST forms. All state-changing operations should use POST requests with CSRF tokens.
    *   **SQL Injection Protection:** Django's ORM uses parameterized queries, which prevents SQL injection vulnerabilities.
    *   **Clickjacking Protection:** Django provides middleware to help prevent clickjacking.
    *   Secure password hashing is handled by Django's authentication system.

    3.6.2. User Authentication and Authorization
    *   **Authentication:** Handled by `django.contrib.auth`. Sensitive views that require a logged-in user are protected using `@login_required` decorator (though not explicitly shown in all provided `views.py` snippets, it's a standard practice).
    *   **Authorization:** Permissions for actions like creating/editing books or accessing admin features would typically be handled by Django's permission system or custom checks (e.g., `request.user.profile.is_reviewer`). Users should only be able to modify their own profiles, comments, and lists.

    3.6.3. API Key Management (Gemini API Key)
    *   **Client-Side Keys (`gemini2.js`, `gemini-bookverse2.js`, `gemini-image.js`):** Storing API keys directly in client-side JavaScript (`API_KEY = "AIzaSyA..."`) is **highly insecure for production environments**. These keys can be easily extracted by anyone viewing the page source.
        *   **Recommended Production Approach:** For client-side calls, a backend proxy endpoint should be created. The client JS would call this Django endpoint, and the Django endpoint would then securely make the API call to Gemini using an API key stored as an environment variable or in a secure configuration file on the server.
    *   **Server-Side Keys (`agent_chat/views.py`):** The API key (`api_key = 'AIzaSyA...'`) is also hardcoded here. In production, this **must be moved** to an environment variable or a secure configuration file, not committed to version control. Example: `api_key = os.environ.get('GEMINI_API_KEY')`.

    3.6.4. Input Validation
    *   **Django Forms:** `BookForm`, `CategoryForm`, etc., provide server-side validation for data submitted through forms.
    *   **API Endpoints:** Backend views handling AJAX requests (e.g., `add_comment_view`, `update_reading_progress_api`) should rigorously validate all incoming data (e.g., checking `book_id` exists, rating is within range, `current_page` is a valid integer) before processing to prevent errors and potential abuse.
    *   **Query Parameters:** Sanitize and validate any user-supplied query parameters used in database lookups, although Django's ORM helps prevent direct SQL injection.
    *   **File Uploads:** Validate file types and sizes for image uploads to prevent malicious uploads. Django's `ImageField` provides some basic validation.

    3.6.5. Server-Sent Events (SSE) Security
    *   Ensure SSE endpoints require authentication if they stream sensitive user data. `agent_chat/views.py` uses `@login_required` for `stream_nlp_results`, which is good.
    *   Be mindful of potential information leakage if error messages in SSE streams are too verbose.

    3.6.6. Dependencies
    *   Keep all dependencies (Django, Python libraries) up-to-date to patch known vulnerabilities.
