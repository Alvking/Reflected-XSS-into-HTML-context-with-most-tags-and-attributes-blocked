# Reflected-XSS-into-HTML-context-with-most-tags-and-attributes-blocked
Reflected XSS into HTML context with most tags and attributes blocked
Markdown
# Reflected XSS into HTML Context with Most Tags and Attributes Blocked

## Lab Walkthrough

### Step 1: Test Initial Injection
Inject a standard XSS vector into the search box:

```html
<img src=1 onerror=print()>
Observe that this request gets blocked (HTTP 400 Bad Request). In the following steps, we will use Burp Intruder to identify which tags and attributes are allowed.

Step 2: Configure Burp Intruder for Tag Discovery
Open Burp's browser and perform a search in the lab.

Send the captured search request to Burp Intruder.

Replace the search term with <>.
<img width="950" height="1081" alt="Screenshot_2026-10-05_19-10-04" src="https://github.com/user-attachments/assets/fd2ad82c-0ffc-40b9-9c87-d7b7d9df6115" />

Place the cursor between the angle brackets and click Add § to define a payload position: <§§>.

Visit the PortSwigger XSS Cheat Sheet and click Copy tags to clipboard.

In Burp Intruder's Payloads side panel under Payload configuration, click Paste to insert the list of tags.

Step 3: Analyze Tag Scan Results
Click Start attack.

Once the attack completes, review the HTTP response codes.

Note that while most payloads return a 400 Bad Request, the body tag returns an HTTP 200 OK response.
<img width="1749" height="960" alt="Screenshot_2026-10-05_19-10-25" src="https://github.com/user-attachments/assets/a4bb08df-d8de-44f3-b416-a52c714fae56" />

Step 4: Prepare Parameter for Attribute Testing
Go back to Burp Intruder positions and set your search parameter to:

Plaintext
<body%20=1>
Clear any existing payload positions or payloads in the configuration tab before defining the new attribute position.

Step 5: Set Event Handler Payload Position
Place the cursor before the = character and click Add § to set the new payload position:

Plaintext
<body%20§§=1>
<img width="950" height="1081" alt="Screenshot_2026-10-05_19-12-05" src="https://github.com/user-attachments/assets/965091d5-efda-4334-bef1-387062ee253d" />


Step 6: Configure Event Handler Payloads
Return to the XSS Cheat Sheet and click Copy events to clipboard.

In Burp Intruder, under Payload configuration, click Clear to remove the previous tag list, then click Paste to populate the event handlers.

Step 7: Analyze Event Handler Scan Results
Click Start attack.

Review the results. Note that most event handlers trigger a 400 Bad Request, but the onresize payload receives a 200 OK status.
<img width="1749" height="960" alt="Screenshot_2026-10-05_19-30-47" src="https://github.com/user-attachments/assets/2d49715a-917b-4ecc-b1bb-10f25c9a4fa1" />

Step 8: Deliver Exploit via Exploit Server
Navigate to the Exploit Server.

Paste the following payload into the Body section (replacing YOUR-LAB-ID with your actual lab ID):

HTML
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E](https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E)" onload=this.style.width='100px'></iframe>
Click Store, then click Deliver exploit to victim.
