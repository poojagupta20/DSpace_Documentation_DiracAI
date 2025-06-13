To ensure your file viewer component handles comments effectively, here's a detailed explanation of the `BitstreamCommentService` and the relevant parts of your `ViewerComponent`, along with instructions for implementation.

---

### `BitstreamCommentService` (src/app/core/serachpage/bitstream-comment.service.ts)

This service acts as the communication layer between your Angular application and the backend API for managing bitstream comments. It provides methods for fetching, adding, updating, and deleting comments.

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';

// Defines the structure for a bitstream comment.
// 'id' and 'comment' are optional for requests (when adding a new comment).
// 'text' is an alternative field for the comment content, often used in responses.
export interface BitstreamComment {
  id?: number;
  bitstreamId: string;
  comment?: string; // Request field
  text?: string;    // Response field
}

@Injectable({ providedIn: 'root' })
export class BitstreamCommentService {
  // Base URL for the bitstream comment API endpoints.
  private baseUrl = 'http://localhost:8080/server/api/bitstream/comment';

  constructor(private http: HttpClient) {}

  /**
   * Fetches all comments associated with a specific bitstream.
   * @param bitstreamId The ID of the bitstream.
   * @returns An Observable of an array of BitstreamComment.
   */
  getComments(bitstreamId: string): Observable<BitstreamComment[]> {
    return this.http.get<BitstreamComment[]>(`${this.baseUrl}/bitstream/${bitstreamId}`, {
      withCredentials: true // Ensures cookies (like session IDs) are sent with the request.
    });
  }

  /**
   * Adds a new comment to a bitstream.
   * @param comment The BitstreamComment object to add.
   * @returns An Observable of the added BitstreamComment.
   */
  addComment(comment: BitstreamComment): Observable<BitstreamComment> {
    return this.http.post<BitstreamComment>(this.baseUrl, comment, {
      withCredentials: true
    });
  }

  /**
   * Updates an existing comment.
   * @param id The ID of the comment to update.
   * @param newText The new text content for the comment.
   * @returns An Observable of the updated BitstreamComment.
   */
  updateComment(id: number, newText: string): Observable<BitstreamComment> {
    // Sets the Content-Type header to 'text/plain' as the payload is raw text.
    const headers = new HttpHeaders({ 'Content-Type': 'text/plain' });
    return this.http.put<BitstreamComment>(`${this.baseUrl}/${id}`, newText, {
      headers,
      withCredentials: true
    });
  }

  /**
   * Deletes a comment by its ID.
   * @param id The ID of the comment to delete.
   * @returns An Observable of any (the backend might return an empty object or a confirmation).
   */
  deleteComment(id: number): Observable<any> {
    return this.http.delete(`${this.baseUrl}/${id}`, {
      withCredentials: true
    });
  }
}
```

---

### `ViewerComponent` (src/app/viewer/viewer.component.ts)

This component is responsible for displaying the file and its associated comments. It interacts with the `BitstreamCommentService` to manage comment data.

```typescript
import {
  Component,
  OnInit,
  ElementRef,
  ViewChild,
  AfterViewInit,
  ChangeDetectorRef,
  OnDestroy,
} from "@angular/core";
import { ActivatedRoute } from "@angular/router";
import { CommonModule } from "@angular/common";
import { ViewEncapsulation } from "@angular/core";
import { Location } from "@angular/common";
import { HttpClient } from "@angular/common/http";
import { interval, Subscription } from "rxjs";
import { FormsModule } from "@angular/forms";
import { BitstreamCommentService, BitstreamComment } from "src/app/core/serachpage/bitstream-comment.service"; // 👈 Import the service and interface

@Component({
  selector: "app-viewer",
  templateUrl: "./viewer.component.html",
  styleUrls: ["./viewer.component.scss"],
  standalone: true,
  imports: [CommonModule, FormsModule], // 👈 Ensure FormsModule is imported for ngModel
  encapsulation: ViewEncapsulation.None,
})
export class ViewerComponent implements OnInit, AfterViewInit, OnDestroy {
  // ... existing properties ...

