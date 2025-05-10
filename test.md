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
