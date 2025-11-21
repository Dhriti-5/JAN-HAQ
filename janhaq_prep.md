# Jan-Haq Project: Comprehensive Feature Documentation

This document provides a detailed breakdown of all features implemented in the Jan-Haq project, covering the frontend, backend, user experience, and data flow for each module.

---

## 1. Core Technologies

-   **Backend**: Node.js with Express.js framework.
-   **Frontend**: React.js with Vite, using React Router for navigation and TailwindCSS for styling.
-   **Database**: MongoDB, accessed via the official `mongodb` driver.
-   **AI & Embeddings**:
    -   **Semantic Search**: Uses `Xenova/all-MiniLM-L6-v2` model via the `@xenova/transformers` library to generate vector embeddings for search queries.
    -   **AI Explanations & Rewrites**: An external service (OpenRouter) is used for AI-powered features like complaint rewriting and explanations, accessed via the backend.

---

## 2. Home Page

The Home Page serves as the primary landing page, introducing the platform's purpose and core features.

-   **User Experience (UX)**:
    -   A welcoming hero section with the title "Your Rights, Simplified."
    -   A "Get Started" button intelligently navigates authenticated users to their `/dashboard` and guests to the `/login` page.
    -   The page is composed of several sections: Feature Cards, How It Works, Impact Section, and a Final Call-to-Action (CTA).
-   **Frontend (`janhaq-frontend/src/pages/Home.jsx`)**:
    -   The page is built from modular components like `HeroSection.jsx`, `FeatureCard.jsx`, `HowItWorksSection.jsx`, and `FinalCTA.jsx`.
    -   It uses the `useAuth` context to check if a user is authenticated for conditional navigation.
    -   Animations are handled by the `framer-motion` library.
-   **Backend & Database**:
    -   The Home Page is entirely static and does not make any calls to the backend.

---

## 3. About & Contact Pages

These pages provide static information about the project and a way for users to get in touch.

-   **About Page (`janhaq-frontend/src/pages/About.jsx`)**:
    -   **UX**: A multi-section page telling the story of the project, its mission, and the team.
    -   **Frontend**: Assembles several descriptive components like `StorySection`, `JourneySection`, and `TeamSection`. All content is hardcoded.
    -   **Backend**: No interaction.

-   **Contact Page (`janhaq-frontend/src/pages/Contact.jsx`)**:
    -   **UX**: Displays contact details and a form for users to submit messages. A success message is shown upon submission.
    -   **Frontend**: Manages form state with `useState`. The `handleSubmit` function currently simulates a submission without a backend call.
    -   **Backend**: No backend endpoint is connected to the contact form at this time.

---

## 4. User Authentication and Onboarding

Handles user registration, login, logout, and the initial user setup process.

-   **User Experience (UX)**:
    -   **Register**: Users provide a name, email, and password. Upon successful registration, they are immediately guided through a multi-step onboarding process to personalize their experience.
    -   **Onboarding**: A modal form asks for Age Group, Role (e.g., Student, Farmer), and up to 3 Interests (e.g., Healthcare, Education). This is crucial for personalizing the dashboard.
    -   **Login**: Standard email and password login.
    -   **Protected Routes**: Key pages are inaccessible to unauthenticated users, who are redirected to the login page.
-   **Frontend**:
    -   **Context**: `AuthProvider` (`janhaq-frontend/src/context/AuthContext.jsx`) manages global user state (`user`, `isAuthenticated`).
    -   **API Calls**: `registerUser` and `loginUser` in `api.js` handle communication with the backend.
    -   **Onboarding Form (`OnboardingForm.jsx`)**: A multi-step component that captures user profile data and calls the `onComplete` function, which triggers an API call to update the user's profile.
    -   **Routing**: `ProtectedRoute.jsx` checks `isAuthenticated` status to control access to protected pages.
-   **Backend (`backend/server.js`)**:
    -   **Register (`/api/auth/register`)**: Hashes the password using `bcryptjs`, creates a new user document in the `users` collection with a default profile structure, and returns a JSON Web Token (JWT).
    -   **Login (`/api/auth/login`)**: Finds the user, compares the password hash, and returns a JWT.
    -   **Profile Update (`/api/auth/profile` - PUT)**: The onboarding form data is sent to this endpoint to populate the `profile` sub-document for the user.
