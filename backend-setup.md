# Backend Setup (Google Apps Script)

A simple, free backend using Google Sheets to track payment clicks.

## Step 1: Create a Google Sheet
1. Go to Google Sheets and create a new spreadsheet named "Stall Payments".
2. Set the Headers in the first row: `Timestamp`, `Stall ID`, `Stall Name`, `Amount`, `Screenshot URL`, and `Status`.

## Step 2: Add Apps Script
1. In the Google Sheet, go to **Extensions > Apps Script**.
2. Replace the default code with this snippet:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);
    
    // Process screenshot if present
    var screenshotUrl = "No Screenshot";
    if (data.image) {
      var decode = Utilities.base64Decode(data.image);
      // Give the file a structured name format: StallID_timestamp.ext
      var fileName = data.stallId + "_" + new Date().getTime();
      var blob = Utilities.newBlob(decode, data.mimeType, fileName);
      
      // Uploads to root Google Drive folder
      var file = DriveApp.createFile(blob);
      // Make it viewable by anyone with the link
      file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
      screenshotUrl = file.getUrl();
    }

    // Append the row mapping to [Timestamp, Stall ID, Stall Name, Amount, Screenshot URL, Status]
    sheet.appendRow([data.timestamp, data.stallId, data.stallName, data.amount, screenshotUrl, data.status]);
    
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

### How to Update Apps Script Without Changing the URL
If you change the Apps Script code, do **not** create a new deployment (otherwise the URL changes). Instead:
1. Click **Deploy > Manage deployments**.
2. Select your active deployment from the left menu.
3. Click the pencil/edit icon on the right side.
4. Under the **Version** dropdown, select **New version**.
5. Click **Deploy**. This updates your live app while keeping the same Web App URL!

## Step 4: Connect to your App
1. Go to `index.html` in your codebase.
2. Find the constant `WEBHOOK_URL` in the `<script>` section.
3. Replace the empty string with your copied Web app URL:
```javascript
const WEBHOOK_URL = 'https://script.google.com/macros/s/AKfycbxnw5ZTEa-EJm_F8SKuyndTVAOk0XYq4lGrFYmmTGe2I2L8iDT1Wlf6KgEHrhHu-XRjNQ/exec';
```