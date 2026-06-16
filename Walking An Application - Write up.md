
![[Pasted image 20260615083540.png]]
# Walking An Application - TryHackMe Write-Up 🚶‍♂️🌐

## Overview

In this room, we explore a web application from a pentester's perspective and learn how information can be unintentionally exposed through source code, hidden files, browser developer tools, and client-side storage.

The target application used throughout the room is **Acme IT Support**.

![[Pasted image 20260615083552.png]]

---

# Task 1 - Start the Machine

The first step is simply to deploy both the target machine and the AttackBox.

Once both systems are running, we can begin exploring the application and answering the room's questions.

---

# Task 2 - Exploring the Application

The objective of this task is to identify the endpoint used to create new support tickets.

After reviewing the provided Acme IT Support documentation and application overview, I identified the endpoint responsible for creating tickets.

✅ **Answer**

```text
/customers/ticket/new
```

![[Pasted image 20260615083609.png]]
---

# Task 3 - Viewing Page Source

Source code is often overlooked during web assessments, yet it can reveal valuable information such as hidden comments, internal paths, development notes, and references to additional resources.

After browsing to:

```text
http://MACHINE_IP
```

I opened the page source using:

- CTRL + U
    
- Right Click → View Page Source
    
- Or the `view-source:` URL prefix

	![[Pasted image 20260615083721.png]]
## Question 1 - HTML Comments

While reviewing the HTML source code, I discovered the following developer comment:

```html
This Page is temporary while we work on the new homepage
```

This immediately suggested that another version of the homepage might exist.

Further inspection revealed the endpoint:

```text
/new-home-beta
```

Browsing to that location revealed the first flag.

![[Pasted image 20260615083744.png]]

🚩 **Flag**
```text
THM{HTML_COMMENTS_ARE_DANGEROUS}
```

---

## Question 2 - Hidden Links

Continuing through the source code, I discovered a hidden hyperlink:

```html
href="/secret-page"
```

The page was not linked anywhere in the visible interface, but manually navigating to the endpoint exposed another flag.

![[Pasted image 20260615083828.png]]

🚩 **Flag**
```text
THM{NOT_A_SECRET_ANYMORE}
```

---

## Question 3 - Asset Enumeration

The source code also revealed that external resources such as JavaScript and CSS files were stored within:

```text
/assets
```

Since publicly accessible asset directories occasionally contain forgotten files, I decided to inspect the directory.

![[Pasted image 20260615084026.png]]

Inside, I discovered:
```text
flag.txt
```

Opening the file revealed the next flag.

🚩 **Flag**
```text
THM{INVALID_DIRECTORY_PERMISSIONS}
```

---

## Question 4 - Changelog Investigation

The final challenge hinted at finding the application's changelog.
![[Pasted image 20260615084206.png]]

While examining the source code, I noticed references to the framework used by the application and a URL leading to additional framework information.

After accessing the changelog page, I discovered an interesting endpoint:

```text
/tmp.zip
```

![[Pasted image 20260615084250.png]]

Files named _backup_, _archive_, or _tmp_ frequently contain forgotten development resources and should always be investigated.

Accessing the archive revealed the final flag for this task.

![[Pasted image 20260615084317.png]]

🚩 **Flag**
```text
THM{KEEP_YOUR_SOFTWARE_UPDATED}
```

![[Pasted image 20260615084348.png]]
---

# Task 4 - Developer Tools: Inspector

![[Pasted image 20260615084407.png|697]]

Modern browsers include powerful developer tools that can be used to inspect and manipulate webpage content.

For this task, we focus on the **Inspector** tab.

Inside the **News** section of the Acme IT Support application, the third article is hidden behind a premium subscription paywall.

![[Pasted image 20260615084442.png]]

Rather than attempting to bypass authentication, we can inspect the HTML responsible for displaying the overlay.

Using **Inspect Element**, I located the component:

```html
premium-customer-blocker
```

![[Pasted image 20260615084517.png]]

By modifying the element and replacing it with:

```html
Member
```

the paywall immediately disappeared, revealing the hidden content.

Remember that these modifications only exist locally within the browser and will disappear after refreshing the page.

![[Pasted image 20260615084550.png]]

🚩 **Flag**
```text
THM{NOT_SO_HIDDEN}
```


![[Pasted image 20260615084604.png]]
---

# Task 5 - Developer Tools: Debugger

The next task focuses on the **Debugger** tab.

After navigating to the **Contact** page, a brief red flash appeared every time the page loaded.

To understand what was happening, I inspected the application's JavaScript files and discovered:

![[Pasted image 20260615084638.png]]

```text
/assets/flash.min.js
```

The file was minified, making it difficult to read.

![[Pasted image 20260615084953.png]]

Using the **Pretty Print** feature (`{ }`) inside Developer Tools, I reformatted the script and searched for the functionality responsible for removing the red message.

![[Pasted image 20260615085030.png]]
Near the bottom of the file, I found:

```javascript
flash.remove();
```

This line was responsible for removing the red popup.

To prevent it from executing, I placed a breakpoint on the corresponding line and refreshed the page.

The script execution paused before the popup could be removed, allowing the hidden flag to remain visible.

![[Pasted image 20260615085053.png]]


🚩 **Flag**
```text
THM{CATCH_ME_IF_YOU_CAN}
```

---

# Task 5 - Developer Tools: Network

The second part of Task 5 introduces the **Network** tab.

The Network panel records every request made by a webpage, making it extremely useful when analyzing web applications.

While on the Contact page, I opened the Network tab and submitted the contact form.

A new request appeared:
![[Pasted image 20260615085231.png]]

```text
contact-msg
```

The task specifically asks for the flag located within the response.

After selecting the request and opening the **Response** tab, the flag became visible.

![[Pasted image 20260615085256.png]]

🚩 **Flag**
```text
THM{GOT_AJAX_FLAG}
```

![[Pasted image 20260615085312.png]]
---

# Task 6 - Developer Tools: Storage

The final task focuses on browser storage mechanisms.

After creating a new account at:

```text
/customers/signup
```

I logged in and opened the **Storage** tab within Developer Tools.

![[Pasted image 20260615085405.png]]

This section allows us to inspect:

- Local Storage
    
- Session Storage
    
- Cookies
    
- Cache Storage
    

For this task, the relevant information was stored within the session data.

The question asks:

> What is the value of the HttpOnly flag after logging in?

After reviewing the stored session information, I identified the value.

![[Pasted image 20260615085431.png]]

✅ **Answer**

```text
FALSE
```


![[Pasted image 20260615085450.png]]
---


![[Pasted image 20260615085506.png]]
# Conclusion 🎉

This room provides a great introduction to manual web application enumeration and browser-based reconnaissance techniques.

Throughout the challenge we learned how to:

- Analyze page source code
    
- Discover hidden endpoints
    
- Enumerate exposed directories
    
- Investigate changelogs and backup files
    
- Use the Inspector tool
    
- Work with breakpoints in the Debugger
    
- Capture requests with the Network tab
    
- Examine browser storage and session data
    

While the room focuses on beginner-friendly concepts, the techniques demonstrated here frequently appear during real-world web application assessments. See you :)