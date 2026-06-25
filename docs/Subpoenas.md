# Subpoena Process
Here is a **complete, regulator‑aligned, FAA‑compliant process** for how **AI ARB** (aiarbitration.us) can accept, evaluate, generate, and return subpoenas requested via **subpoena@aiarbitration.us** or a **web form**, while staying strictly within the bounds of the **Federal Arbitration Act (FAA §7)** and your platform’s audit‑grade workflow.

This is written as a **process blueprint** you can directly implement in your clerk workstation and backend rulebook.

[A Model Federal Arbitration Summons](./A%20MODEL%20FEDERAL%20ARBITRATION%20SUMMONS.pdf)
---

## **1. Intake Channels**
AI ARB accepts subpoena requests through two controlled entry points:

### **A. Email Intake (subpoena@aiarbitration.us)**
- Only accepts **signed PDF** or **web‑form‑generated** requests.
- Auto‑reply acknowledges receipt and assigns a **Request ID**.
- Email must include:
  - Case ID
  - Requesting party (Claimant or Respondent)
  - Specific documents, testimony, or evidence sought
  - Rationale (relevance + necessity)
  - Whether the subpoena is for:
    - A party
    - A non‑party witness
    - A custodian of records

### **B. Web Form Intake**
- Requires login to aiarbitration.us.
- User must have already **accepted the arbitration agreement**.
- Form fields:
  - Case ID (validated)
  - Type of subpoena (documents, testimony, inspection)
  - Target recipient
  - Description of materials sought
  - Relevance justification
  - Upload supporting exhibits (optional)
- Form automatically generates a **draft request packet**.

---

## **2. Fee Handling (Only If Permissible Under Rulebook)**
FAA allows arbitrators to charge administrative fees, but **subpoena fees must be authorized in the arbitration agreement or rulebook**.

If your rulebook permits:

### **Fee Workflow**
- System calculates fee (e.g., $25–$75 administrative).
- User pays via Stripe/PayPal.
- Payment receipt is added to the chain‑of‑custody log.
- Subpoena request is not processed until payment clears.

If your rulebook does **not** authorize fees:
- The fee step is skipped entirely.

---

## **3. AI ARB Preliminary Review (Automated)**
The AI arbitrator performs a **procedural review**, not a merits review:

### **Checks**
- Is the request **within FAA §7 authority**?
- Is the request **specific**, not a fishing expedition?
- Is the target **within jurisdiction** (i.e., located in the U.S.)?
- Is the request **proportional** to the dispute?
- Is the subpoena **necessary** for resolving the claim?

### **Possible Outcomes**
1. **Approved** → moves to drafting.
2. **Needs Clarification** → system emails the requester with required edits.
3. **Denied** → system issues a written explanation citing rulebook provisions.

All outcomes are logged in the audit trail.

---

## **4. Subpoena Drafting (AI‑Generated, Arbitrator‑Issued)**
FAA §7 requires that **the subpoena be issued by the arbitrator**, not the forum.

Your system should generate:

### **Required Elements**
- Caption: *In the Matter of Arbitration Between…*
- Case ID
- Arbitrator name (AI ARB Arbitrator)
- Statement of authority under **FAA §7**
- Description of documents/testimony required
- Date, time, and method of compliance
- Instructions for objections or motions to quash
- Digital signature of the arbitrator (AI‑generated but cryptographically sealed)
- Chain‑of‑custody ID

### **Styling**
- Must be styled as:
  **“Subpoena Issued by Arbitrator Under FAA §7”**
- Not styled as a court subpoena.

---

## **5. Delivery to Requestor**
AI ARB **never** sends subpoenas directly to third parties.

Instead:

### **Delivery Workflow**
- Subpoena is delivered to the **requesting party only**.
- Requestor is responsible for:
  - Serving the subpoena
  - Paying witness fees (if applicable)
  - Filing proof of service

### **System Output**
- PDF subpoena
- Service instructions
- Chain‑of‑custody log
- Deadline calendar entry (optional)

---

## **6. Post‑Issuance Tracking**
AI ARB tracks:

- Service status (uploaded by requestor)
- Compliance status
- Objections or motions to quash
- Deadlines
- Production uploads

All evidence produced is stored in the **evidence vault** with immutable timestamps.

---

## **7. Enforcement (If Needed)**
Under FAA §7:

- Only a **federal district court** can enforce an arbitrator’s subpoena.
- AI ARB provides:
  - Certified copy of the subpoena
  - Certified copy of the arbitration agreement
  - Chain‑of‑custody logs
  - Arbitrator declaration

The requestor files the enforcement action, not AI ARB.

---

## **8. Audit‑Ready Logging**
Every step is logged:

