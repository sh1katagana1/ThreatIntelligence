# Advisory - Voidstealer ABE Bypass

***

## Summary
Researchers at Gendigital put out an article talking about how Voidstealer has a new bypass for the Chromium Application-Bound Encryption. This bypass would give access to a lot of the secrets stored in a Chrome or Edge browser. While the article is quite technical, I try to break it down in digestable bites.

## What is Application-Bound Encryption (ABE)?
In the past, Windows and macOS primarily used User-Based Encryption. If you were logged into your computer as "leroy," any program running under your account could ask the system to decrypt your Chrome passwords. If you accidentally downloaded a malicious file, that file was also running as "leroy." To the computer, the virus and Google Chrome looked identical. The system would hand over the passwords to the virus because it thought, "Well, leroy is asking for them".

Google realized that being "leroy" wasn't enough proof. They introduced ABE to verify the identity of the App. You would essentially need two things now to unlock your passwords:
1. The User Key: You must be logged in as "leroy"
2. The App Fingerprint: The request must come from the official Google Chrome application.

ABE uses a service in Windows called the Data Protection API (DPAPI). It takes a "digital snapshot" of the Chrome program itself. It looks at the file's size, its digital signature from Google, and its location. If a virus tries to pretend to be Chrome, it won't have the correct fingerprint. The system will say: "I know you're leroy, but you aren't the real Chrome app, so you can't have the passwords."

You do have the issue of when the malware is in the Chrome browser itself. ABE is designed to stop other apps from stealing Chrome's keys. But if the malware has hijacked the Chrome app itself, ABE gets confused. Because the malware is now part of the real Chrome process, it has a valid "App Fingerprint." When the malware asks for the passwords, the system looks at the request and says, "Yep, this is coming from the official Chrome app." It then hands over the keys. In short, ABE cannot protect you if the "Trusted App" (Chrome) has been compromised by an exploit. However this is not what is happening with Voidstealer.

## What is Voidstealer?
VoidStealer isn't a browser exploit, it's a separate program (malware) that sits outside Chrome and watches it using a debugger. Normally, when you run a program (like Chrome), it runs at lightning speed. You can't see what's happening inside its "brain". A Debugger is a tool used by software engineers to perform "surgery" on a running program. It allows a programmer to:
1. Pause the program at any exact moment.
2. Look inside the program’s memory to see secret data (like passwords being typed).
3. Step through the code one tiny line at a time to see where a mistake (a "bug") is.

VoidStealer sets a tiny, invisible tripwire inside Chrome's brain. In tech terms, this is a Hardware Breakpoint. Think of Chrome as a person at a desk with a locked briefcase full of secrets. The moment Chrome successfully unlocks the briefcase and the passwords are sitting on the desk, the tripwire goes off. Chrome freezes instantly, via this debugger breakpoint. While Chrome is frozen, VoidStealer reaches in, takes a photo of the passwords, and then tells Chrome, "Okay, you can keep working now."

You may be thinking why is this allowed? Well, for software to work properly, it needs to be bug-free, and to catch these bugs, you need a program that can find them. Without debuggers, you would have buggy software everywhere. Usually, Windows requires you to be an Admin (the boss of the computer) to debug another program. Voidstealer found a way to act like a Debugger without needing those "Boss" permissions. It exploited a loophole in how Windows handles Hardware Breakpoints to spy on Chrome without the computer ever sounding an alarm. 

## What is the loophole Voidstealer found?
On Windows, a "Parent" program has a natural right to watch over its "Child" program. Instead of trying to break into a Chrome window that is already open, VoidStealer launches its own version of Chrome. Because VoidStealer started the process, Windows sees VoidStealer as the "Parent". A Parent is allowed to debug its Child without needing special Admin permission. 

