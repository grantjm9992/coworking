# Google Sheets Setup Instructions

Follow these steps to connect your landing page form to Google Sheets.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it: **"Coworking Salamanca - Leads"**
4. In the first row, add these column headers:

```
Timestamp | Name | Email | Phone | Profile | Interest | Comments | User Agent
```

## Step 2: Set up Google Apps Script

1. In your Google Sheet, click **Extensions** → **Apps Script**
2. Delete any code in the editor
3. Copy and paste this code:

```javascript
function doPost(e) {
  try {
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // Parse the incoming data
    var data = JSON.parse(e.postData.contents);

    // Format timestamp for Spanish locale
    var timestamp = new Date(data.timestamp);
    var formattedTimestamp = Utilities.formatDate(timestamp, "Europe/Madrid", "dd/MM/yyyy HH:mm:ss");

    // Append row with data
    sheet.appendRow([
      formattedTimestamp,
      data.name,
      data.email,
      data.phone,
      data.profile,
      data.interest,
      data.comments,
      data.userAgent
    ]);

    // Return success
    return ContentService.createTextOutput(JSON.stringify({
      'result': 'success',
      'message': 'Data added successfully'
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    // Return error
    return ContentService.createTextOutput(JSON.stringify({
      'result': 'error',
      'message': error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

// Optional: Test function
function testDoPost() {
  var testData = {
    postData: {
      contents: JSON.stringify({
        name: 'Test User',
        email: 'test@example.com',
        phone: '600123456',
        profile: 'remote',
        interest: 'fixed-desk',
        comments: 'This is a test',
        timestamp: new Date().toISOString(),
        userAgent: 'Test Browser'
      })
    }
  };

  var result = doPost(testData);
  Logger.log(result.getContent());
}
```

4. Click the **Save** icon (💾) or press Ctrl+S
5. Name the project: **"Coworking Form Handler"**

## Step 3: Deploy as Web App

1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type"
3. Choose **Web app**
4. Configure the deployment:
   - **Description**: "Coworking lead capture v1"
   - **Execute as**: Me (your email)
   - **Who has access**: Anyone
5. Click **Deploy**
6. **Important**: You'll need to authorize the script:
   - Click **Authorize access**
   - Choose your Google account
   - Click **Advanced** → **Go to Coworking Form Handler (unsafe)**
   - Click **Allow**
7. **Copy the Web app URL** - it will look like:
   ```
   https://script.google.com/macros/s/AKfycbx...../exec
   ```

## Step 4: Update Your Landing Page

1. Open `index.html`
2. Find this line (around line 861):
   ```javascript
   const GOOGLE_SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
   ```
3. Replace `YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` with the URL you copied in Step 3
4. Save the file

## Step 5: Test It!

1. Open your `index.html` in a browser
2. Fill out the form with test data
3. Submit
4. Check your Google Sheet - you should see a new row with the data!

## Troubleshooting

### Form submits but no data appears in Google Sheet

1. Go back to Apps Script
2. Click **Executions** (left sidebar)
3. Look for any errors in recent executions
4. Common issues:
   - Wrong deployment URL (make sure it ends with `/exec`)
   - Script not deployed (redo Step 3)
   - Authorization not completed (redo Step 3.6)

### "Mode: no-cors" warning in browser console

- This is normal and expected! The form will still work.

### Test the Apps Script directly

1. In Apps Script, click the **testDoPost** function in the dropdown
2. Click **Run**
3. Check your Google Sheet - a test row should appear

## Get Email Notifications (Optional)

Add this code to your Apps Script to get emailed when someone submits:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);

    var timestamp = new Date(data.timestamp);
    var formattedTimestamp = Utilities.formatDate(timestamp, "Europe/Madrid", "dd/MM/yyyy HH:mm:ss");

    sheet.appendRow([
      formattedTimestamp,
      data.name,
      data.email,
      data.phone,
      data.profile,
      data.interest,
      data.comments,
      data.userAgent
    ]);

    // Send email notification
    var emailBody =
      "Nuevo interesado en Coworking Salamanca!\n\n" +
      "Nombre: " + data.name + "\n" +
      "Email: " + data.email + "\n" +
      "Teléfono: " + data.phone + "\n" +
      "Perfil: " + data.profile + "\n" +
      "Interés: " + data.interest + "\n" +
      "Comentarios: " + data.comments + "\n\n" +
      "Fecha: " + formattedTimestamp;

    MailApp.sendEmail({
      to: Session.getActiveUser().getEmail(), // Change this to your email
      subject: "🎯 Nuevo lead - Coworking Salamanca",
      body: emailBody
    });

    return ContentService.createTextOutput(JSON.stringify({
      'result': 'success',
      'message': 'Data added successfully'
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({
      'result': 'error',
      'message': error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

After adding this, click **Deploy** → **Manage deployments** → **Edit** (pencil icon) → **Version: New version** → **Deploy**

---

## Data You'll Collect

Each form submission will create a row with:

- **Timestamp**: When they submitted (Madrid time)
- **Name**: Full name
- **Email**: Their email address
- **Phone**: Phone number (optional)
- **Profile**: What type of professional they are
- **Interest**: What they're interested in (desk/office)
- **Comments**: Any additional comments
- **User Agent**: Browser/device info (helps identify mobile vs desktop)

## Privacy Note

Make sure you comply with GDPR:
- ✅ You have a privacy notice on the form
- ✅ You're only collecting necessary data
- ✅ You're stating how you'll use the data
- ⚠️ Consider adding: "Al enviar este formulario, acepto la política de privacidad"

---

**Questions?** Check the Google Apps Script documentation or test using the `testDoPost()` function first.
