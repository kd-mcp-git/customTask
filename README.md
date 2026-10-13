# GA4 Fetch Interceptor for Custom Tasks

A lightweight, dependency-free JavaScript utility that intercepts Google Analytics 4 `fetch` requests to enable data modification and augmentation, mimicking the functionality of Universal Analytics' `customTask`.

This script is designed to be deployed via a Custom HTML tag in Google Tag Manager.

## Features

*   **Fetch Interception:** Patches `window.fetch` to inspect and modify GA4 measurement protocol hits.
*   **Task-Based Architecture:** Chain multiple "tasks" to perform a series of modifications on the request payload.
*   **PII Scrubber:** An included task automatically redacts email addresses and phone numbers from all parameters.
*   **Client ID Mapper:** An included task captures the GA4 `client_id` and maps it to a custom event or user parameter.
*   **Safe & Specific:** The interceptor only targets GA4 `/collect` endpoints, leaving all other `fetch` requests untouched.
*   **Unminified & Minified Versions:** Develop with a readable source and deploy a minified version for production.

## Getting Started

### Prerequisites

*   A website using Google Analytics 4, implemented via `gtag.js`.
*   Access to Google Tag Manager for the corresponding property.

### Installation

1.  **Configure the Script:** Open `Core.js` and locate the `SETUP` section at the bottom of the file.
    ```javascript
    // 4. SETUP
    const ga4CustomTaskInterceptor = new GA4CustomTask({
      allowedMeasurementIds: ['G-XXXXXXXXXX'], // <-- Replace with your Measurement ID
      tasks: [
        // Add and configure your desired tasks here
        (req) => mapClientIdTask(req, 'client_id', 'event'),
        piiScrubberTask
      ]
    });
    ```
    *   Update `allowedMeasurementIds` with your GA4 Measurement ID(s). This acts as a whitelist.
    *   Configure the `tasks` array with the functions you wish to run.

2.  **Minify the Code:** For production use in GTM, copy the entire contents of `Core.js` and paste it into a JavaScript minifier (e.g., Terser).

3.  **Deploy in Google Tag Manager:**
    *   In GTM, create a new **Custom HTML** tag.
    *   Paste the **minified** code inside `<script>` tags.
    *   Set the tag to fire on a trigger that loads before your GA4 config tag, such as **Consent Initialization - All Pages**. This ensures the `fetch` function is patched before any GA4 hits are sent.
    *   Save and publish your container.

## Included Tasks

*   `piiScrubberTask`: Scrubs email and phone numbers. No configuration needed.
*   `mapClientIdTask(request, paramName, scope)`: Maps the Client ID to a parameter.
    *   `paramName`: The name for your new custom dimension (e.g., 'client_id').
    *   `scope`: `'event'` (for an event-scoped parameter) or `'user'` (for a user property).

See the **USER_GUIDE.md** for details on creating your own custom tasks.