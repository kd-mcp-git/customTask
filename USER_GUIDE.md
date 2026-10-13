# User Guide: GA4 Custom Task Interceptor

This guide provides detailed instructions for configuring and extending the GA4 Custom Task Interceptor.

## Table of Contents

1.  Configuration
2.  Understanding the Request Object
3.  Creating a Custom Task
4.  Example: Custom Task for Renaming an Event

---

## 1. Configuration

All configuration happens within the `SETUP` block at the end of the `Core.js` file.

```javascript
const ga4CustomTaskInterceptor = new GA4CustomTask({
  allowedMeasurementIds: ['G-XXXXXXXXXX', 'G-YYYYYYYYYY'],
  tasks: [
    (req) => mapClientIdTask(req, 'client_id', 'event'),
    piiScrubberTask,
    myCustomTask // Add your own tasks here
  ]
});
```

*   **`allowedMeasurementIds`**: An array of strings. The interceptor will only process hits destined for a GA4 Measurement ID included in this array. All other hits (including those to other GA4 properties) are ignored.
*   **`tasks`**: An array of functions. Each function in this array is a "task" that will be executed sequentially.
    *   Each task receives a `request` object.
    *   Each task **must** return the `request` object (or a modified version of it) for the next task in the chain.

---

## 2. Understanding the Request Object

When you create a custom task, you will receive a `request` object with the following structure:

```javascript
{
  "endpoint": "https://www.google-analytics.com/g/collect",
  "sharedPayload": {
    "v": "2",
    "tid": "G-XXXXXXXXXX",
    "gtm": "45je3a40",
    "cid": "12345.67890",
    // ... other parameters common to all events in the hit
  },
  "events": [
    {
      "en": "page_view",
      "ep.page_location": "https://www.example.com/",
      // ... other event-specific parameters
    }
    // If the hit is a batch, there may be more event objects
  ]
}
```

*   **`sharedPayload`**: An object containing all the query parameters that are common to every event in the hit (e.g., `tid`, `cid`, `sid`).
*   **`events`**: An array of objects, where each object represents a single event. For standard `page_view` or `click` events, this array will usually contain only one object. For batched requests (like on page unload), it may contain multiple.

---

## 3. Creating a Custom Task

A custom task is simply a function that accepts the `request` object and returns it after performing modifications.

### Template for a Custom Task

```javascript
const myCustomTask = (request) => {
  // 1. Loop through each event in the hit
  request.events.forEach(event => {
    // 2. Add your logic here. For example, read or modify parameters.
    // Example: Add a static parameter to every event
    event['ep.custom_task_ran'] = 'true';
  });

  // 3. Always return the request object
  return request;
};
```

---

## 4. Example: Custom Task for Renaming an Event

Let's say you want to rename the `scroll` event to `scroll_90_percent` only when the `epn.percent_scrolled` parameter is `90`.

```javascript
const renameScrollEventTask = (request) => {
  request.events.forEach(event => {
    // Check if it's the right event and the condition is met
    if (event.en === 'scroll' && event['epn.percent_scrolled'] === '90') {
      // Modify the event name
      event.en = 'scroll_90_percent';
    }
  });
  return request;
};

// Then, add it to your tasks array in the setup block:
tasks: [
  piiScrubberTask,
  renameScrollEventTask // Add the new task
]
```