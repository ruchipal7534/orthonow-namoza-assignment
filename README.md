# OrthoNow - Digital Growth & Martech Assignment

This repository contains the complete Martech setup, Google Tag Manager (GTM) event schema, optimized landing page, and CRM integration architecture for OrthoNow's 9 orthopaedic clinics.

---

## Task 01 - GTM Event Schema

### 1. Full Event Tracking Schema
This table outlines the structured event tracking architecture required for OrthoNow before launching paid campaigns. It captures all key user interactions across the website, including the multi-step form, click-to-calls, WhatsApp widget, gated guide downloads, location views, and blog engagement.

| Event Name | Trigger Type | Key Parameters (Min. 3 per event) | GA4 Report / Audience Destination |
| :--- | :--- | :--- | :--- |
| `booking_step_complete` | Custom Event | 1. `step_number`<br>2. `step_name`<br>3. `clinic_location`<br>4. `specialty` | Funnel Exploration, Reports > Engagement > Events |
| `click_to_call` | Just Links (URL contains `tel:`) | 1. `click_text`<br>2. `page_location`<br>3. `clinic_name` (if clicked on a specific clinic page) | Reports > Engagement > Events, Retargeting Audiences |
| `whatsapp_widget_click` | Just Links (URL contains `wa.me`) | 1. `click_url`<br>2. `page_location`<br>3. `chat_placement` (e.g., 'floating_button') | Reports > Engagement > Events |
| `patient_guide_download` | Custom Event (On successful form submission) | 1. `guide_name`<br>2. `form_id`<br>3. `lead_type` (e.g., 'gated_pdf') | Reports > Engagement > Conversions, Lead Gen Audiences |
| `clinic_location_view` | Page View (Page URL contains `/clinics/`) | 1. `page_location`<br>2. `clinic_name`<br>3. `clinic_city` | Reports > Engagement > Pages and screens, Geo-targeted Audiences |
| `blog_scroll_depth` | Scroll Depth (Thresholds: 25%, 50%, 75%, 90%) | 1. `scroll_threshold`<br>2. `article_title`<br>3. `article_category` | Reports > Engagement > Events, Content Engagement Optimization |

---

### 2. 3-Step Booking Form Funnel & Drop-off Tracking

#### The Strategy
Google Tag Manager cannot natively listen to or understand multi-step form state changes without assistance. To track funnel drop-offs accurately, the front-end developer must explicitly push custom events to the `dataLayer` at the successful validation and completion of each step.

* **Step 1 Trigger:** Custom Event standardizing on `booking_step_complete` where the dataLayer variable `step_number` equals `1`.
* **Step 2 Trigger:** Custom Event standardizing on `booking_step_complete` where the dataLayer variable `step_number` equals `2`.
* **Step 3 Trigger:** Custom Event standardizing on `booking_step_complete` where the dataLayer variable `step_number` equals `3`.

#### DataLayer JSON Pushes

**Step 1: Clinic Location + Specialty Selected**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Pain / Arthroscopy"
}