-   **Database**:
    -   The `users` collection stores documents with `name`, `email`, `password` (hashed), and a nested `profile` object containing `ageGroup`, `role`, and `interests`.

---

## 5. Forgot Password

A secure, multi-step flow for users to reset their password.

-   **UX**:
    1.  User enters their email address. The system verifies if the email exists.
    2.  If the email is valid, the UI transitions to a password reset form.
    3.  User enters and confirms a new password.
    4.  On success, the user is notified and redirected to the login page.
-   **Frontend (`janhaq-frontend/src/pages/ForgotPassword.jsx`)**:
    -   Manages the flow with a `step` state (1 for email, 2 for password).
    -   `handleEmailSubmit` calls the `/api/auth/verify-email` endpoint.
    -   `handlePasswordReset` calls the `/api/auth/reset-password` endpoint.
-   **Backend (`backend/server.js`)**:
    -   **`/api/auth/verify-email` (POST)**: Checks if a user exists with the given email in the `users` collection.
    -   **`/api/auth/reset-password` (POST)**: Finds the user by email and updates their `password` field with a new hash.
-   **Database**: This feature reads from and writes to the `users` collection.

---

## 6. Laws, Schemes, and Departments

These pages serve as browsable libraries of static and dynamic information.

-   **UX**:
    -   Laws and Schemes are presented in a card format with a real-time search filter.
    -   Departments are displayed in cards showing contact details.
-   **Frontend**:
    -   **Laws (`Laws.jsx`) & Schemes (`Schemes.jsx`)**: Fetch data from static JSON files (`all_laws.json`, `schemes.json`) via backend endpoints. The `useState` hook manages the list and a `filtered` version for the search.
    -   **Departments (`Departments.jsx`)**: Fetches a list of departments from the database by calling the `getDepartments` function in `api.js`.
-   **Backend (`backend/server.js`)**:
    -   **`/api/laws` & `/api/schemes`**: Read and return data from static JSON files.
    -   **`/api/departments`**: Fetches and returns all documents from the `departments` collection in MongoDB.
-   **Database**:
    -   Laws and Schemes data is static (JSON files).
    -   Departments data is dynamic and stored in the `departments` collection.

---

## 7. Problem Solver (AI Search)

An AI-powered search tool that provides users with relevant laws and schemes based on a natural language query.

-   **UX**:
    -   User enters a problem description (e.g., "unfair dismissal from job").
    -   The system displays a list of relevant laws and schemes, ranked by relevance.
    -   An "Explain" button provides an AI-generated summary of the item.
-   **Frontend (`ProblemSolver.jsx`)**:
    -   `handleSearch` calls the `/search` API endpoint.
    -   `handleExplain` calls the `/api/explain` API endpoint and renders the returned Markdown as HTML using the `marked` library.
-   **Backend (`backend/server.js`)**:
    -   **`/search`**:
        1.  Generates a vector embedding for the user's query using the `Xenova/all-MiniLM-L6-v2` model.
        2.  Calculates the cosine similarity between the query vector and the pre-computed vectors of all items in `knowledge_base.json`.
        3.  Returns the top matching items sorted by score.
    -   **`/api/explain`**: Sends the item's title and description to the OpenRouter AI service with a prompt to explain it in simple terms.
-   **Data**: The core of this feature is `knowledge_base.json`, which contains laws and schemes along with their pre-computed vector embeddings.

---

## 8. Dashboard & Recommendations

The central hub for authenticated users, providing personalized content.

-   **UX**:
    -   Greets the user by name.
    -   Prompts the user to complete their profile if they haven't, linking to the profile page.
    -   Displays a "Recommended for You" section with relevant laws and schemes if the profile is complete.
    -   Provides "Quick Access" cards for major features.
-   **Frontend (`Dashboard.jsx`)**:
    -   Uses the `user` object from `useAuth` to display the name and check profile completeness.
    -   If the profile is complete, it calls `getMyRecommendations` to fetch personalized content.
-   **Backend (`backend/server.js`)**:
    -   **`/api/recommendations` (GET)**: This protected route identifies the user via their JWT. It retrieves the user's `role` and `interests` from their profile, then scores items in the `knowledge_base.json` based on matching tags. Items with a higher score (e.g., matching role and interests) are returned.
