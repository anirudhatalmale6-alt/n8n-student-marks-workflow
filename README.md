# Student Marks Retrieval System - n8n Workflow

This n8n workflow allows parents to check their child's marks by submitting a simple form. The system looks up student records in Google Sheets, displays the marks on screen, and **automatically sends them via WhatsApp using Interakt**.

## Workflow Overview

```
Form Trigger → Google Sheets Lookup → IF (Student Found?)
                                          ├─ Yes → Format Response → Send WhatsApp (Interakt) → Display Marks
                                          └─ No → Display Error Message
```

## Features

- Built-in form with validation (Student Name, Student ID, Mobile Number)
- Google Sheets integration for marks lookup
- **WhatsApp notification via Interakt API** (automatic)
- Beautiful HTML response showing marks
- Error handling for non-existent students
- Automatic 200 OK response

---

## Setup Instructions

### Step 1: Import the Workflow

1. Open your n8n instance
2. Go to **Workflows** → **Add Workflow**
3. Click the **⋮** menu (three dots) → **Import from File**
4. Select `student-marks-workflow.json`
5. Click **Save**

### Step 2: Prepare Your Google Sheet

Create a Google Sheet with these columns (first row as headers):

| StudentID | StudentName | Mobile | Maths | Science | English | Total |
|-----------|-------------|--------|-------|---------|---------|-------|
| STU101    | Rahul Kumar | 9876543210 | 85 | 78 | 90 | 253 |
| STU102    | Priya Singh | 9876543211 | 92 | 88 | 85 | 265 |

**Important:**
- Column names must match exactly (case-sensitive)
- StudentID must be unique for each student
- The sheet name should be "Sheet1" (or update in workflow)

### Step 3: Connect Google Sheets

1. In the workflow, click the **"Lookup Student in Google Sheets"** node
2. Click on **Credential to connect with** → **Create New Credential**
3. Choose **Google Sheets OAuth2 API**
4. Follow the OAuth flow to connect your Google account
5. In the node settings:
   - **Document**: Select your Google Sheet or paste its ID
   - **Sheet**: Select "Sheet1" (or your sheet name)

**To get Sheet ID:** Open your sheet, the URL looks like:
`https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit`

### Step 4: Set Up Interakt WhatsApp Integration

#### 4.1 Get Your Interakt API Key

1. Log in to your Interakt account at https://app.interakt.ai
2. Go to **Settings** → **Developer Settings**
3. Copy your **API Key**

#### 4.2 Create the WhatsApp Template in Interakt

Before messages can be sent, you need to create and get approval for a WhatsApp template:

1. In Interakt, go to **Templates** → **Create Template**
2. Create a template with these details:
   - **Template Name**: `student_marks_report`
   - **Category**: Utility
   - **Language**: English (en)
   - **Body**:
   ```
   Student Marks Report

   Name: {{1}}
   ID: {{2}}

   Maths: {{3}}
   Science: {{4}}
   English: {{5}}

   Total: {{6}}

   Thank you for using our system!
   ```
3. Submit for WhatsApp approval (usually takes 24-48 hours)

#### 4.3 Configure n8n Credentials

1. In n8n, go to **Credentials** → **Add Credential**
2. Search for **HTTP Basic Auth**
3. Create new credential:
   - **Name**: `Interakt API`
   - **User**: Your Interakt API Key
   - **Password**: Leave empty (or enter the API key again)
4. Click **Save**

#### 4.4 Connect Credential to the Node

1. In the workflow, click the **"Send WhatsApp via Interakt"** node
2. Under **Credential to connect with**, select your `Interakt API` credential
3. Save the workflow

**Note:** If your country code is different from India (+91), edit the HTTP Request node and change the `countryCode` field.

### Step 5: Activate the Workflow

1. Click the **Save** button
2. Toggle the **Active** switch (top right) to ON
3. The form URL will appear in the Form Trigger node

### Step 6: Get Your Form URL

1. Click the **Form Trigger** node
2. Copy the **Form URL** (looks like: `https://your-n8n-url/form/student-marks-form`)
3. Share this URL with parents

---

## Testing the Workflow

### Method 1: Use the Built-in Form
1. Open the Form URL in a browser
2. Fill in:
   - Student Name: Rahul Kumar
   - Student ID: STU101
   - Mobile Number: 9876543210 (10 digits, no country code)
3. Click "Submit"
4. You should see the marks displayed on screen
5. Check WhatsApp - marks should be received!

### Method 2: Test in n8n
1. Click **Test Workflow** in n8n
2. Click **Test URL** button in the Form Trigger node
3. Fill the form and submit

---

## Expected Output

### On Screen (Success)
A beautiful HTML page showing:
- Student Name
- Student ID
- Individual subject marks (Maths, Science, English)
- Total marks
- Green banner: "Marks also sent to your WhatsApp!"

### On WhatsApp
The parent receives a WhatsApp message with:
```
Student Marks Report

Name: Rahul Kumar
ID: STU101

Maths: 85
Science: 78
English: 90

Total: 253

Thank you for using our system!
```

### Error Response (Student Not Found)
```
Student Not Found
No record found for Student ID: [entered ID]
Please verify the Student ID and try again.
```

---

## Interakt API Reference

**Endpoint:** `https://api.interakt.ai/v1/public/message/`

**Authentication:** HTTP Basic Auth (API Key as username)

**Request Body:**
```json
{
  "countryCode": "+91",
  "phoneNumber": "9876543210",
  "type": "Template",
  "template": {
    "name": "student_marks_report",
    "languageCode": "en",
    "bodyValues": ["Name", "ID", "Maths", "Science", "English", "Total"]
  }
}
```

**Rate Limits:**
- Growth Plan: 300 requests/minute
- Advanced Plan: 600 requests/minute

---

## Customization Guide

### Changing Country Code
Edit the **"Send WhatsApp via Interakt"** node and modify the `countryCode` field:
- India: `+91`
- USA: `+1`
- UK: `+44`
- etc.

### Adding More Subjects
1. Add columns to your Google Sheet (e.g., "Hindi", "History")
2. Edit the **"Format Marks Response"** Code node
3. Add new variables and update the message template
4. Update your Interakt template with additional placeholders
5. Update the `bodyValues` array in the HTTP Request node

### Changing Form Fields
1. Click the **Form Trigger** node
2. Modify fields under **Form Fields**
3. Add/remove fields as needed

### Using a Different Template Name
If your Interakt template has a different name:
1. Click the **"Send WhatsApp via Interakt"** node
2. Edit the JSON body
3. Change `"name": "student_marks_report"` to your template name

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "No data returned from Google Sheets" | Check Sheet ID and column names match exactly |
| Form not loading | Ensure workflow is activated |
| Student not found (but exists) | Verify StudentID matches exactly (case-sensitive) |
| Google Sheets connection error | Re-authenticate Google credentials |
| WhatsApp not received | Check Interakt API key, template approval status, and phone number format |
| 401 Unauthorized from Interakt | Verify API key is correct in credentials |
| 400 Bad Request from Interakt | Check template name matches exactly, verify bodyValues count matches template |

---

## Google Sheets Column Reference

| Column | Description | Required |
|--------|-------------|----------|
| StudentID | Unique identifier (e.g., STU101) | Yes |
| StudentName | Student's full name | Yes |
| Mobile | Parent's mobile number | No |
| Maths | Maths marks | Yes |
| Science | Science marks | Yes |
| English | English marks | Yes |
| Total | Total marks (can be auto-calculated) | Optional |

---

## Support

If you need help or modifications, feel free to reach out!
