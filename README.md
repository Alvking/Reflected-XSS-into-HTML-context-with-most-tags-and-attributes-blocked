# Reflected XSS into HTML context with most tags and attributes blocked

This lab demonstrates how to bypass a filter that blocks most HTML tags and attributes in a reflected XSS challenge.

## Lab walkthrough

### Step 1: Test the initial injection

Inject a standard XSS payload into the search box:

```html
<img src=1 onerror=print()>
```

The request is rejected with an HTTP 400 Bad Request, which confirms that the application is blocking common XSS vectors.

To determine which tags and attributes are permitted, we use Burp Intruder to brute-force the allowed input.

### Step 2: Configure Burp Intruder for tag discovery

Open Burp's browser and perform a search in the lab.

Send the captured request to Burp Intruder.

Replace the search term with `<>`.

![Search parameter set to `<>`](https://github.com/user-attachments/assets/fd2ad82c-0ffc-40b9-9c87-d7b7d9df6115)

Place the cursor between the angle brackets and click Add § to define the payload position:

```text
<§§>
```

Visit the PortSwigger XSS cheat sheet and click Copy tags to clipboard.

In Burp Intruder, under Payload configuration, click Paste to insert the tag list.

### Step 3: Analyze the tag scan results

Click Start attack and wait for the results.

After the attack completes, review the HTTP response codes.

Most payloads return HTTP 400 Bad Request, but the `body` tag returns HTTP 200 OK. This indicates that the application allows the `body` element in this HTML context.

![Tag scan results showing `body` returning HTTP 200](https://github.com/user-attachments/assets/a4bb08df-d8de-44f3-b416-a52c714fae56)

### Step 4: Prepare the parameter for attribute testing

Return to Burp Intruder and set the search parameter to:

```text
<body%20=1>
```

Clear any existing payload positions or payloads before defining the new attribute position.

### Step 5: Set the event handler payload position

Place the cursor before the `=` character and click Add § to define the new payload position:

```text
<body%20§§=1>
```

![Attribute payload position setup](https://github.com/user-attachments/assets/965091d5-efda-4334-bef1-387062ee253d)

### Step 6: Configure the event handler payloads

Go back to the PortSwigger XSS cheat sheet and click Copy events to clipboard.

In Burp Intruder, under Payload configuration, click Clear to remove the previous tag list, then click Paste to load the event handlers.

### Step 7: Analyze the event handler scan results

Click Start attack.

Review the results. Most event handlers trigger HTTP 400 Bad Request, but the `onresize` payload returns HTTP 200 OK.

This is the critical finding: the challenge permits the `body` element and the `onresize` event handler.

![Event handler scan results showing `onresize` returning HTTP 200](https://github.com/user-attachments/assets/2d49715a-917b-4ecc-b1bb-10f25c9a4fa1)

### Step 8: Deliver the exploit via the Exploit Server

Navigate to the Exploit Server.

Paste the following payload into the Body section, replacing `YOUR-LAB-ID` with your actual lab ID:

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload="print()"></iframe>
```

Click Store, then click Deliver exploit to victim.

When the victim loads the page, the allowed `body` tag and `onresize` event fire the JavaScript payload, resulting in reflected XSS execution.

## Why this works

The filter is not blocking every HTML tag or every attribute. By enumerating the allowed values with Burp Intruder, we identify the exact browser features that are still accepted by the application. Once `body` and `onresize` are confirmed as valid, a payload can be crafted that triggers execution in the vulnerable HTML context.
