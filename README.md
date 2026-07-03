# OrthoNow - Digital Growth & Martech Assignment
This repository contains the complete Martech setup, Google Tag Manager (GTM) event schema, optimized landing page, and CRM integration architecture for OrthoNow's 9 orthopaedic clinics.

---

## Task 01 - GTM Event Schema

### 1. Full Event Tracking Schema
This table outlines the structured event tracking architecture required for OrthoNow before launching paid campaigns.

| Event Name | Trigger Type | Key Parameters (Min. 3 per event) | GA4 Report / Audience Destination |
| :--- | :--- | :--- | :--- |
| booking_step_complete | Custom Event | 1. step_number, 2. step_name, 3. clinic_location, 4. specialty | Funnel Exploration, Reports > Engagement > Events |
| click_to_call | Just Links (URL contains 'tel:') | 1. click_text, 2. page_location, 3. clinic_name | Reports > Engagement > Events, Retargeting Audiences |
| whatsapp_widget_click | Just Links (URL contains 'wa.me') | 1. click_url, 2. page_location, 3. chat_placement | Reports > Engagement > Events |
| patient_guide_download | Custom Event (Form Submit) | 1. guide_name, 2. form_id, 3. lead_type | Reports > Engagement > Conversions, Lead Gen Audiences |
| clinic_location_view | Page View (URL contains '/clinics/') | 1. page_location, 2. clinic_name, 3. clinic_city | Reports > Engagement > Pages, Geo-targeted Audiences |
| blog_scroll_depth | Scroll Depth (25%, 50%, 75%, 90%) | 1. scroll_threshold, 2. article_title, 3. article_category | Reports > Engagement > Events |

---

### 2. 3-Step Booking Form Funnel & Drop-off Tracking

#### The Strategy
Google Tag Manager cannot natively listen to multi-step form state changes. To track funnel drop-offs accurately, the front-end JavaScript must explicitly push custom events to the dataLayer upon successful validation of each step.

* Step 1 Trigger: Custom Event 'booking_step_complete' where step_number === 1
* Step 2 Trigger: Custom Event 'booking_step_complete' where step_number === 2
* Step 3 Trigger: Custom Event 'booking_step_complete' where step_number === 3

#### DataLayer JSON Pushes

[Step 1: Clinic Location + Specialty Selected]
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Pain / Arthroscopy"
}

[Step 2: User Contact Details & Preferred Date Entered]
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Pain / Arthroscopy"
}

[Step 3: Final Booking Confirmation (Success State)]
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Pain / Arthroscopy",
  "booking_id": "BK-2026-9843"
}

---

## Task 03 - CRM & Integration Architecture Design

### 1. End-to-End Workflow Architecture
To connect the landing page with HubSpot CRM and Karix (WhatsApp Business API), we will use Make.com over Zapier for its superior multi-path routing, error-handling capabilities, and cost-efficiency.

* Step 1 (The Trigger): A lightweight custom Webhook hosted via Make.com captures the landing page data instantly when the form is submitted.
* Step 2 (The Routing & CRM Creation): The webhook routes data into HubSpot via the HubSpot Search & Upsert API.
* Step 3 (WhatsApp Automation): Once HubSpot successfully creates or updates the record, an API request is instantly forwarded to Karix WhatsApp Business API to dispatch a predefined verification template message.
* Step 4 (Google Ads Feedback): Concurrently, the client-side Google Ads conversion tag intercepts the consultation_form_submitted dataLayer event to optimize the bidding engine in real-time.

---

### 2. The HubSpot Deduplication Trap & Fallback Strategy
The Critical Trap: By default, HubSpot deduplicates contacts only by Email Address. Since Indian healthcare lead generation primarily utilizes a 2-field form collecting only Name and Phone Number, native integration will create duplicate contact records if a user registers multiple times.

#### The Solution:
Instead of utilizing standard form embedding, we will implement a custom JavaScript API execution via a Make.com scenario:
1. Search Phase: Search the HubSpot CRM database using the 'phone' property filter.
2. Conditional Router:
   * If Phone Exists: Update the existing Contact record, append a new note detailing the timestamp, update Clinic Preference, set Source to 'Google Ads - Consultation Landing Page', and move Lead Status back to 'New Enquiry'.
   * If Phone Does Not Exist: Execute a POST request to create a completely new Contact object with all fields defined.

#### Main Failure Point & Fallback:
The biggest failure point is API Rate Limiting or Webhook timeouts during high-volume ad traffic.
* The Fallback: We will script a secondary client-side fail-safe that writes submissions to a local browser localStorage buffer cache. If the primary webhook fails to respond with a 200 OK status within 4 seconds, a fallback script saves the record and schedules an asynchronous sync payload retry, ensuring zero lead leakage.

---

### 3. WhatsApp SLA Monitoring & Breach Mitigation
The 2-minute Service Level Agreement (SLA) for sending the WhatsApp message via Karix can break due to two reasons: downstream Karix API degradation or message queue stagnation.

#### Monitoring and Resolution:
* Real-time Alerting: We will configure an Error-Catching Routine inside Make.com. If the Karix API throws an error code (e.g., 4xx or 5xx) or delays its response past 15 seconds, a webhook automatically fires a critical priority alert to the internal Slack operations channel via PagerDuty.
* Automated Queue Retry: The failed payloads will instantly route to an Amazon SQS (Simple Queue Service) Dead-Letter Queue (DLQ). This queue retries the message push automatically up to 3 times with exponential backoff, ensuring the 2-minute SLA is protected even during server overloads.
