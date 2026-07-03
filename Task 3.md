Task 3 – Integration Design

## Integration Architecture

The landing page form will submit the patient's Name, Phone Number, and Clinic Preference to a secure backend API. The backend will 
validate the data and first search HubSpot for an existing contact using the phone number. Since HubSpot's default deduplication works 
on email rather than phone, I would implement a custom lookup using the HubSpot CRM Search API before creating or updating the contact. 
If the phone number already exists, the existing contact will be updated; otherwise, a new contact will be created. 
Along with the patient details, the backend will also store Source as "Google Ads - Consultation Landing Page" and Lead Status as 
"New Enquiry".

After a successful HubSpot response, the backend will trigger the Karix WhatsApp Business API to send a confirmation message to the
patient. At the same time, the landing page will fire the Google Ads conversion event (`consultation_form_submitted`) using Google Tag 
Manager so that Google Ads can optimize campaigns based on completed consultation enquiries.

I prefer using the HubSpot CRM API with a backend service instead of Zapier or Make because it provides better control, improved security,
lower latency, and easier error handling for production systems.


## Biggest Failure Point

The biggest failure point is duplicate contact creation because HubSpot does not automatically deduplicate contacts based on phone
numbers. To prevent duplicates, every submission will first search HubSpot using the phone number before deciding whether to create
or update a contact. If the API fails temporarily, the request will be stored in a retry queue and processed automatically once the 
service is available.


## WhatsApp SLA Monitoring

The WhatsApp confirmation message must be delivered within two minutes. Possible failures include HubSpot API delays, backend server
issues, Karix API downtime, or network failures. To maintain the SLA, I would implement automatic retries with logging and monitoring. 
Every API request and response would be logged, and alerts would be generated if the WhatsApp message is not successfully delivered 
within two minutes. This ensures that failures are detected quickly and can be resolved before affecting patients.

