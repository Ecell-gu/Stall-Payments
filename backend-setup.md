# Backend Setup (Google Apps Script)

A simple, free backend using Google Sheets to track payment clicks.

## Step 1: Create a Google Sheet
1. Go to Google Sheets and create a new spreadsheet named "Stall Payments".
2. Set the Headers in the first row: `Timestamp` in A1, `Stall ID` in B1, `Stall Name` in C1, and `Amount` in D1.

## Step 2: Add Apps Script
1. In the Google Sheet, go to **Extensions > Apps Script**.
2. Replace the default code with this snippet:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);
    
    // Append the row mapping to [Timestamp, Stall ID, Stall Name, Amount]
    sheet.appendRow([data.timestamp, data.stallId, data.stallName, data.amount]);
    
    // Return success
    return ContentService.createTextOutput(JSON.stringify({ "status": "success" }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    // Return error
    return ContentService.createTextOutput(JSON.stringify({ "status": "error", "message": error.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

## Step 3: Deploy as Web App
1. Click the **Deploy** button on the top right, then **New deployment**.
2. Under "Select type", click the gear icon and choose **Web app**.
3. Under "Description", type something like "Payment API".
4. Under "Execute as", select **Me**.
5. Under "Who has access", select **Anyone** (This allows our fetch request to post data without authentication).
6. Click **Deploy**. (You will need to authorize permissions on the next screen).
7. Copy the generated **Web app URL**.

## Step 4: Connect to your App
1. Go to `index.html` in your codebase.
2. Find the constant `WEBHOOK_URL` in the `<script>` section.
3. Replace the empty string with your copied Web app URL:
```javascript
const WEBHOOK_URL = 'https://script.google.com/macros/s/AKfycbyj5g9jxVRY7H6mwMwi0mxJXUDN3qoUNyU_3MtQKFP6xVIJSKawLnF07smBTDiVr1ca1A/exec';
```