This then brings us to the hardware breakpoints. There are two ways to pause a program:
1. Software Breakpoints: These involve changing the program's code (like drawing a big red STOP sign on a road). Security software easily sees these changes and screams "VIRUS!"
2. Hardware Breakpoints (The VoidStealer Way): This uses the CPU (the computer's brain) itself. The CPU has a few tiny storage slots (called Debug Registers) meant for developers. VoidStealer puts the tripwire address into the CPU's physical memory. Since it's happening inside the processor chips and not in the software code, it doesn't look like the code has been tampered with. It's totally invisible to most basic security checks.

VoidStealer doesn't use custom "hacking tools." It uses legitimate Windows functions that are built-in for every user, even those without Admin rights:
1. CreateProcessW: To start Chrome in a "Suspended" (frozen) state. https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw
2. ReadProcessMemory: A standard command that lets the Parent read what the Child is thinking. https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-readprocessmemory
3. SetThreadContext: This is the needle used for the surgery. It tells the CPU, "When you hit this specific spot in the code, pause and tell me." https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadcontext

VoidStealer uses the command called CreateProcessW to launch Chrome. But it adds two secret flags:
1. CREATE_SUSPENDED: This starts Chrome but freezes it instantly before a single line of code runs.
2. DEBUG_PROCESS: This tells Windows, "I am the official debugger for this specific Chrome window."

Because VoidStealer is the "Parent," Windows grants it SeDebugPrivilege for that one specific child process, no Admin password required. 

VoidStealer doesn't just guess where to put the tripwire. It looks for a specific "nerve" in Chrome's brain. It scans the Chrome files (specifically chrome.dll) for a very specific string of text that is only used during the password decryption process. Once it finds that text, it calculates the exact mathematical address in the computer's RAM where the "Master Key" will appear for a split second. In the article this advisory is based on, it states it as v20_master_key. 

![](https://www.gendigital.com/blog/sites/default/files/styles/blogs_paragraph_image_large_2x/public/2026-03/voidstealer_figure_13.jpeg.webp?itok=Vq7XJm0q)

The CPU has 8 special slots called Debug Registers (DR0 through DR7). DR0–DR3 are the slots where you can store an address you want to watch. DR7 is the "on/off switch". To get into these slots, VoidStealer uses a WinAPI command called SetThreadContext. VoidStealer sends a command that says: "Hey CPU, take this memory address and put it into Slot DR0. Then, flip the switch in DR7 to 'Alert me if anyone touches this address'." 

Now that the tripwire is set, VoidStealer "unfreezes" Chrome. Chrome starts up normally and reaches the point where it asks Windows to unlock your passwords. The moment Chrome's code touches that specific address in DR0, the CPU sends a hardware signal (an Exception) that freezes Chrome again. VoidStealer receives a notification: "The tripwire was hit!". It uses ReadProcessMemory to "snap a photo" of the plaintext Master Key, saves it, and then tells Chrome to keep going. Once VoidStealer has that "snapshot" of the v20_master_key (the Master Key), it essentially has the "Skeleton Key". 

The Master Key itself is just a long string of random-looking numbers and letters. It isn't a password you can read. Instead, VoidStealer uses it to decrypt your browser's database files. Your browser stores three main treasure chests on your hard drive:
1. Logins.json / Login Data: Every username and password you've ever told Chrome to "Remember."
2. Cookies: The secret VIP passes that keep you logged into Gmail, Facebook, or your Bank so you don't have to type your password every time.
3. Web Data: Your credit card numbers and addresses used for Auto-fill.

Without the key, these files look like gibberish. With the key, VoidStealer turns them back into plain text instantly. 

VoidStealer doesn't send the files one by one (that would be too easy to spot). Instead, it creates a log file or a hidden folder on your computer. It stuffs everything it found into this folder:
1. Every password.
2. Your browser history.
3. Your location and IP address.
4. A screenshot of your desktop (to see what you're working on).

VoidStealer then connects to a C2 Server, and it uploads your data via an encrypted connection (making it look like normal web traffic). To stay stealthy, many versions of this malware will then delete themselves from your computer. 

## Mitigations
1. Antivirus/EDR - The obvious one, you wouldn't Voidstealer doing all of this if you never got it in the first place. A lot of these have been seen being proliferated through cracked software, so stay away from software piracy.
2. Since the goal is to steal your browser's "Master Key," the best defense is to make sure your most important secrets aren't in the browser to begin with. Use a standalone password manager to store your passwords. 
3. If you aren't using a site, log out. This kills the session cookie so that even if a hacker steals it, it won't work. 
4. Be mindful of ghost Chrome processes. If you dont have Chrome up, but you look in Task Manager and see instances of Chrome, be suspicious.
5. Modern EDR's are getting better at looking for programs trying to debug other programs for no good reason. 
6. Utilize Hardware Security Keys (YubiKey). This is the "Gold Standard." Even if a hacker has your password and your session cookies, they cannot log into your account without physically touching a USB key plugged into your computer.
7. Use app-based 2FA (like Google Authenticator) rather than SMS/Text.

## References
https://www.gendigital.com/blog/insights/research/voidstealer-abe-bypass

## IOC
VoidStealer v2.0 - f783fde5cf7930e4b3054393efadd3675b505cbef8e9d7ae58aa35b435adeea4

















