  comments: BitstreamComment[] = []; // 👈 Array to hold comments
  newCommentText = ""; // 👈 Binds to the new comment input
  isAddingComment = false; // 👈 Flag to manage comment addition state

  // ✅ Custom confirmation modal properties
  showDeleteConfirmation = false; // 👈 Controls visibility of the delete confirmation modal
  commentToDelete: number | null = null; // 👈 Stores the ID of the comment to be deleted
  deletingCommentId: number | null = null; // 👈 Flag to show loading state for a specific comment deletion

  constructor(
    private bitstreamCommentService: BitstreamCommentService, // 👈 Inject the service
    private cdr: ChangeDetectorRef, // 👈 Inject ChangeDetectorRef for manual change detection
    // ... other injected services ...
  ) {}

  // ... existing lifecycle hooks and methods ...

  /**
   * Loads comments for the current bitstream.
   * @param bitstreamId The ID of the bitstream for which to load comments.
   */
  loadComments(bitstreamId: string): void {
    this.bitstreamCommentService.getComments(bitstreamId).subscribe({
      next: (res) => {
        this.comments = res;
        this.cdr.detectChanges(); // Manually trigger change detection after comments are loaded
      },
      error: (err) => console.error("Failed to load comments", err),
    });
  }

  /**
   * Adds a new comment to the bitstream.
   * Only administrators can add comments.
   */
  addComment(): void {
    const comment = this.newCommentText.trim();
    if (!comment || this.isAddingComment) return; // Prevent adding empty comments or multiple comments simultaneously

    this.isAddingComment = true; // Set adding state

    const newComment: BitstreamComment = {
      bitstreamId: this.currentBitstreamId,
      comment: comment, // Use the 'comment' field for the request body
    };

    this.bitstreamCommentService.addComment(newComment).subscribe({
      next: (res) => {
        this.comments.push(res); // Add the newly added comment to the list
        this.newCommentText = ""; // Clear the input field
        this.isAddingComment = false; // Reset adding state
        this.cdr.detectChanges();
      },
      error: (err) => {
        console.error("Failed to add comment", err);
        this.isAddingComment = false; // Reset adding state on error
        this.cdr.detectChanges();
      },
    });
  }

  /**
   * Initiates the delete confirmation process for a comment.
   * @param commentId The ID of the comment to be deleted.
   */
  confirmDelete(commentId: number): void {
    this.commentToDelete = commentId; // Store the ID of the comment to delete
    this.showDeleteConfirmation = true; // Display the confirmation modal
  }

  /**
   * Cancels the comment deletion process, hiding the confirmation modal.
   */
  cancelDelete(): void {
    this.showDeleteConfirmation = false; // Hide the confirmation modal
    this.commentToDelete = null; // Clear the stored comment ID
  }