-   **Database**: The recommendation logic relies on the `profile` object within the user's document in the `users` collection.

---

## 9. Complaint Generator & Tracking

An AI-assisted tool to draft and track formal complaints.

-   **UX**:
    -   **Generator (`ComplaintGenerator.jsx`)**: A form where the user selects a department and describes their issue. An AI rewrites the description into a formal complaint, which the user can then submit.
    -   **Tracking (`MyComplaints.jsx`)**: A table lists all complaints filed by the user with their status. A "View" button opens a modal with the full complaint details.
-   **Frontend**:
    -   `handleGenerate` in `ComplaintGenerator.jsx` calls the `rewriteComplaint` API.
    -   `handleSubmitComplaint` calls the `submitComplaint` API and redirects to `/my-complaints`.
    -   `MyComplaints.jsx` fetches the user's complaint history via the `getComplaints` API.
-   **Backend (`backend/server.js`)**:
    -   **`/api/rewriteComplaint` (POST)**: Forwards the user's description to the OpenRouter AI service with a prompt to rewrite it formally.
    -   **`/api/complaints` (POST)**: A protected route that saves the complete complaint to the `complaints` collection, linking it to the user via their `userId`.
    -   **`/api/complaints` (GET)**: A protected route that queries the `complaints` collection for all documents matching the current `userId`.
    -   **`/api/complaints/:id` (GET)**: Fetches a single complaint by its ID, ensuring it belongs to the current user.
-   **Database**: A `complaints` collection stores documents containing `userId`, department details, original and formal text, status, and timestamps.

---

## 10. Profile Management

Allows users to view and update their personal information.

-   **UX**:
    -   A form pre-filled with the user's data.
    -   Users can update their name, password, and other details.
    -   An "Edit Personalization" button opens the `OnboardingForm.jsx` component in an "edit mode," allowing them to change their role and interests.
-   **Frontend (`Profile.jsx`)**:
    -   Fetches profile data using `getUserProfile`.
    -   Saves changes using `updateUserProfile` and `changePassword` from `api.js`.
    -   Toggles the `OnboardingForm` for editing personalization settings.
-   **Backend (`backend/server.js`)**:
    -   **`/api/auth/profile` (PUT)**: Updates the user's `profile` sub-document.
    -   **`/api/auth/change-password` (PUT)**: Verifies the current password and updates it with a new hash.
-   **Database**: This feature directly reads from and writes to the user's document in the `users` collection.

---

## 11. Saved Items (Bookmarking)

Allows users to bookmark laws, schemes, and search results.

-   **UX**:
    -   A bookmark icon on item cards allows users to save or unsave an item.
    -   If a user is not logged in, a modal prompts them to log in first.
    -   The `SavedLaws.jsx` page displays all bookmarked items, with an option to remove them.
-   **Frontend**:
    -   `handleSaveClick` calls the `saveItem` or `unsaveItem` API.
    -   `SavedLaws.jsx` fetches all items using `getSavedItems`.
-   **Backend (`backend/server.js`)**:
    -   The user's saved items are stored in a `savedItems` array directly within their user document in the `users` collection.
    -   **`/api/saved-items` (POST)**: Uses `$push` to add a new item object to the `savedItems` array for the current user.
    -   **`/api/saved-items` (GET)**: Retrieves the `savedItems` array from the user's document.
    -   **`/api/saved-items/:itemId` (DELETE)**: Uses `$pull` to remove an item from the `savedItems` array based on its `itemId`.
-   **Database**: This feature modifies the `users` collection, not a separate `saved_items` collection.

---

## 12. Minor Features & UI

-   **Notifications Page (`Notifications.jsx`)**:
    -   A dedicated page for notifications.
    -   Currently, it displays a static list of notifications from a mock data file (`mockData.js`). It is not connected to a backend system.

-   **Theme Toggle (`ThemeToggle.jsx`)**:
    -   A UI component, present in the navbar, that allows users to switch between light and dark modes.
    -   It uses `localStorage` to persist the user's theme preference across sessions.
    -   It also respects the user's operating system preference (`prefers-color-scheme`) on the first visit.
