# CTF Writeups - v1t CTF

**Profile:** [https://ctf.v1t.site/teams/1170](https://ctf.v1t.site/teams/1170)

---

## OSINT Challenges

### Among Us at School

**Objective:** Identify the school acronym from an image containing Vietnamese text.

**Approach:**
1. Located Vietnamese text on building: "TRƯỜNG ĐẠI HỌC CÔNG NGHỆ THÔNG TIN"
2. Translated to English: "University of Information Technology"
3. Determined Vietnamese acronym: **UIT** (Đại học Công nghệ Thông tin)

<img width="639" height="362" alt="amongus" src="https://github.com/user-attachments/assets/2f7b5054-d451-4be4-aaf0-a29cd873fae4" />

**Flag:** `v1t{UIT}`

---

### WikiLeaks Iraq Equipment List

**Objective:** Locate the email address of the official who requested removal of a 1,996-page equipment list from Operation Iraqi Freedom that surfaced in summer 2007.

**Approach:**
1. Researched WikiLeaks Iraq military equipment leak (2007)
2. Found document reference: "US Military Equipment in Iraq (2007)" - 1,996 pages CSV format
3. Discovered archived correspondence containing takedown request
4. Identified requesting official: **David J. Hoskins**

**Flag:** `v1t{david.j.hoskins@us.army.mil}`

---

### Wooden Duck Halloween Decoration

**Objective:** Find the website selling a specific wooden duck Halloween decoration.

**Approach:**
1. Searched for wooden duck with witch hat decoration
2. Identified manufacturer: DCUK (Duck Company UK)

   <img width="666" height="666" alt="duck_company" src="https://github.com/user-attachments/assets/7a102aa2-1f3f-49fd-aa94-383953eeb336" />

**Flag:** `v1t{dcuk.com}`

---

### FSB Location

**Objective:** Determine coordinates of FSB facility using badges and aerial imagery showing a compound in Russian forest, with "playing music" as an additional clue.

**Approach:**
1. Analyzed FSB 100th anniversary badges (2017) from medal images
2. Researched "16 Center" (FSB 16th Center)
3. Checked the unit number in the pdf
4. coordinates were given in the pdf
   
<img width="624" height="654" alt="place" src="https://github.com/user-attachments/assets/761460f2-72aa-463d-ba43-3661c32dee29" />

**Flag:** `v1t{55.592169,37.689097}`

---

## Web Challenges

### Tiny Flag

**Challenge URL:** [https://tommytheduck.github.io/tiny_flag](https://tommytheduck.github.io/tiny_flag)

**Objective:** Locate a hidden flag on the webpage.

**Approach:**
1. Inspected page elements and discovered flag embedded in favicon image

**Flag:** Found in favicon

---

### Mark The Lyrics

**Challenge URL:** [http://tommytheduck.github.io/mckey](http://tommytheduck.github.io/mckey)

**Objective:** A friend created a website featuring their favorite song, but the lyrics appear modified.

**Approach:**
1. Visited website and examined displayed lyrics
2. Compared with authentic song lyrics
3. Identified discrepancies between versions
4. Extracted flag from altered lyrics

**Flag:** Obtained from lyric differences

---

### 5571

**Challenge URL:** [http://chall.v1t.site:30300/](http://chall.v1t.site:30300/)

**Objective:** A friend achieved the highest score on this challenge. Can you surpass it?

**Approach:**
1. Tested standard JavaScript commands initially
2. Identified potential for SSTI (Server-Side Template Injection)
3. Confirmed SSTI vulnerability through mathematical expression evaluation
4. Discovered multiple keyword blacklist restrictions
5. Used `%7B%7Bconfig%7D%7D` payload to retrieve configuration
6. Identified template engine as Jinja2
7. Iterated through various commands to locate and retrieve `flag.txt`

**Flag:** Retrieved from flag.txt file

---

### Stylish Flag

**Objective:** Front-end development challenge with hidden flag.

**Approach:**
1. Removed `hidden` class from flag element
2. Applied 180-degree rotation transformation
3. Adjusted positioning: `left: 0`

**Flag:** Revealed through CSS manipulation

---

### Login Panel

**Objective:** Authenticate to access protected content.

**Approach:**
1. Extracted username and password from page source via browser inspector
2. Identified credentials stored as hash values
3. Utilized [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash) to decrypt hashes

**Flag:** Obtained after successful decryption

---

## Cryptography Challenges

### Lost Some Binary

**Objective:** Recover missing binary data from provided file.

**Approach:**
1. Examined file containing 8-bit binary groups
2. Initially decoded as ASCII text
3. Recognized LSB (Least Significant Bit) steganography technique
4. Extracted final bit from each byte
5. Concatenated extracted bits and decoded to ASCII

**Flag:** `v1t{LSB:>}`

---

## 🔧 Reverse Engineering Challenges

### Extract Flag from obs.py

**Objective:** Retrieve flag from obfuscated Python script.

**Approach:**
1. Analyzed `obs.py` and identified runtime decoder mechanism
2. Executed decoder in isolated sandbox environment
3. Captured decoded output containing flag

**Flag:** `v1t{d4ng_u_kn0w_pyth0n_d3bugg}`

---



