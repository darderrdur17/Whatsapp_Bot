# 📱 WhatsApp Time-Logging Bot (No-Code MVP)

Save employee check-ins and check-outs straight to Google Sheets using only **Twilio**, **Make (Integromat)**, and a little spreadsheet magic—no traditional coding required.

---

## ☕️ At a Glance
1. Google Sheet holds employee records & time logs.
2. Twilio's WhatsApp Sandbox receives messages.
3. Make catches those messages via a webhook, looks up the employee, writes a row, and fires back a confirmation.

That's the entire Minimum Viable Product; everything else is optional polish. 👌

---

## 🔧 What You'll Need
| Service | Free Tier | Purpose |
|---------|-----------|---------|
| Google Sheets | ✅ | Data storage (Employees, Timesheet) |
| Twilio | ✅ (Sandbox) | WhatsApp messages |
| Make (Integromat) | ✅ | Drag-and-drop automation |

> **Tip :** Use the same Google account for Sheets and Make; it speeds up OAuth prompts later.

---

## 1️⃣  Create Your Google Sheet
1. New spreadsheet → name it e.g. **`Time Log MVP`**.
2. **Employees** tab  
   `A1 = Phone Number` `B1 = Employee Name`  
   Example:
   ```
   6581234567 | Alice Tan
   6589876543 | Bob Lim
   ```
   > **⚠️ Important:** Google Sheets automatically removes the `+` from phone numbers. Enter numbers WITHOUT the `+` prefix (e.g., `6581234567` not `+6581234567`).
3. **Timesheet** tab  
   `A1 = Date` • `B1 = Time` • `C1 = Employee Name` • `D1 = Event Type`
4. Share the sheet with **edit** permission to the Google account you'll connect in Make.

---

## 2️⃣  Fire Up the Twilio WhatsApp Sandbox
1. In Twilio Console: **Messaging → Try it out → Send a WhatsApp message**.
2. Follow the on-screen *join* instructions (e.g. text `join able-warm` to the sandbox number). You'll get an "all set" reply.
3. Keep the sandbox settings page open—we'll paste a webhook URL here soon.

---

## 3️⃣  Build Your Scenario in Make
> **Zero code, promise!** You'll drag modules, pick functions from menus, and map fields.

### A. New Scenario & Webhook
1. **Create new scenario → Webhooks > Custom Webhook**.
2. Name it **Twilio Incoming**, click *Save*. Copy the generated URL (looks like `https://hook.make.com/abc123`).
3. Paste that URL into Twilio Sandbox under **WHEN A MESSAGE COMES IN** → *Save*.
4. Hit **Run once** in Make. Send *anything* (e.g. "ping") to your sandbox number; Make captures a sample payload and your webhook is initialized.