- Request received
- Payment (if applicable)
- AI review
- Arbitrator issuance
- Subpoena generation
- Delivery to requestor
- Service proof
- Compliance or non‑compliance
- Evidence uploads

This satisfies your platform’s **chain‑of‑custody**, **procedural fairness**, and **regulatory‑grade auditability** requirements.

# API
Yes — **POST /subpoena** is exactly the right architectural move, and it fits cleanly into Clinfera’s existing pattern of:  
`/scan → /factfinding → /adjudicate`  

A dedicated **/subpoena** route lets you isolate FAA‑§7 logic, preserve chain‑of‑custody, and keep subpoena issuance fully deterministic and clerk‑free.

Below is the **correct, regulator‑aligned design** for that route.

---

# ✅ **Clinfera API Route: POST /subpoena**

**Purpose:**  
Accept a *formatted IRAC subpoena fact‑packet*, run FAA‑compliant analysis, and—if approved—auto‑generate and return a subpoena to the requestor **without clerk involvement**.

This keeps subpoena issuance procedural, neutral, and audit‑ready.

---

## **1. Request Format (IRAC‑style Subpoena Facts)**

Your POST body should be a structured fact object, similar to your adjudication facts:

```json
{
  "caseId": "C-2026-0041",
  "requestor": "claimant",
  "facts": [
    { "id": "F1", "source": "requestor", "text": "Requesting documents from non-party witness." },
    { "id": "F2", "source": "requestor", "text": "Documents relate to disputed billing entries." },
    { "id": "F3", "source": "requestor", "text": "Witness is located in the United States." }
  ],
  "issue": "Whether the arbitrator should issue a subpoena under FAA §7.",
  "rule": "FAA §7 allows arbitrators to summon witnesses and documents when necessary.",
  "analysis": "Request is specific, relevant, and proportional.",
  "conclusion": "Subpoena should be issued."
}
```

This gives the AI arbitrator everything needed to run a procedural review.

---

# 2. **Internal Logic of POST /subpoena**

The route performs **three sequential steps**:

---

## **Step A — Validate Request**
- Case exists  
- Requestor is a party  
- Arbitration agreement is active  
- Subpoena authority is permitted under FAA §7  
- Facts are well‑formed IRAC objects  

If validation fails → return `400 Bad Request`.

---

## **Step B — AI Arbitrator Review**
The AI arbitrator evaluates the IRAC packet and produces one of:

### **1. Approved**
Criteria:
- Specific documents/testimony  
- Clear relevance  
- Not a fishing expedition  
- Target is within U.S. jurisdiction  
- Proportional to dispute  
- Necessary for resolving the claim  

### **2. Needs Clarification**
Examples:
- Overbroad request  
- Missing relevance explanation  
- Target not identified  
- Time period unclear  

System returns:
```json
{
  "status": "needs_clarification",
  "message": "Please specify the date range for the requested documents."
}
```

### **3. Denied**
Examples:
- Non‑party request irrelevant  
- Burdensome  
- Outside FAA §7 authority  
- Attempt to compel pre‑hearing discovery (not allowed)  

System returns:
```json
{
  "status": "denied",
  "reason": "Request is overbroad and not tied to a specific issue in dispute."
}
```

---

# 3. **If Approved → Auto‑Generate Subpoena**

No clerk involvement.  
No manual review.  
Fully deterministic.

### **Subpoena Generation Includes:**
- FAA §7 caption  
- Arbitrator name (AI ARB Arbitrator)  
- Case ID  
- Description of materials/testimony required  
- Compliance deadline  
- Instructions for objections  
- Cryptographic signature  
- Chain‑of‑custody ID  

Output:

```json
{
  "status": "approved",
  "subpoenaPdf": "base64-encoded-pdf",
  "chainOfCustodyId": "COC-8821",
  "instructions": "Serve this subpoena on the witness and upload proof of service."
}
```

---

# 4. **Delivery to Requestor**
The system emails the requestor:

- PDF subpoena  
- Service instructions  
- Chain‑of‑custody log  
- Compliance deadline  

**AI ARB never serves the subpoena.**  
The requestor must serve it themselves.

---

# 5. **Audit Logging**
Every action is logged:

- Request received  
- AI analysis  
- Approval/denial  
- Subpoena generated  
- Subpoena delivered  
- Chain‑of‑custody entry  

This satisfies your PRESSPAGE‑grade audit requirements.

---

# 🔥 Why this design is correct
Because it:

- Keeps subpoena authority **with the arbitrator**, not the platform  
- Avoids clerks making legal decisions  
- Ensures **deterministic, rule‑based issuance**  
- Complies with **FAA §7**  
- Maintains **chain‑of‑custody integrity**  
- Allows fully automated subpoena generation  
- Keeps the requestor responsible for service (legally required)  
