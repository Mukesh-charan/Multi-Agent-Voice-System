
# 📞 Call Workflow Automation – README

This system is composed of **two automated workflows** integrated with **Vapi** and **Airtable**. It manages outbound call initiation and post-call processing.

---

## 📤 Workflow 1: Outbound Calling

### 📋 Description

This flow is triggered manually or by form submission (e.g., lead form or enquiry form). It prepares and sends a call request using Vapi and logs metadata in Airtable.

### 🧩 Workflow Nodes

| Index | Node Name       | Type          | Description |
|-------|------------------|---------------|-------------|
| 1     | **Webhook**      | Webhook       | Listens for incoming HTTP POST requests from the enquiry form. |
| 2     | **Airtable**     | Airtable      | Creates a new record with submitted form data. |
| 3     | **Airtable1**    | Airtable      | Searches for records with status = 'TBC' (To Be Called). |
| 4     | **Edit Fields**  | Set           | Prepares the retrieved data (e.g., name, mobile) for the API call. |
| 5     | **HTTP Request** | HTTP Request  | Sends POST request to Vapi to initiate the call. |
| 6     | **Airtable2**    | Airtable      | Updates record to reflect that the call was initiated. |

### 🔄 Sequence Flow

1. `Webhook` → 2. `Airtable` → 3. `Airtable1` → 4. `Edit Fields` → 5. `HTTP Request` → 6. `Airtable2`

---

## 📥 Workflow 2: Call Result Processing

### 📋 Description

This flow is triggered via Vapi’s webhook after the call ends. It records the outcome, checks if the call was answered or missed, and determines next steps such as retries or marking final status.

### 🧩 Workflow Nodes

| Index | Node Name                  | Type              | Description |
|-------|----------------------------|-------------------|-------------|
| 1     | **Webhook Node**           | Webhook           | Listens for HTTP POST request from Vapi with call data. |
| 2     | **Event = end-of-call-report** | IF            | Validates if the incoming data is an "end-of-call-report". |
| 3     | **Raw Data Node**          | Airtable Create   | Logs raw call data such as duration, cost, etc. |
| 4     | **Answered Call Node**     | IF                | Checks if the call was answered. |
| 5     | **Answered but Incomplete**| IF                | Checks if the call ended due to reasons like "customer-did-not-answer". |
| 6     | **Get ID #1**              | Airtable Search   | Retrieves record by customer phone number. |
| 7     | **Airtable Node**          | Airtable Update   | Updates lead status in Airtable. |
| 8     | **Raw Data2 Node**         | Airtable Create   | Logs extended call data including reason for termination. |
| 9     | **Didn’t Answer the Call** | IF                | Verifies if the call wasn’t answered. |
| 10    | **Get ID #2**              | Airtable Search   | Searches customer record again for missed call follow-up. |
| 11    | **First Call Node**        | IF                | Checks if this was the first call attempt. |
| 12    | **Attempt #2 Node**        | Airtable Update   | Updates record to indicate a second attempt will be made. |
| 13    | **1 Minute Wait**          | Wait              | Introduces a 1-minute delay before retry. |
| 14    | **Call #2 Node**           | HTTP POST         | Sends API request to initiate a second call. |
| 15    | **Attempt #2**             | Airtable Update   | Logs drop or second attempt failure. |
| 16    | **Get ID #3**              | Airtable Search   | Retrieves record for missed call scenario. |
| 17    | **Airtable1 Node**         | Airtable Update   | Updates record with missed call resolution. |
| 18    | **Airtable2 Node**         | Airtable Update   | Updates after failed first attempt (before retry). |
| 19    | **Get ID**                 | Airtable Search   | Fetches ID if call was answered. |
| 20    | **Airtable3 Node**         | Airtable Update   | Final update of lead status. |

### Scenario 1: Call Answered

1. Webhook receives report.

2. Event = "end-of-call-report"

3. Log data in Raw Data Node 

4. Check if Answered Call Node

5. Search record (Get ID)

6. Update call status (Airtable3 Node).


### Scenario 2: Answered but Incomplete

1. Follow steps 1–4.

2. Answered but Incomplete → TRUE

3. Search with Get ID #2.

4. Update record in Airtable1 Node.

### Scenario 3: Not Answered/Voicemail

Same starting flow → Answered Call Node → FALSE

1. **Didn’t Answer the Call → TRUE**
   - Search for the record using **Get ID #2**.

2. **Check First Call Node**:
   - **First Attempt**:
     - Update the attempt status in **Attempt #2 Node**.
     - Wait for 1 minute using the **1 Minute Wait Node**.
     - Trigger the second call using **Call #2 Node**.
     - Log the second attempt in the **Attempt #2 Node**.

   - **Second Attempt**:
     - Update the final call status in **Airtable2 Node**.

- The workflow is triggered when a call is not answered and proceeds to check if it’s the first attempt.
- If it is the first attempt, the workflow logs the data and waits before initiating a second call.
- Final status updates are logged in Airtable after both attempts.


---

## 🔁 Retry Logic

| Condition | Action |
|----------|--------|
| First attempt failed | Wait 1 min and retry |
| Retry fails | Mark as "Attempt #2" |
| Call answered later | Update Airtable with status |

---

## 📊 Data Logging & Tracking

- **All Airtable Logs**: Every call action is recorded for transparency.
- **Phone Number Matching**: Used to fetch the right lead.
- **Retry & Follow-up**: Handled seamlessly using Airtable + Vapi APIs.

---

💡 Notes

1. Phone number is the primary key used for all record searches.

2. All outcomes are recorded in Airtable to ensure a clear call trail.

3. You can expand this logic to support 3rd attempts or escalation workflows.

---
