# NFC Credit Card reader basic

This is an app that shows how to read a credit card. 

A credit card is not a simple storage device like NTAG21x, Mifare classic or Mifare ultralight tags but a small computer with a 
file directory that needs to get read with the correct commands. Inside of this main directory are subdirectories or 
"applications" because a credit card may serve for special purposes.

Most of the data on the card is in a data structure called **TLV** = **T**ag | **L**ength | **V**alue. To analyze the data 
it is extreme helpful to use a TLV-library, for that reason I'm adding the **BER-TLV** library from Evgeniy.

As the response from the card is just binary data it is helpful to use a TLV decoder to get a human readable form of the data. If you prefer 
a copy & paste workflow you can use the "official" EMV tool to do the work (link below). If you want to print the response within your program 
you need another library for this task. I used the library **EMV-NFC-Paycard-Enrollment** from Julien Millau. This library is a complete card 
reading library but I'm using just the "pretty print" part of it.

This document will explain the **general procedure** to get access to the data on the card. In some other documents you get the 
programmatically commands to read a **VisaCard**, **MasterCard** and a **(German) GiroCard** (see links below).

There are 7 or 9 steps to read the PAN (primary account number = credit card number) and the card's expiration date that needs to get 
executed in this workflow:

1) read the main directory using the **select PPSE** command. This will retrieve all applications and their **Application ID** available on the card
2) **analyze the select PPSE respond**: search for tag(s) 0x4F (Application Identifier = AID)
3) **select an application**: select an application by its id (AID) found in step 2, after this step the card is unlocked for further readings
4) **search for tag 0x9F38 in the response from step 3**: The tag 0x9F38 is the **Processing Options Data Object List (PDOL)** that we need for 
the next step. The PDOL is a list of tags that the card requests from the reader (usually a payment terminal) and we need to provide the data 
for each element and return the data. Some cards like (some ?) MasterCards do not return a PDOL so we need to proceed with an "empty" PDOL.
5) **build a Get Processing Option command**: We return the data requested in the PDOL to the card in a GPO command 
6) **analyze the Get Processing Option response**: at this point the general workflow divides up in two different workflows. We are searching for 
the tag 0x94 that is the **Application File Locator (AFL)**. This is the directory files to read and retrieve more data from the card - proceed 
with step 8. But not all cards provide this information. The VisaCards I analyzed do not return the AFL but give us enough information included 
in tag **0x57 (Track 2 Equivalent Data)** - proceed with step 7.
7) **search for tag 0x57 (Track 2 Equivalent Data)**: The value of this tag is a long data field and the first 16 characters (after encoding the 
byte array to a hex string) are the **PAN (Primary Account Number)** that is the credit card number. The next character is a "D" working as 
separator to the next 4 characters that are the **Expiration Date** in the format "YYMM". The card reading workflow can end at this point.
8) **search for tag 0x94 Application File Locator (AFL)**: The AFL points to one or more files located on the card are in 4 byte format for each file. 
After encoding to a hex string one entry may look like this: 08 01 01 00. Each byte is a single information in this format: 
08 = short file identifier (SFI) | 01 = first record to read | 01 = last record to read | 00 = files included in offline transaction. So this is the 
information: read sfi at position 8 and one record (1).
9) **analyze each file read from step 8**: there are some tags to search for to retrieve the data we want to get. It could be tag 0x9F6B 
(Track 2 Data), tag 0x57 (Track 2 Equivalent Data) or tag 0x5a (Application Primary Account Number (PAN)) that contain the card number. 
Finding tag 0x5f24 gives the Application "Expiration Date". The reading workflow can end at this point.

Note: If you know the AID you want to read you can skip the steps 1) and 2) and start directly with step 3).

## Links to specific card type readings

[VisaCard reading](visacard.md)

[MasterCard reading](mastercard.md)

[GiroCard reading](girocard.md)

Complete logfiles from card reading:

[complete VisaCard logfile:](visacard_logfile.md)

[complete VisaCard logfile (second card):](visacard2_logfile.md)

## useful links

PPSE (Paypass Payment System Environment)

For response analysis: BER-TLV library: https://github.com/evsinev/ber-tlv

For a human readable view use the "official" TLV decoder: https://emvlab.org/tlvutils/

Here is link that will show the response from step 3: https://emvlab.org/tlvutils/?data=6f5d8407a0000000031010a5525010564953412044454249542020202020208701029f38189f66049f02069f03069f1a0295055f2a029a039c019f37045f2d02656ebf0c1a9f5a0531082608269f0a080001050100000000bf6304df200180

For pretty printing of the response: EMV-NFC-Paycard-Enrollment: https://github.com/devnied/EMV-NFC-Paycard-Enrollment

Soundfiles: https://mobcup.net/ringtone/ping-euf272ye/download/mp3

Below is a full workflow for the steps above. In most cases there are 3 parts for each step:
- the command send to the card in hex encoding
- the response from the card in hex encoding
- the human readable analyze of the response, manually by copy & paste from a TLV decoder website.

For my manual analysis I used the "official" website https://emvlab.org/tlvutils/

Second note: the response from the card has 2 additional bytes at the end (0x9000) that indication that the processing was successful.
For better reading experience I cut them off.

build.gradle (application):
```plaintext
    ...
    // parsing BER-TLV encoded data, e.g. a credit card
    implementation 'com.payneteasy:ber-tlv:1.0-11'
    implementation group: 'org.slf4j', name: 'slf4j-api', version: '1.7.36'
    // https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12
    implementation group: 'org.slf4j', name: 'slf4j-log4j12', version: '1.7.36', ext: 'pom'

    // pretty print for responses
    implementation 'com.github.devnied.emvnfccard:library:3.0.1'
```
