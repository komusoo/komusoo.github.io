# Command Injection - Low Security (Windows 10)
It begins, learning my first attack type. Right away I read the [scribd](https://www.scribd.com/document/2530476/Php-Endangers-Remote-Code-Execution) and [OWASP](https://www.owasp.org/index.php/Command_Injection) articles to get a familiarity with the attack type. At a glance it looks simple, abuse unsanitized inputs that are sent to the OS (like ping, nslookup, etc.) In this case, it looked lie DVWA does a simple `ping` to whatever IP you provide, and displays the raw `ping` output to the webpage. 

After `1.1.1.1; ls` simply displayed "Ping request could not find host 1.1.1.1;. Please check the name and try again" on the screen, I questioned if I knew understood the nature of this attack. Common sense slapped me across the face as I realized I was using Unix commands on a Windows web app, and once more when I realized I didn't know how to do multiple Windows CLI commands. The included [ss64 link](https://ss64.com/nt/) was an amazing reference to RTFM with.

Hitting the drawing board again, the "breaker" input `1.1.1.1 && dir` worked! The screenshot is below, it displayed both the undesired (to me at least, maybe not the developer) `ping` results, and then the much more desireable contents of the folder this script runs out of. 

A `dir` is fine, but how much further could this go? I was familiar with Unix post-exploitation, but not Windows; I spent a few hours looking into it, but those results will be in another section. Breaking an unprotected app isn't that impressive, so I moved to Medium.

![Injected web app](img/dvwa_cmd_inj_low1.png)

# Command Injections - Medium Security (Windows 10)
To start, I tried the previous `1.1.1.1 && dir` and it displayed 'Bad parameter dir' or whatever parameter I put after the &&. Curious if piping was considered a 'parameter' in this case, tried `1.1.1.1 >> hackerman.txt`. It was worrisome at first seeing absolutely 0 output this time, but then I realized it must have piped it to hackerman.txt, and it did! I tried `8.8.8.8 >> hackerman.txt && dir` which again gave the 'Bad Parameter dir'. 

Trying to imagine the script's logic, I imagined it would check if there was any input besides an IP and then just say it was bad. But that couldn't be true, as ">> hackerman.txt" worked; questioning what && did exactly, I went [back to RTFM](https://ss64.com/nt/syntax-conditional.html). Here it stated && only works if the first command succeeded; naively, I thought `ping` "failed" and && wouldn't run, so I just tried `1.1.1.1 & dir`, as & will just execute the next command. Lo and behold, this worked! But not for the reasons I thought.

Seeing there was a 'source' folder, I ran `1.1.1.1 & cd source & dir` which revealed a medium.php file that I opened with `1.1.1.1 & type medium.php`. The file was cut off a bit due to GUI shenanigans, but I could infer through the comments there was a blacklist the developer implemented. This tells me && must be on it, and not &; getting a bit tired at this point, I cheated and looked at the source code within DVWA (hey, I'm still in the learning phase and did break the app!) This was exactly the case.

Thankful I could expand my thinking (and for this write-up, which made me catch that), I moved on to Hard mode.

![results of 1.1.1.1 & type medium.php](img/dvwa_cmd_inj_med1.png)