  /**
   * Confirms and proceeds with the deletion of the selected comment.
   */
  confirmDeleteComment(): void {
    if (this.commentToDelete === null) return; // Ensure there's a comment selected for deletion

    this.deletingCommentId = this.commentToDelete; // Set loading state for the specific comment
    this.showDeleteConfirmation = false; // Hide the confirmation modal immediately

    this.bitstreamCommentService.deleteComment(this.commentToDelete).subscribe({
      next: () => {
        // Remove the deleted comment from the local array
        this.comments = this.comments.filter((c) => c.id !== this.commentToDelete);
        this.deletingCommentId = null; // Clear loading state
        this.commentToDelete = null; // Clear the stored comment ID
        this.cdr.detectChanges();
      },
      error: (err) => {
        console.error("Delete failed", err);
        this.deletingCommentId = null; // Clear loading state on error
        this.commentToDelete = null; // Clear the stored comment ID on error
        this.cdr.detectChanges();
      },
    });
  }
}
```

---

### `viewer.component.html` (src/app/viewer/viewer.component.html)

This template integrates the comments panel, displaying existing comments, providing a form to add new ones, and including a custom confirmation modal for deletions.

```html
<div class="viewer-container">
  <div class="comment-panel">
    <h4>Comments</h4>

    <div *ngIf="comments.length === 0" class="no-comments">
      <p style="color: #666; font-style: italic;">No comments yet.</p>
    </div>

    <div *ngFor="let comment of comments" class="comment-box">
      <div class="comment-header">
        <span class="comment-author">{{ comment.commenterName || 'Anonymous' }}</span>
        <span class="comment-date">{{ comment.createdDate | date: 'medium' }}</span>
      </div>
      <div class="comment-body">
        <p>{{ comment.comment || comment.text }}</p>
      </div>

      <div class="comment-actions" *ngIf="isAdmin">
        <button class="delete-button"
                [disabled]="deletingCommentId === comment.id"
                (click)="confirmDelete(comment.id!)">
          {{ deletingCommentId === comment.id ? 'Deleting...' : 'Delete' }}
        </button>
      </div>
    </div>

    <div class="add-comment-box">
      <textarea [(ngModel)]="newCommentText"
                placeholder="Add comment..."
                [disabled]="!isAdmin || isAddingComment"
                rows="3">
      </textarea>
      <button (click)="addComment()"
              [disabled]="!isAdmin || isAddingComment || !newCommentText.trim()">
        {{ isAddingComment ? 'Adding...' : 'Add Comment' }}
      </button>

      <div *ngIf="!isAdmin" style="margin-top: 8px; font-size: 12px; color: #666; font-style: italic;">
        Only administrators can add comments.
      </div>
    </div>
  </div>

</div>

<div class="modal-overlay" *ngIf="showDeleteConfirmation" (click)="cancelDelete()">
  <div class="confirmation-modal" (click)="$event.stopPropagation()">
    <div class="modal-header">
      <h3>Confirm Delete</h3>
    </div>
    <div class="modal-body">
      <p>Are you sure you want to delete this comment?</p>
      <p class="warning-text">This action cannot be undone.</p>
    </div>
    <div class="modal-footer">
      <button class="btn-cancel" (click)="cancelDelete()">Cancel</button>
      <button class="btn-delete" (click)="confirmDeleteComment()">Delete</button>
    </div>
  </div>
