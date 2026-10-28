# Reflected XSS into HTML context with most tags and attributes blocked

## Lab walkthrough

### Step 1: Test the initial injection

Inject a standard XSS vector into the search box:

```html
<img src=1 onerror=print()>
```

This request is blocked with an HTTP 400 Bad Request. In the following steps, we will use Burp Intruder to identify which tags and attributes are allowed.

### Step 2: Configure Burp Intruder for tag discovery

Open Burp's browser and perform a search in the lab.

Send the captured search request to Burp Intruder.

Replace the search term with `<>`.

![Screenshot of the search term set to `<>`](https://github.com/user-attachments/assets/fd2ad82c-0ffc-40b9-9c87-d7b7d9df6115)

Place the cursor between the angle brackets and click Add § to define a payload position:

```text
<§§>
```

Visit the PortSwigger XSS Cheat Sheet and click Copy tags to clipboard.

In Burp Intruder's Payloads side panel, under Payload configuration, click Paste to insert the list of tags.

### Step 3: Analyze the tag scan results

Click Start attack.

Once the attack completes, review the HTTP response codes.

Note that while most payloads return a 400 Bad Request, the `body` tag returns an HTTP 200 OK response.

![Tag scan results showing the `body` tag returning 200 OK](https://github.com/user-attachments/assets/a4bb08df-d8de-44f3-b416-a52c714fae56)

### Step 4: Prepare the parameter for attribute testing

Go back to Burp Intruder positions and set your search parameter to:

```text
<body%20=1>
```

Clear any existing payload positions or payloads in the configuration tab before defining the new attribute position.

### Step 5: Set the event handler payload position

Place the cursor before the `=` character and click Add § to set the new payload position:

```text
<body%20§§=1>
```

![Screenshot of the attribute payload position setup](https://github.com/user-attachments/assets/965091d5-efda-4334-bef1-387062ee253d)

### Step 6: Configure the event handler payloads

Return to the XSS Cheat Sheet and click Copy events to clipboard.

In Burp Intruder, under Payload configuration, click Clear to remove the previous tag list, then click Paste to populate the event handlers.

### Step 7: Analyze the event handler scan results

Click Start attack.

Review the results. Note that most event handlers trigger a 400 Bad Request, but the `onresize` payload receives a 200 OK status.

![Event handler scan results showing `onresize` returning 200 OK](https://github.com/user-attachments/assets/2d49715a-917b-4ecc-b1bb-10f25c9a4fa1)

### Step 8: Deliver the exploit via the Exploit Server

Navigate to the Exploit Server.

Paste the following payload into the Body section (replacing YOUR-LAB-ID with your actual lab ID):

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload="print()"></iframe>
```

Click Store, then click Deliver exploit to victim.

This payload abuses the allowed `body` element and the permitted `onresize` event handler to trigger JavaScript execution when the page loads.
