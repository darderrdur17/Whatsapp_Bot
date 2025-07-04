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
2. Connection: authorize with your Google account.  
   File: *Time Log MVP* • Sheet: *Employees*.
3. Condition: **Phone Number = `replace(From; "whatsapp:"; "")`** (removes the "whatsapp:" prefix from Twilio's phone number). Limit = 1.
   > **💡 Phone Number Matching:** Twilio sends phone numbers as `whatsapp:+6581234567`, but your sheet stores them as `6581234567`. The `replace()` function strips the `whatsapp:` prefix for proper matching.

### C. Router & Command Paths
Add a **Router** after the Search step; create three branches:

| Path | Filter Condition (`lower(trim(Body))`) |
|------|---------------------------------------|
| Check-in | `/checkin` |
| Check-out | `/checkout` |
| Default | *else* |

### D. Check-in Branch
1. **Google Sheets > Add a Row** (Timesheet sheet):
   - `Date`: `formatDate(now; "YYYY-MM-DD"; "Asia/Singapore")`
   - `Time`: `formatDate(now; "HH:mm:ss"; "Asia/Singapore")`
   - `Employee Name`: value from Search Rows
   - `Event Type`: `check-in`
2. **Twilio > Send a Message**:
   - From: your sandbox number (starts with `whatsapp:+1…`)
   - To: `From`
   - Body: `✅ You checked in at {{Time}}.`

### E. Check-out Branch
Exact same as above, but `Event Type = check-out` and tweak the message text.

### F. Default Branch (Invalid Command)
Single Twilio Send Message:  
`❓ Unknown command. Use /checkin or /checkout.`

### G. Unregistered Number Handling
If **Search Rows** returns **0**, add an error route:
```
🚫 You are not registered. Please contact HR.
```

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
| Duplicate-Check | Prevent multiple check-ins per day | Google Sheets > Search Timesheet before insert; if already exists, short-circuit with a polite "Already checked in today" reply. |
| Signature Validation | Verify Twilio really sent the webhook | Use Make's `hash_hmac()` to compare `X-Twilio-Signature`. |
| Daily Summary | Manager gets a recap | Scheduler module + aggregator + Twilio or Gmail send. |
| Shift Duration | Auto-calc hours worked | On checkout, look up latest unmatched check-in and `dateDiff()` the times. |

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

---

Made with ☕️, spreadsheets, and zero lines of code. Enjoy shipping! 🚀 