</div>
```

---

### Key Features & Explanations:

#### 1. `BitstreamCommentService` 🛠️
* **`@Injectable({ providedIn: 'root' })`**: Makes the service a singleton and available throughout your application.
* **`HttpClient`**: Used to make HTTP requests to your backend API.
* **`baseUrl`**: The base endpoint for all comment-related API calls.
* **`BitstreamComment` Interface**: Defines the structure of a comment object, useful for type safety. Note the `comment` (for sending data) and `text` (for receiving data) fields, which might be used based on your backend's API contract.
* **Methods**:
    * `getComments(bitstreamId: string)`: Retrieves all comments for a given bitstream ID.
    * `addComment(comment: BitstreamComment)`: Sends a POST request to create a new comment.
    * `updateComment(id: number, newText: string)`: Sends a PUT request to modify an existing comment.
    * `deleteComment(id: number)`: Sends a DELETE request to remove a comment.
* **`withCredentials: true`**: Crucial for sending cookies (like session IDs) with your requests, enabling proper authentication and authorization with your backend.

#### 2. `ViewerComponent` Logic 🧠
* **`comments: BitstreamComment[]`**: An array to store the fetched comments, which is then rendered in the template.
* **`newCommentText`**: A `ngModel`-bound property to capture the text entered by the user for a new comment.
* **`isAddingComment` & `deletingCommentId`**: Boolean flags to manage the UI state during API calls (e.g., showing "Adding..." or "Deleting..." text on buttons to prevent multiple submissions).
* **`loadComments(bitstreamId: string)`**: Calls the `getComments` method of the service and updates the `comments` array.
* **`addComment()`**:
    * Takes the `newCommentText`, constructs a `BitstreamComment` object, and calls the `addComment` service method.
    * Upon successful addition, it pushes the new comment to the local `comments` array and clears the input field.
    * **Admin Check**: The `addComment` button and textarea are disabled if `isAdmin` is false, ensuring only authorized users can add comments.
* **`confirmDelete(commentId: number)`**: This method is part of your **custom confirmation modal**. It stores the ID of the comment to be deleted and sets `showDeleteConfirmation` to `true` to display the modal.
* **`cancelDelete()`**: Hides the confirmation modal and clears the `commentToDelete` value.
* **`confirmDeleteComment()`**:
    * This is called when the user clicks "Delete" in the confirmation modal.
    * It uses `deletingCommentId` to indicate that a specific comment is being deleted, showing "Deleting..." on the button.
    * Calls the `deleteComment` service method.
    * On success, it filters the `comments` array to remove the deleted comment from the UI.
* **`ChangeDetectorRef` (`cdr`)**: Injected and used with `this.cdr.detectChanges()` to ensure that the Angular change detection mechanism updates the UI when asynchronous operations (like fetching or adding comments) complete, especially if you're working with `OnPush` change detection strategy or experiencing UI update issues.

#### 3. `viewer.component.html` UI Elements 🎨
* **Comments Panel**: A dedicated `div` on the right side of the viewer to display comments.
* **`*ngIf="comments.length === 0"`**: Displays a "No comments yet" message when there are no comments.
* **`*ngFor="let comment of comments"`**: Iterates over the `comments` array to render each comment.
    * Each comment displays the `commenterName` (or 'Anonymous') and `createdDate`.
    * The actual comment text is displayed using `comment.comment || comment.text`, providing a fallback.
* **Comment Actions (`*ngIf="isAdmin"`)**: The "Delete" button is only visible to administrators.
    * The `[disabled]` attribute for the delete button uses `deletingCommentId === comment.id` to prevent multiple deletion requests for the same comment and show a loading state.
    * `confirmDelete(comment.id!)` is called on button click to trigger the confirmation modal.
* **Add Comment Form**:
    * `textarea` with `[(ngModel)]="newCommentText"` for two-way data binding.
    * `[disabled]="!isAdmin || isAddingComment"`: The textarea and button are disabled if the user is not an admin or if a comment is currently being added.
    * `addComment()` is called when the "Add Comment" button is clicked.
    * A message clearly states that only administrators can add comments.
* **Custom Confirmation Modal (`.modal-overlay`, `.confirmation-modal`)**:
    * `*ngIf="showDeleteConfirmation"` controls its visibility.
    * It prevents propagation of clicks on the modal content itself (`(click)="$event.stopPropagation()"`) so that clicking inside doesn't close it.
    * Provides "Cancel" and "Delete" buttons linked to `cancelDelete()` and `confirmDeleteComment()` respectively.

---

### Implementation Steps:

1.  **Backend API**: Ensure your backend API for `/server/api/bitstream/comment` is fully functional and matches the expected request/response formats for `GET`, `POST`, `PUT`, and `DELETE` operations, including handling `withCredentials` for authentication.
2.  **Dependencies**:
    * Make sure `@angular/common/http` is imported in your `app.module.ts` (if not standalone) or directly in `ViewerComponent` (as it is currently).
    * Ensure `FormsModule` is imported in `ViewerComponent` for `[(ngModel)]` to work.
3.  **Authentication/Authorization**: The `isAdmin` property in your `ViewerComponent` needs to be correctly populated based on the logged-in user's roles or permissions. This is crucial for controlling who can add or delete comments.
4.  **Error Handling**: Expand on the basic `console.error` for more robust error handling (e.g., displaying user-friendly messages, logging to a monitoring service).
5.  **Loading Comments on Init**: Remember to call `this.loadComments(this.currentBitstreamId)` within your `ngOnInit` or `ngAfterViewInit` method in the `ViewerComponent` to fetch existing comments when the component loads. You will need to ensure `this.currentBitstreamId` is correctly set.




By following this documentation, you can confidently manage bitstream comments within your Angular file viewer application.