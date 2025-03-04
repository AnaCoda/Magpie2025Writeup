# MagpieCTF 2025 Forensics Writeup

## Eyes Everywhere

> “Surveillance cameras never lie... or do they? The footage from Christina Krypto’s mansion on the night of her murder has gaps—corrupted frames, missing segments, and unexplained distortions. Someone tampered with the system, but who and why?
> 
> The Cybercrime Division recently recovered fragments of a surveillance image, but it's a mess. Your task is to piece it back together, extract any hidden data, and uncover the truth.
> 
> Rumors suggest that Jake "Kaylined" might have been up to his usual tricks, but was he really there? Or is someone else pulling the strings?”

**Downloads:** `security_footage.zip`

----------

## Exploring the unknown (security footage)

```sh
curl -O https://<linkto>/security_footage.zip
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/4fa8419d-2683-4de4-acb1-5d7fab580a07" width="20%">
</p>

First, let’s check if it really is a zip file. After all, CTFs often try to trick you with file extensions... especially in the forensics section.

```sh
> file security_footage.zip
security_footage.zip: Zip archive data, at least v2.0 to extract, compression method=deflate
```

Looks like it is a zip file! Let’s unzip it and see what’s inside...

```sh
> unzip -d security_footage security_footage.zip
Archive:  security_footage.zip
  inflating: security_footage/TLOhPc5G.jpg  
  inflating: security_footage/2V1idHPN.jpg  
  inflating: security_footage/iBSgsTXy.jpg  
  inflating: security_footage/5ZEOKOaT.jpg  
  inflating: security_footage/JF729NWc.jpg  
  inflating: security_footage/D8mYRpEx.jpg
```

Hm... appears to be a bunch of image files. The filenames look interesting, like they could be decoded into some string. Perhaps in base64 format (from experience, base64 looks a bit like that). Let’s take a look at the images themselves, first by checking if they really are images, of course.

```sh
> cd security_footage
> file *.jpg
2V1idHPN.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 320x213, components 3
5ZEOKOaT.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 320x213, components 3
D8mYRpEx.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 320x213, components 3
... etc.

```

Alright, seems like they really are images. Let’s take a look at the images themselves:

<p align="center">
  <img src="https://github.com/user-attachments/assets/555fd0f1-327f-4833-8ed4-30d438d6aca9" width="50%">
</p>


_Creepy, and.. what is that timestamp?_ But, it looks like we could fit them into a collage. Maybe that will indicate what order the filenames are in and then we could decode a message:

<p align="center">
  <img src="https://github.com/user-attachments/assets/a0404300-2ea5-48a3-809d-e29db82c460d" width="30%">
</p>

Alright, if we go with reading order (left to right, then top to bottom), the filenames form the string `5ZEOKOaTTLOhPc5GJF729NWcD8mYRpExiBSgsTXy2V1idHPN`.

Putting that into a base64 decoder such as at [https://www.base64decode.org/](https://www.base64decode.org/), we get... **DRUMROLL...**

```
(L=F$^՜ɘF15]bts
```

Okay, looks like maybe it’s not base64 :(. Even after using cipher identifiers such as at [https://www.dcode.fr/cipher-identifier](https://www.dcode.fr/cipher-identifier), I was not able to decode that string into anything meaningful.

As usual with forensics CTF challenges, we resort to **throwing everything but the kitchen sink at it**, such as a bunch of tools which we might find at places like [https://github.com/gregalletti/CTF_tools](https://github.com/gregalletti/CTF_tools), or regular Linux commands that show us more information.

<p align="center">
  <img width="490" alt="image" src="https://github.com/user-attachments/assets/4e0e4fac-ca1b-4b0b-a990-b12a700e3d68" />
</p>

Using the `stat` command did not show anything useful, only that all the files were created at the same time. I also tried the tool `binwalk` (didn’t reveal anything) which in hindsight was a bit silly since it's for finding embedded files in binary files, which these aren’t. However, desperate times lead to desperate measures, and for some quick things, it's worth trying anyway.

At this point, I tried all the tools I knew off the top of my head and started looking at steganography resources like [https://0xrick.github.io/lists/stego/](https://0xrick.github.io/lists/stego/). I read a bit about each resource and noticed that `steghide` is able to hide data quite well in various image and audio files, including JPEG files!

I could also have used the online tool [https://www.aperisolve.com/](https://www.aperisolve.com/) which itself is a lot of things (but not the kitchen sink) and also runs `steghide`. I believe a combination is best, to help you save time but also not miss tools that may not be included.

So, I try out `steghide` and lo and behold!

```sh
> steghide info 2V1idHPN.jpg 
"2V1idHPN.jpg":
  format: jpeg
  capacity: 870.0 Byte
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "text_6.txt":
    size: 29.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

