4.5.4. **Database Configuration:**
    *   **SQLite:** If SQLite was retained for the initial deployment (common for smaller projects on PythonAnywhere's free/basic tiers), the `DATABASES` setting in `settings.py` pointing to the SQLite file (`db.sqlite3` in the project root) would work directly. The SQLite file would be part of the deployed codebase.
    *   **MySQL/PostgreSQL (PythonAnywhere Hosted):** For more robust database needs, PythonAnywhere offers hosted MySQL or PostgreSQL instances. If one of these was used:
        1.  A database was created via the "Databases" tab in the PythonAnywhere dashboard.
        2.  The `DATABASES` configuration in `settings.py` was updated with the credentials (host, name, user, password) provided by PythonAnywhere for the hosted database.
        3.  `python manage.py migrate` was run from a PythonAnywhere Bash console within the project's virtual environment to set up the schema in the production database.

4.5.5. **Static and Media Files Handling:**
    *   **Static Files (CSS, JS, Images for templates):**
        1.  `STATIC_URL = '/static/'` was set in `settings.py`.
        2.  `STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')` (or a similar path) was configured in `settings.py`.
        3.  `python manage.py collectstatic` was run in a PythonAnywhere Bash console. This command gathers all static files from the apps and project into the `STATIC_ROOT` directory.
        4.  In the PythonAnywhere "Web" tab, a static files mapping was configured:
            *   URL: `/static/`
            *   Directory: The path to `STATIC_ROOT` (e.g., `/home/<username>/<your_project_directory>/staticfiles`).
    *   **Media Files (User Uploads - e.g., profile pictures, book covers):**
        1.  `MEDIA_URL = '/media/'` was set in `settings.py`.
        2.  `MEDIA_ROOT = os.path.join(BASE_DIR, 'media')` was configured in `settings.py`.
        3.  A static files mapping was also set up for media files in the PythonAnywhere "Web" tab:
            *   URL: `/media/`
            *   Directory: The path to `MEDIA_ROOT` (e.g., `/home/<username>/<your_project_directory>/media`).
        *   The `media` directory needed appropriate write permissions for the web server user.

4.5.6. **Environment Variables for Sensitive Information:**
    *   Crucially, sensitive information like Django's `SECRET_KEY` and the `GEMINI_API_KEY` should **not** be hardcoded in `settings.py` or other Python files for a production deployment.
    *   PythonAnywhere allows setting environment variables via the "Web" tab under the "Environment variables" section.
    *   The code was modified to read these from environment variables:
