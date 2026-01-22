# How to Connect Your Signup Form to Google Sheets

Follow these steps to save your waitlist signups directly to a Google Sheet.

## 1. Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new sheet.
2. Name it "Octostore Waitlist".
3. In the first row, add these headers:
   - **Column A:** `timestamp`
   - **Column B:** `name`
   - **Column C:** `email`
   - **Column D:** `usecase`

## 2. Add the Script

1. In your Google Sheet, go to **Extensions** > **Apps Script**.
2. Delete any code in the editor and paste the following code exactly:

```javascript
var sheetName = "Sheet1";
var scriptProp = PropertiesService.getScriptProperties();

function intialSetup() {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty("key", activeSpreadsheet.getId());
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty("key"));
    var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function (header) {
      if (header === "timestamp") {
        return new Date();
      }
      return e.parameter[header];
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService.createTextOutput(
      JSON.stringify({ result: "success", row: nextRow }),
    ).setMimeType(ContentService.MimeType.JSON);
  } catch (e) {
    return ContentService.createTextOutput(
      JSON.stringify({ result: "error", error: e }),
    ).setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

3. Click the **Save** icon (floppy disk).
4. Run the `intialSetup` function:
   - Select `intialSetup` from the dropdown toolbar next to "Debug".
   - Click **Run**.
   - You will see a "Review Permissions" dialog. Click it, select your account, then click **Advanced** > **Go to (Script Name) (unsafe)** > **Allow**.

## 3. Deploy the Script

1. Click the blue **Deploy** button > **New deployment**.
2. Click the **Select type** gear icon > **Web app**.
3. Fill in the details:
   - **Description:** "Waitlist Form"
   - **Execute as:** `Me (your email)`
   - **Who has access:** `Anyone` (This is crucial so your website can send data)
4. Click **Deploy**.
5. **Copy the "Web app URL"**. It will look like `https://script.google.com/macros/s/.../exec`.

## 4. Updates Your Website Code

1. Open `src/pages/signup.astro`.
2. Find the form tag:
   ```html
   <form
     id="waitlist-form"
     class="flex flex-col gap-4"
     method="POST"
     action="YOUR_GOOGLE_SCRIPT_URL_HERE"></form>
   ```
3. Replace `YOUR_GOOGLE_SCRIPT_URL_HERE` with the Web app URL you copied.
4. **Important:** In the `<script>` section at the bottom of `signup.astro`, update the submit handler to actually send the data. Search for `// await fetch` and uncomment it:

   ```javascript
   // Change this section:

   // Simulate network request
   // await new Promise(resolve => setTimeout(resolve, 1000));

   // REAL IMPLEMENTATION (Uncomment this):
   const formData = new FormData(form);
   await fetch(form.action, { method: "POST", body: formData });
   ```

That's it! Your form will now save data directly to your Sheet.