Amazing! I do a little dance and have a celebratory slice of pizza. Yes, it’s a bit early to celebrate, but we take what we can get at this stage of the CTF...

So we extract them all and take a look:

```sh
> steghide extract -sf 2V1idHPN.jpg 
Enter passphrase: 
wrote extracted data to "text_6.txt".
```

It asks for a passphrase, so I tried the empty passphrase and it worked. However, to make sure the data isn't just written no matter what the passphrase is, I tried something random, which gave the result `steghide: could not extract any data with that passphrase!`.

**Phew.** After extracting them all (text_1..6.txt), they look like this:

```sh
> cat *.txt
bWF$k$nc$~$GllQ1RGe0IxSU5EbkV
TNV8hc18$e$0X1BSSVY$b$0$)$N2V
fTWE3$!$dDNSX$x$$/$zgz$r$dHdF
M25fQV9QRVIkb$+$25$O$fYU5EX3R
IRV8zWUU1X1$K$dpN0h$m$fd0$J$h
JY0$3$hfdEhleV93M3JFX0Jvck59
```

Or without newlines:
```sh
bWF$k$nc$~$GllQ1RGe0IxSU5EbkVTNV8hc18$e$0X1BSSVY$b$0$)$N2VfTWE3$!$dDNSX$x$$/$zgz$r$dHdFM25fQV9QRVIkb$+$25$O$fYU5EX3RIRV8zWUU1X1$K$dpN0h$m$fd0$J$hJY0$3$hfdEhleV93M3JFX0Jvck59
```

## Decrypting the cryptic message

Hmm... interesting! Looks like it might be some kind of encoding. Of course, we probably need to splice them together in the correct order to reveal any message. 

Should we do it in the order of the numbers in the filename, or in the order that the images fit together? Well, I checked and it turns out they are the same!

I’m very thankful for moments like these because it means I don't have to try every future thing on 2+ different paths. Since `cat` reads files sequentially, it already put them in order, so let's try to decipher that string. 

Putting it into online cipher identifiers (as mentioned previously) did not reveal anything. One thought it was a Crypt() hash, but it seems to be too long for a hash. Another was unable to identify it.

Again, base64 is super common in CTFs and apart from the dollar signs, it really does look like base64, so I put it into [Base64 Decode](https://www.base64decode.org/) which reveals:

```
mad
Q%9L||{E$a
WXM
ԗηGtS6U”FۓaND_tHE_3YE5_R!t&Xx_tHey_w3rE_BorN}
```

Okay! It’s super broken, but clearly this is related to base64. My first instinct is to remove all the dollar signs, but that doesn’t work.

**Next idea: Compare it to a similar, valid base64 string.**

We know that flags usually start with `magpieCTF{` and here, the “mad” at the beginning probably should become part of that. So, let’s encode magpieCTF{ to base64 and compare it to the beginning of what we have:

```
magpieCTF{ in base64
bWFncGllQ1RGew==

Beginning of our encoded string
bWF$k$nc$~$GllQ1RGe
```

## Having an epiphany

Notice that they are very similar. The `==` is simply base64 padding so that can be disregarded. The only differences I can see at the beginning are that `$k$` and `$~$` are not present in the correct string… wait a second!

WHAT IF WE REMOVED ALL THE TEXT BETWEEN (AND INCLUDING) THE DOLLAR SIGNS??? COULD IT BE? :O

And we get…

`magpieCTF{B1INDnES5_!s_4_PRIV47e_Ma7t3R_83twE3n_A_PER$on_aND_tHE_3YE5_Wi7H_wHIcH_tHey_w3rE_BorN}`

I submit the flag to make sure its correct (what if it was a super elaborate red herring??), enjoy my new points, and consume another slice of pizza!

<p align="center">
  <img width="300" alt="image" src="https://github.com/user-attachments/assets/01a6e30e-ba04-402d-ac65-e906771cb4fc" />
</p>

