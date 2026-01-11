# Student Marks Retrieval System - n8n Workflow

This n8n workflow allows parents to check their child's marks by submitting a simple form. The system looks up student records in Google Sheets and returns the marks.

## Workflow Overview

```
Form Trigger → Google Sheets Lookup → IF (Student Found?)
                                          ├─ Yes → Format Response → Display Marks
                                          └─ No → Display Error Message
```

## Features

- Built-in form with validation (Student Name, Student ID, Mobile Number)
- Google Sheets integration for marks lookup
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

### Step 4: Activate the Workflow

1. Click the **Save** button
2. Toggle the **Active** switch (top right) to ON
3. The form URL will appear in the Form Trigger node

### Step 5: Get Your Form URL

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
   - Mobile Number: 9876543210
3. Click "Get Marks"
4. You should see the marks displayed

### Method 2: Test in n8n
1. Click **Test Workflow** in n8n
2. Click **Test URL** button in the Form Trigger node
3. Fill the form and submit

---

## Expected Output

### Success Response (Student Found)
A beautiful HTML page showing:
- Student Name
- Student ID
- Individual subject marks (Maths, Science, English)
- Total marks

### Error Response (Student Not Found)
```
Student Not Found
No record found for Student ID: [entered ID]
Please verify the Student ID and try again.
```

---

## Customization Guide

### Adding More Subjects
1. Add columns to your Google Sheet (e.g., "Hindi", "History")
2. Edit the **"Format Marks Response"** Code node
3. Add new variables and update the message template

### Changing Form Fields
1. Click the **Form Trigger** node
2. Modify fields under **Form Fields**
3. Add/remove fields as needed

### Adding Email Notification (Optional)
1. Add an **Email** node after "Format Marks Response"
2. Connect your email credentials
3. Use `{{ $json.formattedMessage }}` in the email body

### Adding WhatsApp Notification (Optional)
1. Add a **HTTP Request** node
2. Configure with WhatsApp Business API
3. Use the formatted message as the payload

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "No data returned from Google Sheets" | Check Sheet ID and column names match exactly |
| Form not loading | Ensure workflow is activated |
| Student not found (but exists) | Verify StudentID matches exactly (case-sensitive) |
| Google Sheets connection error | Re-authenticate Google credentials |

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
