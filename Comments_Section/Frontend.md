## 💬 Comment Management System: API Integration & Frontend Interaction

This documentation focuses on the backend API interactions for the comment management system within the file viewer component, detailing how the frontend communicates with the server to perform comment-related operations.

### 🔗 API Service: `BitstreamCommentService`

The `BitstreamCommentService` is an Angular injectable service responsible for all HTTP communication with the backend for managing comments.

* **Base URL**: `http://localhost:8080/server/api/bitstream/comment`
* **Credentials**: All requests are sent `withCredentials: true` to ensure session cookies are included for authentication and authorization.

####  fetching All Comments (`getComments`) 📥

* **Endpoint**: `GET /server/api/bitstream/comment/bitstream/{bitstreamId}`
* **Method**: `getComments(bitstreamId: string): Observable<BitstreamComment[]>`
* **Description**: This method retrieves all comments associated with a specific bitstream (file). The `bitstreamId` is appended to the URL path.
* **Frontend Usage**:
    ```typescript
    loadComments(bitstreamId: string): void {
      this.bitstreamCommentService.getComments(bitstreamId).subscribe({
        next: (res) => {
          this.comments = res; // Assigns the fetched comments to the `comments` array
          this.cdr.detectChanges(); // Triggers change detection to update the UI
        },
        error: (err) => console.error("Failed to load comments", err), // Logs any error during fetching
      });
    }
    ```
    * The `loadComments` function is typically called when the viewer component initializes or when the `bitstreamId` changes.
    * On a successful response, the `comments` array in the component is updated, and change detection is manually triggered (`cdr.detectChanges()`) to refresh the displayed comments.

#### ➕ Adding a New Comment (`addComment`) 🚀

* **Endpoint**: `POST /server/api/bitstream/comment`
* **Method**: `addComment(comment: BitstreamComment): Observable<BitstreamComment>`
* **Description**: This method sends a new comment object to the backend for creation. The `comment` object should contain `bitstreamId` and the `comment` text.
* **Frontend Usage**:
    ```typescript
    addComment(): void {
      const commentText = this.newCommentText.trim();
      if (!commentText || this.isAddingComment) return; // Prevent adding empty or duplicate comments

      this.isAddingComment = true; // Set loading state

      const newComment: BitstreamComment = {
        bitstreamId: this.currentBitstreamId,
        comment: commentText, // The comment text from the textarea
      };

      this.bitstreamCommentService.addComment(newComment).subscribe({
        next: (res) => {
          this.comments.push(res); // Add the newly created comment (with ID, etc.) to the local array
          this.newCommentText = ""; // Clear the input field
          this.isAddingComment = false; // Reset loading state
          this.cdr.detectChanges();
        },
        error: (err) => {
          console.error("Failed to add comment", err);
          this.isAddingComment = false; // Reset loading state even on error
          this.cdr.detectChanges();
        },
      });
    }
    ```
    * Before sending, the `newCommentText` is trimmed, and checks prevent adding empty comments or multiple comments if one is already being added.
    * `isAddingComment` flag manages the button state ("Add Comment" vs. "Adding...").
    * Upon success, the backend's response (which typically includes the assigned `id` and `createdDate`) is pushed to the local `comments` array, the input field is cleared, and loading state is reset.

#### 🗑️ Deleting a Comment (`deleteComment`) ❌

* **Endpoint**: `DELETE /server/api/bitstream/comment/{id}`
* **Method**: `deleteComment(id: number): Observable<any>`
* **Description**: This method sends a request to the backend to delete a comment identified by its `id`.
* **Frontend Usage (with Confirmation Modal)**:
    ```typescript
    // Initiates the deletion process, showing the confirmation modal
    confirmDelete(commentId: number): void {
      this.commentToDelete = commentId; // Store the ID of the comment to be deleted
      this.showDeleteConfirmation = true; // Show the confirmation modal
    }

    // Cancels the deletion process
    cancelDelete(): void {
      this.showDeleteConfirmation = false;
      this.commentToDelete = null; // Clear the stored comment ID
    }

    // Confirms and executes the actual API call for deletion
    confirmDeleteComment(): void {
      if (this.commentToDelete === null) return; // Safeguard if no comment ID is set

      this.deletingCommentId = this.commentToDelete; // Set loading state for the specific comment
      this.showDeleteConfirmation = false; // Close the confirmation modal

      this.bitstreamCommentService.deleteComment(this.commentToDelete).subscribe({
        next: () => {
          // Filter out the deleted comment from the local array
          this.comments = this.comments.filter((c) => c.id !== this.commentToDelete);
          this.deletingCommentId = null; // Reset loading state
          this.commentToDelete = null; // Clear stored ID
          this.cdr.detectChanges();
        },
        error: (err) => {
          console.error("Delete failed", err);
          this.deletingCommentId = null; // Reset loading state on error
          this.commentToDelete = null; // Clear stored ID on error
          this.cdr.detectChanges();
        },
      });
    }
    ```
    * The deletion process involves a two-step confirmation:
        1.  `confirmDelete(commentId)`: Stores the `commentId` and displays a custom confirmation modal.
        2.  `cancelDelete()`: Closes the modal and clears the stored `commentId`.
        3.  `confirmDeleteComment()`: Only called if the user confirms in the modal. It then makes the actual API call.
    * `deletingCommentId` is used to show a "Deleting..." state on the specific delete button while the request is in progress.
    * On success, the comment is removed from the local `comments` array using `filter()`, and loading states are reset.

#### ✏️ Updating a Comment (`updateComment`) - *Note: Not currently used in provided HTML*

* **Endpoint**: `PUT /server/api/bitstream/comment/{id}`
* **Method**: `updateComment(id: number, newText: string): Observable<BitstreamComment>`
* **Description**: This method sends a request to the backend to update the text of an existing comment. It uses `Content-Type: text/plain` as the `newText` is sent directly in the request body.
* **Frontend Usage**: While the `updateComment` method is defined in the service, it is **not currently utilized** in the provided HTML and component logic. If comment editing functionality were to be added, this method would be used.