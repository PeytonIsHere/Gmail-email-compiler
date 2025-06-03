# 📬 Google Form to Gmail Drafts (BCC Email Sender)

This project allows you to collect email addresses via a Google Form and automatically create a Gmail draft with all collected emails in **BCC**. A checkbox in the linked Google Sheet triggers the draft creation. Great for sending clean group emails without revealing recipients.

---

## 🚀 What It Does

- Collects emails via a **Google Form**.
- Stores them in a **Google Sheet**.
- On checking a box, generates a **Gmail draft** with all emails BCC'd.

---

## 🛠 Setup Instructions

### 1. 📝 Create the Google Form

1. Go to [Google Forms](https://forms.google.com) and **create a new form**.
2. Add **one question**:
   - Change `Multiple choice` to `Short Answer`
   - Click the **three dots (⋮)** > **Response validation**:
     - First dropdown: `Text`
     - Second dropdown: `Email`
3. Add a description if desired.
4. In the **Responses** tab:
   - Click the green Sheets icon to **link to a spreadsheet**.
   - Create a **new spreadsheet** (e.g., `Email List`).
5. Make sure the form is **shareable** via link (click “Send” > link icon).

---

### 2. 📊 Set Up the Google Sheet

1. Open the linked **Google Sheet**.
2. In cell **C1**, go to `Insert → Checkbox`.
3. Optional: Rename the sheet tab for clarity (e.g., `Email List`).

---

### 3. 💻 Add the Apps Script

1. In the Sheet, go to `Extensions → Apps Script`.
2. Replace any existing code with the following:

```javascript
function onEdit(e) {
  const sheet = e.source.getActiveSheet();
  const range = e.range;
  const checkboxCell = sheet.getRange('C1');
  
  if (range.getA1Notation() == checkboxCell.getA1Notation()) {
    const isChecked = checkboxCell.getValue();
    
    if (isChecked) {
      createEmailDraft();
    }
  }
}

function createEmailDraft() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const lastRow = sheet.getLastRow();

  const emails = sheet.getRange(2, 2, lastRow - 1).getValues()
    .flat()
    .filter(email => email !== '');

  if (emails.length === 0) {
    SpreadsheetApp.getUi().alert('⚠️ No emails found.');
    return;
  }

  const uniqueEmails = [...new Set(emails)];
  const recipients = uniqueEmails.join(', ');

  const subject = "Email Test";
  const body = "Test email body content here.";

  GmailApp.createDraft(
    'yourpersonal@email.com', // CHANGE THIS to your own email
    subject,
    body,
    {
      bcc: recipients
    }
  );

  SpreadsheetApp.getUi().alert('✅ Draft created in Gmail! Check your Drafts folder.');
  sheet.getRange('C1').setValue(false); // Reset checkbox
}
````

3. Click **Save** (💾 icon).
4. Click **Run ▶️**. Accept the permissions and grant access when prompted.

   * It's normal to get a small error at the bottom after first run.

---

### 4. ⏰ Set the Trigger

1. In Apps Script, click the **Triggers (clock icon)** in the left sidebar.
2. Click **“+ Add Trigger”** (bottom-right corner).
3. Configure it like this:

   * **Function to run**: `onEdit`
   * **Deployment**: `Head`
   * **Event source**: `From spreadsheet`
   * **Event type**: `On edit`
4. Save the trigger.

---

## ✅ How to Use

1. Share the form and collect emails.
2. Open the linked Google Sheet.
3. **Check the box in C1** — this triggers the script.
4. A **Gmail draft** is created with all email addresses in **BCC**.
5. **Uncheck the box**, and you can **repeat** anytime with updated emails.

---

## 🔒 Notes

* The script creates a **draft** — not an actual send. You can manually review and send from your Gmail drafts folder.
* Change `'yourpersonal@email.com'` to your own address.
* If you'd like to send automatically, you can replace `GmailApp.createDraft()` with `GmailApp.sendEmail()`.

---

