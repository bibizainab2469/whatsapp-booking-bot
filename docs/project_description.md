## **1. Project Overview** 

WF7 is an automated WhatsApp Booking Bot built on n8n, designed to handle customer interactions for a service-based business. The bot listens for incoming WhatsApp messages via a Twilio webhook, classifies the customer's intent, and responds intelligently — either providing available booking slots from Calendly, sharing pricing information, assisting with cancellations, or routing unrecognized messages to a human-friendly fallback. 

**Property Value** Workflow Name WF7 — WhatsApp Booking Bot Platform n8n (cloud/self-hosted) Messaging Channel WhatsApp via Twilio Sandbox Scheduling Integration Calendly API Intent Detection Rule-based Switcher (4 branches) Log Storage Excel/Google Sheets (WhatsApp Log) 

## **2. Project Objectives** 

- Automate responses to common customer queries on WhatsApp without human intervention. 

- Integrate Calendly to fetch and present real-time available booking slots to customers. 

- Provide pricing information instantly upon request. 

- Handle cancellation/rescheduling requests gracefully with a direct Calendly management link. 

- Route unknown or out-of-scope messages to a helpful fallback reply that directs the customer to 

- a human agent. 

• Log all interactions (timestamp, customer phone, message text, intent branch, message SID) for analysis. 

## **3. Tools & Technologies** 

Tool Role Notes n8n Workflow automation engine        Hosts and executes the entire workflow Twilio (WhatsApp API) Messaging gateway Sandbox number: +14155238886 Calendly API Slot fetching GET /event_type_available_times Webhook (n8n) Entry point Receives POST from Twilio on every message 

Edit Fields node         Data extraction Parses CustomerPhone, BusinessNumber, MessageText, messageSid Switch node Intent routing 4 branches: Booking, Pricing, Cancel, Fallback Code / JSON node      Slot formatting Formats slot times and builds Calendly booking links 

## **4. Workflow Architecture** 

The workflow follows a linear trigger-then-branch pattern: 

1. Webhook node receives an HTTP POST from Twilio when a WhatsApp message arrives. 

2. Respond to Webhook node immediately acknowledges the request (keeps Twilio happy). 

3. Edit Fields node extracts the key fields: CustomerPhone, BusinessNumber, MessageText, and messageSid. 

4. Intent detection Switcher evaluates the MessageText against keyword rules and routes to one of four branches. 

5. Each branch executes its dedicated response node(s) and sends a reply via Twilio SMS node. 

6. All executions are logged to an Excel sheet for auditing. 

## **4.1 Intent Detection Rules** 

Branch Trigger Keywords Action Booking book, slot, appointment, schedule Fetch Calendly slots, format, send options Pricing price, cost, how much, rate, fee Send static pricing message Cancel cancel, reschedule, change slot Send Calendly management link Fallback Anything else Send generic help message with contact info 

## **5. Task-by-Task Breakdown** 

## **Task 1 — Workflow Creation** 

A new n8n workflow was created and named WF7 — WhatsApp Booking Bot. This established the base canvas for all subsequent nodes. The workflow URL was noted for reference and submission. 

**Task 2 — Webhook Entry Point** 

A Webhook node was added as the workflow trigger. It listens for HTTP POST requests sent by Twilio whenever a customer sends a WhatsApp message to the sandbox number. A Respond to Webhook node was connected immediately after to send a 200 OK acknowledgment back to Twilio, preventing timeout errors. The successful execution passing 1 item was confirmed. 

## **Task 3 — Edit Fields (Data Extraction)** 

An Edit Fields node (set to manual mapping) was added after the webhook response. It extracts four fields from the incoming Twilio payload: 

- CustomerPhone — the sender's WhatsApp number 

- BusinessNumber — the Twilio sandbox number (e.g., whatsapp:+14155238886) 

- MessageText — the raw text of the customer's message 

- messageSid — Twilio's unique message identifier (e.g., 

- SM711a9e84608b2932f913c858b7602063) 

## **Task 4 — Intent Detection Switcher** 

A Switch node was configured in Rules mode and connected to the Edit Fields output. Four output branches were defined: Booking, Pricing, Cancel, and Fallback. Each branch uses a text-matching rule on the MessageText field to route execution to the appropriate response path. 

## **Task 5 — Booking Branch (Calendly Slot Fetching)** 

The Booking branch triggers a multi-step sub-flow: 

7. Fetch Calendly Slots: An HTTP Request node calls the Calendly API (GET 

/event_type_available_times) to retrieve the next available 30-minute slots. 

8. Format Slot Options: A Code/JSON node processes the API response, formats slot times into a human-readable numbered list, and generates individual Calendly booking links (slot1Link, slot2Link, slot3Link) for each available time. 

9. Send Booking Options: A Twilio SMS node sends the formatted message to the customer's WhatsApp, listing the 3 nearest slots with clickable Calendly booking links. 

Example output sent to customer: 

Hi! Here are the next available slots: 1. Thu, 25 Jun, 9:00 am 2. Thu, 25 Jun, 9:30 am 3. Thu, 25 Jun, 10:00 am Reply 1, 2, or 3 to confirm your booking — or tap a link to book directly. 

## **Task 6 — Booking Flow WhatsApp Test** 

