**LINK TO :** OSINT

**TYPE:** OSINT

## Among Us at School

**Challenge:** Find the acronym of the school in the image.


**Solution:**
1. The building shows Vietnamese text: "TRƯỜNG ĐẠI HỌC CÔNG NGHỆ THÔNG TIN"
2. Translation: "University of Information Technology"
3. Vietnamese acronym: **UIT** (Đại học Công nghệ Thông tin)
4. **Flag:** `v1t{UIT}`

## WikiLeaks Iraq Equipment List

**Challenge:** In summer 2007, a massive archive surfaced - a 1,996 page equipment list from Operation Iraqi Freedom. Someone demanded its removal. Find the email address of the official who tried to bury the list.

**Solution:**
1. Searched for WikiLeaks Iraq military equipment leak from 2007
2. Found reference to "US Military Equipment in Iraq (2007)" - 1,996 pages CSV
3. Located archived correspondence showing takedown request
4. Identified sender: **David J. Hoskins**
5. **Flag:** `v1t{david.j.hoskins@us.army.mil}`

## Wooden Duck Halloween Decoration

**Challenge:** Find website selling wooden duck Halloween decoration.

**Solution:**
1. Searched for wooden duck with witch hat
2. Identified brand: DCUK (Duck Company UK)
3. **Flag:** `v1t{dcuk.com}`

## FSB Location

**Challenge:** Find location with FSB badges and "playing music" clue. Aerial image shows a compound in Russian forest.

**Solution:**
1. Identified FSB 100th anniversary badges (2017) from the medal images
2. Searched for "16 Center" (FSB 16th Center)
3. Located the badge unit facility at the coordinates
4. **Flag:** `v1t{55.592169,37.689097}`

**TYPE:** WEB

## Tiny Flag

**Challenge:** Do you see the tiny flag? Link: https://tommytheduck.github.io/tiny_flag

**Solution:**
1. Visited the webpage "Tiny Flag" by TommyTheDuck
2. The flag was hidden in the favicon image

##  Mark The Lyrics

**Challenge:** My friend make a website for his favourite, but the lyrics seem a little bit odd
http://tommytheduck.github.io/mckey

**Solution:**
1. Visted the website and saw the given lyrics and comapred it with actaul lyrics of the song and the different lyrics got the flag

##  5571

**Challenge:** My friend got highest mark with this challenge, can you beat it :>
http://chall.v1t.site:30300/

**Solution:**
1. Started checking with normal Javascript commands , got help from cgpt and started using  SSTI (Server-Side Template Injection)
2. First confirmed SSTI by math evaluation
3. Found out many keywords are banned
4. used %7B%7Bconfig%7D%7D to get the config payload and got to know its JINJA
5. Trying different commands got the flag.txt

##  Stylish Flag

**Challenge:** front end developer

**Solution:**
1. Flag had the hidden class which i removed.
2. Rotated the flag by 180deg
3. pushed flag to left = 0.

##  Login Panel

**Challenge:** login

**Solution:**
1. Got the username and password from the inspect.
2. They were in hash so i used https://hashes.com/en/decrypt/hash to get the flag.


**TYPE:** CRYPTO

## Lost Some binary

**Challenge:** SOS we lost some binary sir! Given a file with binary data.

**Solution:**
1. File contains binary data (8-bit groups)
2. Decoded the ASCII"
3. Real flag hidden in **LSB (Least Significant Bit)** steganography
4. Extracted the last bit of each byte, concatenated and decoded to ASCII
5. **Flag:** `v1t{LSB:>}`


**TYPE:** REV

## Extract the flag from the provided obs.py

**Solution:**
1. Inspected obs.py and found a runtime decoder.
2. Extracted / ran the decoder in a safe sandbox to recover its output.
3. Read the recovered output containing the flag.

**Flag**: v1t{d4ng_u_kn0w_pyth0n_d3bugg}