### B. Google Sheets → Search Rows
1. Add **Google Sheets > Search Rows**.
2. **Connection:** authorize with your Google account.
3. **Spreadsheet ID:** Select your *Time Log MVP* spreadsheet.
4. **Sheet Name:** Select *Employees*.
5. **Table contains headers:** ✅ Check this box.
6. **Column range:** Leave as default (A:B) to search both Phone Number and Employee Name columns.
7. **Filter:** Click the filter icon and add:
   - **Field:** `Phone Number` (Column A)
   - **Condition:** `Equal to`
   - **Value:** `{{replace(1.From; "whatsapp:"; "")}}` (this removes the "whatsapp:" prefix from Twilio's phone number)
8. **Limit:** Set to `1`.
   > **💡 Phone Number Matching:** Twilio sends phone numbers as `whatsapp:+6581234567`, but your sheet stores them as `6581234567`. The `replace()` function strips the `whatsapp:` prefix for proper matching.

### C. Router & Command Paths
Add a **Router** after the Search step; create three branches:

| Path | Filter Condition |
|------|------------------|
| Check-in | `{{lower(trim(1.Body))}}` equals `/checkin` |
| Check-out | `{{lower(trim(1.Body))}}` equals `/checkout` |
| Default | *else* (no filter condition) |

> **💡 Router Logic:** The `lower(trim(1.Body))` function converts the message to lowercase and removes extra spaces, then compares it exactly to `/checkin` or `/checkout`. Only one branch should execute based on the exact command received.

### D. Check-in Branch
1. **Google Sheets > Add a Row** (Timesheet sheet):
   - **Connection:** Use the same Google account connection
   - **Spreadsheet ID:** Select your *Time Log MVP* spreadsheet
   - **Sheet Name:** Select *Timesheet*
   - **Table contains headers:** ✅ Check this box
   - **Values mapping:**
     - **Date (A):** `{{formatDate(now; "YYYY-MM-DD"; "Asia/Singapore")}}`
     - **Time (B):** `{{formatDate(now; "HH:mm:ss"; "Asia/Singapore")}}`
     - **Employee Name (C):** `{{2.0.Employee Name}}` (from Search Rows step - the `0` refers to the first result)
     - **Event Type (D):** `check-in`
   - **Unformatted:** No (leave unchecked)
   - **Value input option:** Leave as default
   - **Insert data option:** Leave as default
2. **Twilio > Send a Message**:
   - **Connection:** Authorize with your Twilio account
   - **Send a message from:** Your sandbox number (starts with `whatsapp:+1…`)
   - **To:** `{{1.From}}` (the phone number that sent the message)
   - **Message Body:** `✅ You checked in at {{formatDate(now; "HH:mm:ss"; "Asia/Singapore")}}.`
   - **Media URL:** Leave empty
   - **Smart encoded:** No (leave unchecked)
   - **Validity period:** Leave as default
   - **Status callback:** Leave empty
   - **Application:** Leave empty
   - **Max price:** Leave empty
   - **Provide feedback:** No (leave unchecked)

### E. Check-out Branch
1. **Google Sheets > Add a Row** (Timesheet sheet):
   - **Connection:** Use the same Google account connection
   - **Spreadsheet ID:** Select your *Time Log MVP* spreadsheet
   - **Sheet Name:** Select *Timesheet*
   - **Table contains headers:** ✅ Check this box
   - **Values mapping:**
     - **Date (A):** `{{formatDate(now; "YYYY-MM-DD"; "Asia/Singapore")}}`
     - **Time (B):** `{{formatDate(now; "HH:mm:ss"; "Asia/Singapore")}}`
     - **Employee Name (C):** `{{2.0.Employee Name}}` (from Search Rows step - the `0` refers to the first result)
     - **Event Type (D):** `check-out`
   - **Unformatted:** No (leave unchecked)
   - **Value input option:** Leave as default
   - **Insert data option:** Leave as default
2. **Twilio > Send a Message**:
   - **Connection:** Use the same Twilio connection
   - **Send a message from:** Your sandbox number (starts with `whatsapp:+1…`)
   - **To:** `{{1.From}}` (the phone number that sent the message)
   - **Message Body:** `✅ You checked out at {{formatDate(now; "HH:mm:ss"; "Asia/Singapore")}}.`
   - **Media URL:** Leave empty
   - **Smart encoded:** No (leave unchecked)
   - **Validity period:** Leave as default
   - **Status callback:** Leave empty
   - **Application:** Leave empty
   - **Max price:** Leave empty
   - **Provide feedback:** No (leave unchecked)

### F. Default Branch (Invalid Command)
**Twilio > Send a Message**:
- **Connection:** Use the same Twilio connection
- **Send a message from:** Your sandbox number (starts with `whatsapp:+1…`)
- **To:** `{{1.From}}` (the phone number that sent the message)
- **Message Body:** `❓ Unknown command. Use /checkin or /checkout.`
- **Media URL:** Leave empty
- **Smart encoded:** No (leave unchecked)
- **Validity period:** Leave as default
- **Status callback:** Leave empty
- **Application:** Leave empty
- **Max price:** Leave empty
- **Provide feedback:** No (leave unchecked)

### G. Unregistered Number Handling
If **Search Rows** returns **0**, add an error route with **Twilio > Send a Message**:
- **Connection:** Use the same Twilio connection
- **Send a message from:** Your sandbox number (starts with `whatsapp:+1…`)
- **To:** `{{1.From}}` (the phone number that sent the message)
- **Message Body:** `🚫 You are not registered. Please contact HR.`
- **Media URL:** Leave empty
- **Smart encoded:** No (leave unchecked)
- **Validity period:** Leave as default
- **Status callback:** Leave empty
- **Application:** Leave empty
- **Max price:** Leave empty
- **Provide feedback:** No (leave unchecked)

### H. Save & Activate!
Toggle the scenario ON. 🎉

---

## 4️⃣  Test the Flow End-to-End
1. Put your own WhatsApp number in *Employees* (international format without `+`, e.g., `6581234567`).
2. Send `/checkin` to the sandbox number.
   - Expect an instant ✅ reply.
   - Confirm a new row in *Timesheet*.
3. Send `/checkout` → second row + confirmation.
4. Try a random message ("hello") → see the ❓ invalid-command reply.
5. Remove your number from *Employees* and repeat → get the 🚫 unregistered message.

---

## 5️⃣  Optional Upgrades (Nice-to-Have)
| Feature | Why | How (Still No-Code) |
|---------|-----|---------------------|
| **WhatsApp Group Support** | Track team check-ins in a group chat | Add a filter to check if message is from a group, then extract sender info differently |
| Duplicate-Check | Prevent multiple check-ins per day | Google Sheets > Search Timesheet before insert; if already exists, short-circuit with a polite "Already checked in today" reply. |
| Signature Validation | Verify Twilio really sent the webhook | Use Make's `hash_hmac()` to compare `X-Twilio-Signature`. |
| Daily Summary | Manager gets a recap | Scheduler module + aggregator + Twilio or Gmail send. |
| Shift Duration | Auto-calc hours worked | On checkout, look up latest unmatched check-in and `dateDiff()` the times. |

### 📱 WhatsApp Group Implementation
**Why Groups?** Better team coordination, everyone can see who's checked in/out, and managers can monitor in real-time.

**How to Set Up:**
1. **Create a WhatsApp group** with your team members
2. **Add your Twilio sandbox number** to the group
3. **Modify the webhook logic** to handle group messages:
   - **Check if group:** `{{1.NumMedia}}` or `{{1.GroupId}}` (if available)
   - **Extract sender:** Use `{{1.Author}}` or `{{1.From}}` depending on group vs individual
   - **Add group info:** Store group name/ID in your timesheet for better tracking

**Enhanced Timesheet Structure:**
```
Date | Time | Employee Name | Event Type | Group Name | Phone Number
```

---

## 6️⃣  Going to Production
1. Apply for a WhatsApp Business number in Twilio (can take a few days).
2. Update the *From* number in Make + migrate the webhook.
3. Consider moving data from Sheets to a real DB if headcount grows.

---

## ❓ FAQ
**Do I need to code?** Nope! All logic lives inside Make and its built-in functions.  
**Is this secure enough?** Great for an MVP. For production, add signature checks and tighter permissions.  
**Can I export the data?** It's a Google Sheet—download CSV anytime or connect another Make scenario.

## 🔧 Troubleshooting

### Phone Number Issues
- **"You are not registered" error:** Check that the phone number in your Google Sheet matches exactly what Twilio sends (without the `+` prefix)
- **Google Sheets removes the `+`:** This is normal! Always enter phone numbers without the `+` in your sheet
- **Test your number format:** In Make's webhook test, check the `From` field to see exactly how Twilio formats your number

### Employee Name Not Saving
- **Check the reference:** In the Add a Row module, try these variations for Employee Name (C):
  - `{{2.0.Employee Name}}` (most common)
  - `{{2[0].Employee Name}}`
  - `{{2.Employee Name[0]}}`
  - `{{2.0.B}}` (if using column reference)
- **Debug the Search Rows output:** Add a "Set up a filter" step after Search Rows to see what data is actually returned
- **Verify column headers:** Make sure your Employees sheet has exactly "Employee Name" as the header (case sensitive)

---

Made with ☕️, spreadsheets, and zero lines of code. Enjoy shipping! 🚀 