The booking flow was tested end-to-end via the Twilio WhatsApp sandbox. The customer sent the keyword Book and received the formatted slot list with Calendly links. The test confirmed that the Calendly API integration, slot formatting, and WhatsApp delivery all worked correctly. 

## **Task 7 — Pricing Branch** 

The Pricing branch was built by adding a Send Pricing Reply node (Twilio SMS) connected to the Pricing output of the Switch node. When triggered by pricing-related keywords, the bot responds with a fixed pricing message: 

Hi! Here's our pricing: 🏆 30-min session — AED 150 🏆 60-min session — AED 250 🏆 90-min session — AED 350 Reply BOOK to schedule your appointment or ask us anything! 

Test confirmed via WhatsApp — the message How much it will cost triggered the correct pricing reply. 

## **Task 8 — Cancellation Branch** 

The Cancel branch routes to a Send Cancel Reply node. When the customer sends a message about cancelling or changing their slot, the bot responds with: 

No problem! You can cancel or reschedule your appointment here: htps://calendly.com/business Need help? Reply HELP and a team member will get back to you shortly. 

Test confirmed — the message Can I change my slot triggered the cancellation reply with the Calendly self-management link. 

## **Task 9 — Fallback Branch** 

The Fallback branch handles all unrecognized messages. A Send Fallback Reply node (Twilio SMS) was added to the Fallback output of the Switch. The fallback message gracefully informs the customer that automated handling is limited and directs them to a human agent or to use specific keywords: 

We're sorry to make you wait, but you can manage a human assistant to continue.  Please reply with your booking reference number and we'll take care of it right away.  Or call us directly at +971-XX-XXXXXXX for immediate assistance.  or for booking, price info or cancelling, use specific words.  Thanks. 

Test confirmed — the message Hello business triggered the fallback reply as expected. 

## **Task 10 — WhatsApp Interaction Log** 

All customer interactions were logged to an Excel file (WF7-Task010 WhatsApp Log.xlsx). The log captures the following fields for every message processed by the workflow: 

|Timestamp|CustomerPhone<br>MessageText<br>Branch MessageSID|CustomerPhone<br>MessageText<br>Branch MessageSID|CustomerPhone<br>MessageText<br>Branch MessageSID|
|---|---|---|---|
|2026-06-23|11:25|whatsapp:+9237...|how much it cost man<br>Pricing SM2d515...|
|2026-06-23|11:39|whatsapp:+9237...|I want to book a slot<br>Booking<br>SMe8552...|
|2026-06-23|11:39|whatsapp:+9237...|Whats the cost for it<br>Pricing SMc6b53...|
|2026-06-29|09:14|whatsapp:+9237...|Hello business Fallback<br>SMeeb36...|
|2026-06-29|09:21|whatsapp:+9237...|Book me a slot Booking<br>SMf1b88...|
|2026-06-29|09:21|whatsapp:+9237...|How much it would costPricing SM4bb43...|
|2026-06-29|09:22|whatsapp:+9237...|Is there any way to cancel this slot<br>Cancel|
|SMb3efa...||||
|2026-06-29|09:22|whatsapp:+9237...|Are you there<br>Fallback<br>SM646ae...|



The log demonstrates all four intent branches being triggered successfully during testing, confirming that the intent detection rules work correctly across a variety of natural-language phrasings. 

## **6. Final Workflow Summary** 

The completed workflow contains the following nodes in its final state: 

|#|Node Name<br>|Purpose|
|---|---|---|
|1|Webhook<br>|Entry point — receives Twilio POST|
|2|Respond to Webhook<br>|Sends 200 OK acknowledgment to Twilio|
|3|Edit Fields<br>|Extracts CustomerPhone, BusinessNumber, MessageText, messageSid|
|4|Intent detecton Switcher|Routes to Booking / Pricing / Cancel / Fallback|
|5|Fetch Calendly Slots<br>|Calls Calendly API for available tmes (Booking branch)|
|6|Format Slot Optons<br>|Formats slot list and builds Calendly links (Booking branch)|
|7|Send booking optons|Sends formated slots to customer via WhatsApp|
|8|Send Pricing Reply<br>|Sends pricing info to customer via WhatsApp|
|9|Send Cancel Reply<br>|Sends Calendly management link for cancellatons|
|10|Send Fallback Reply<br>|Sends human-agent redirect message for unknown queries|



## **7. Key Learnings & Challenges** 

• Twilio requires an immediate webhook response — the Respond to Webhook node must come before any processing to avoid Twilio timeout errors. 

- Calendly API requires an Authorization Bearer token and the event_type URI to be passed as 

- query parameters. 

- WhatsApp messages sent via Twilio sandbox require both sender and receiver to join the 

- sandbox first. 

• The Switch node in n8n uses the first matching rule only — ordering of branches matters for correct intent routing. 

• Logging interactions to a spreadsheet in real-time required careful field mapping in the workflow to capture all relevant data. 

## **8. Submission Checklist** 

- WF7-output.zip containing all task screenshots (Task01 through Task10) 

- WF7-Task010 WhatsApp Log.xlsx — interaction log file 

- This project description document 

- n8n json file 

**All tasks (01–10) have been completed and tested successfully.** 

