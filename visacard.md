# Reading a VisaCard

A **VisaCard** is issued by a bank under the licence of Visa. The are three flavours of cards
available on the market: the classic **Credit card**, the **Debit card** and sometimes **Prepaid cards**.

The Primary Account Number (PAN) on the chip may differ from the PAN printed on the card for some internal reasons.

In this document I'm showing the steps to read the PAN and Expiration date of the card by showing 
the commands and responses for each step, a more generalized information is available in the 
[readme](readme.md) document.

A general notice: I could test my program only with the cards I'm obtaining so it might be that your 
card can't get read because of different tags or templates used on your card. If you give me the commands 
and responses up to the point where the reading of your card is failing I'm trying to help you - 
just open an issue and copy & paste the data from the export file, thank you.

**Warning: the usage of this program reveals secret information from your Credit Card.** 
This program does not send the data anywhere but when exporting the mail and/or the file 
will contain these informations. So please be very careful when sending those data using unsecured 
media like Email or simple internet uploading or posting anywhere!

In this workflow I'm using a **VisaCard from Lloyds Bank** that was terminated some months ago so the PAN and 
other data shown here are not from an active card.

## step 1: read the main directory using the SELECT PPSE command

```plaintext
01 select PPSE command length 20 data: 00a404000e325041592e5359532e444446303100
01 select PPSE response length 47 data: 6f2b840e325041592e5359532e4444463031a519bf0c1661144f07a00000000310109f0a080001050100000000
------------------------------------
6F 2B -- File Control Information (FCI) Template
      84 0E -- Dedicated File (DF) Name
            32 50 41 59 2E 53 59 53 2E 44 44 46 30 31 (BINARY)
      A5 19 -- File Control Information (FCI) Proprietary Template
            BF 0C 16 -- File Control Information (FCI) Issuer Discretionary Data
                     61 14 -- Application Template
                           4F 07 -- Application Identifier (AID) - card
                                 A0 00 00 00 03 10 10 (BINARY)
                           9F 0A 08 -- [UNKNOWN TAG]
                                    00 01 05 01 00 00 00 00 (BINARY)
------------------------------------
```
## step 2: analyze the SELECT PPSE response

We are searching for one or more tags **0x4f** that represents the **Application Identifier (AID)**. 
Here we found one AID (A0000000031010), searching in an online directory (see link below) shows it is a VisaCard: 
*VISA Debit/Credit (Classic)*.

```plaintext
4F 07 -- Application Identifier (AID) - card
      A0 00 00 00 03 10 10 (BINARY)
```

## step 3: select an application on the card

The AID we found ist used for the next step - we are selecting this AID and with this step we are unlocking the card 
for further readings. This important because skipping this step may end in failures when reading data from the card.

```plaintext
03 select AID command length 13 data: 00a4040007a000000003101000
03 select AID response length 95 data: 6f5d8407a0000000031010a5525010564953412044454249542020202020208701029f38189f66049f02069f03069f1a0295055f2a029a039c019f37045f2d02656ebf0c1a9f5a0531082608269f0a080001050100000000bf6304df200180
------------------------------------
6F 5D -- File Control Information (FCI) Template
      84 07 -- Dedicated File (DF) Name
            A0 00 00 00 03 10 10 (BINARY)
      A5 52 -- File Control Information (FCI) Proprietary Template
            50 10 -- Application Label
                  56 49 53 41 20 44 45 42 49 54 20 20 20 20 20 20 (=VISA DEBIT      )
            87 01 -- Application Priority Indicator
                  02 (BINARY)
            9F 38 18 -- Processing Options Data Object List (PDOL)
                     9F 66 04 -- Terminal Transaction Qualifiers
                     9F 02 06 -- Amount, Authorised (Numeric)
                     9F 03 06 -- Amount, Other (Numeric)
                     9F 1A 02 -- Terminal Country Code
                     95 05 -- Terminal Verification Results (TVR)
                     5F 2A 02 -- Transaction Currency Code
                     9A 03 -- Transaction Date
                     9C 01 -- Transaction Type
                     9F 37 04 -- Unpredictable Number
            5F 2D 02 -- Language Preference
                     65 6E (=en)
            BF 0C 1A -- File Control Information (FCI) Issuer Discretionary Data
                     9F 5A 05 -- Terminal transaction Type (Interac)
                              31 08 26 08 26 (BINARY)
                     9F 0A 08 -- [UNKNOWN TAG]
                              00 01 05 01 00 00 00 00 (BINARY)
                     BF 63 04 -- [UNKNOWN TAG]
                              DF 20 01 -- [UNKNOWN TAG]
                                       80 (BINARY)
------------------------------------
```

## step 4: search for tag 0x9F38 in the response from step 3

We are searching for the tag **0x9f38** that represents the **Processing Options Data Object List (PDOL)**. This is the result:

```plaintext:
9F 38 18 -- Processing Options Data Object List (PDOL)
         9F 66 04 -- Terminal Transaction Qualifiers
         9F 02 06 -- Amount, Authorised (Numeric)
         9F 03 06 -- Amount, Other (Numeric)
         9F 1A 02 -- Terminal Country Code
         95 05 -- Terminal Verification Results (TVR)
         5F 2A 02 -- Transaction Currency Code
         9A 03 -- Transaction Date
         9C 01 -- Transaction Type
         9F 37 04 -- Unpredictable Number
```

Don't get confused about the kind of data requested: usually the card is used in a payment terminal for paying 
goods, restaurant bills or getting cash at the ATM so the data is part of the transaction initiating.

The card requests these data for the next step and we do need to provide the data in the length that is given in the 
last byte of the each subtag. Here are two examples:

The card is asking for "Terminal Transaction Qualifiers" (tag 0x9f66) and the length has to be 4 bytes long ("04"):
```plaintext
9F 66 04 -- Terminal Transaction Qualifiers
```

The card is asking for the "Transaction Date" (tag 9a) that has to by 3 bytes long ("03"):
```plaintext
9A 03 -- Transaction Date
```

When adding all length information's (4+6+..+1+4) we get a **total length of 33 bytes** that has to be returned to the card.

Unfortunately we now have to examine each tag and find useful data to provide. The transaction date e.g. needs to be a 
date in the format YYMMDD (byte array in hex string encoding). Luckily most of the data fields can be zeroed so there is 
not much work to do but - so data fields need to have "accepted" data.

For this purpose I'm using a lookup class were some tags are stored with a working dataset 
(DolValues.java class in nfccreditcards package), e.g.:
```plaintext
tag 0x9f66 Terminal Transaction Qualifiers: A0 00 00 00
tag 0x9f1a Terminal Country Code: 09 78
...
```

Note: each card issuer can request an individual set of tags so it is possible that your card is requesting some "exotic" data.

## step 5: build a Get Processing Option (GPO) command 

The PDOL data are embedded in a command and here are all data elements in one row: `a00000000000000010000000000000000978000000000009782303010038393031`. 
This string is 66 characters long, after conversion to a byte array the length is 33 byte - that is the sum of all tags requested by the card in step 4.

Note: firing a Get Processing Option command increases the (card internal) **Application Transaction Counter** (ATC). This is non resetable counter that 
is 2 byte long, so the **maximum GPO commands are 65535 commands**. When the card reaches this value all further readings are blocked and the card gets 
**unusable or irrevocal destroyed**, so be vary careful when programming reader loops including this command.

## step 6: analyze the Get Processing Option response

**Warning**: this step may reveal confidential data like the PAN/CreditCard number - be extreme cautious when publishing or posting or sending the response over
an unsecure channel like Email !**

The card answers with useful details, most important is the content of tag 0x57.

```plaintext
05 get the processing options command length: 41 data: 80a80000238321a0000000000000001000000000000000097800000000000978230301003839303100
05 run GPO response length: 73 data: 77478202200057131122334455667788d25022013650000000000f5f3401009f100706040a03a020009f2608011ab2fe7d6dd2309f2701809f360203689f6c0216009f6e0420700000
------------------------------------
77 47 -- Response Message Template Format 2
      82 02 -- Application Interchange Profile
            20 00 (BINARY)
      57 13 -- Track 2 Equivalent Data
            11 22 33 44 55 66 77 88D2 50 22 01 36 50 00 00
            00 00 0F (BINARY)
      5F 34 01 -- Application Primary Account Number (PAN) Sequence Number
               00 (NUMERIC)
      9F 10 07 -- Issuer Application Data
               06 04 0A 03 A0 20 00 (BINARY)
      9F 26 08 -- Application Cryptogram
               01 1A B2 FE 7D 6D D2 30 (BINARY)
      9F 27 01 -- Cryptogram Information Data
               80 (BINARY)
      9F 36 02 -- Application Transaction Counter (ATC)
               03 68 (BINARY)
      9F 6C 02 -- Mag Stripe Application Version Number (Card)
               16 00 (BINARY)
      9F 6E 04 -- Visa Low-Value Payment (VLP) Issuer Authorisation Code
               20 70 00 00 (BINARY)
------------------------------------
```

**Note: only one of my VisaCards showed a different behaviour - the card didn't answered with a
Track 2 Equivalent Data but an Application File Locator (AFL). At this point we are following the steps 8) 
and 9) in the general workflow - scroll down to see the difference.** 

# step 7: search for tag 0x57 (Track 2 Equivalent Data)

The **Track 2 Equivalent Data** are a relic from the time when there was just a magnet stripe on the card that had 2 tracks. 
As the storage was limited on those storage medium all data are compressed, so lets have a deeper look into the data:

```plaintext
57 13 -- Track 2 Equivalent Data
      11 22 33 44 55 66 77 88D2 50 22 01 36 50 00 00
```

Again a warning regarding the revealing of confidential data: the Credit Card number shown here is from a 
credit card that is not any more in use since some months and the card number is blocked by the bank. I 
anonymize the output so there is a "1122 3344 5566 7788" shown as dummy number.

The data are the hex encoded string of a 13 byte long byte array. The first 16 character (8 bytes) are 
the Credit Card number we are looking for (1122 3344 5566 7788). The number is followed by a "D" character 
that works a separator to the next field that is the 4 characters long Expiration date (format YYMM, 25 02 
= End of February 2025).

```plaintext
07 get PAN and Expiration date from tag 0x57 (Track 2 Equivalent Data)
data for AID a0000000031010 (VISA credit/debit)
PAN: 1122334455667788
Expiration date (YYMM): 2502
```

At his point the reading can stop, my program retrieves 4 additional data fields from the card: the Application 
Transaction Counter, the left PIN try counter, the Last Online Transaction Counter and the log format.

At last the program tries to run the "getApplicationCryptoCommand" but it fails, probably because my card doesn't 
have the function available.

On a separate page you can see the complete logfile for this reading (see link section below).

The following steps 8) and 9) are from a different behaviour from step 6)

## step 8: search for tag 0x94 Application File Locator (AFL)

The response of the GetProcessingOptions command in step 6) was different to all of my other VisaCards:

```plaintext
05 get the processing options command length: 41 data: 80a80000238321a0000000000000001000000000000000097800000000000978230301003839303100
05 run GPO response length: 201 data: 7781c68202000094041002040057131122334455667788d26092012166408100000f9f100706010a03a020009f26082076e13635995cfb9f2701809f3602005f9f6c0238009f4b81800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
------------------------------------
77 81 C6 -- Response Message Template Format 2
         82 02 -- Application Interchange Profile
               00 00 (BINARY)
         94 04 -- Application File Locator (AFL)
               10 02 04 00 (BINARY)
         57 13 -- Track 2 Equivalent Data
               11 22 33 44 55 66 77 88D2 60 92 01 21 66 40 81
               00 00 0F (BINARY)
         9F 10 07 -- Issuer Application Data
                  06 01 0A 03 A0 20 00 (BINARY)
         9F 26 08 -- Application Cryptogram
                  20 76 E1 36 35 99 5C FB (BINARY)
         9F 27 01 -- Cryptogram Information Data
                  80 (BINARY)
         9F 36 02 -- Application Transaction Counter (ATC)
                  00 5F (BINARY)
         9F 6C 02 -- Mag Stripe Application Version Number (Card)
                  38 00 (BINARY)
         9F 4B 81 80 -- Signed Dynamic Application Data
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 (BINARY)
```

The card provided data for tag 0x94 **Application File Locator (AFL)** together with the known tag 0x57
**Track 2 Equivalent Data**.

We can read the data from tag 0x57 to get the PAN and the Expiration date of the card of course or we can read the 
content of a file on the card. Here I'm providing just a very short explanation how to get the file's data, a more 
detailed information is available in the MasterCard file.

The value of the AFL is a 4 byte coded information:

```plaintext
94 04 -- Application File Locator (AFL)
      10 02 04 00 (BINARY)
```

We can decode the data as follows (explained like a regular storage device):
```plaintext
10 => is like the comand: go into subfolder 10 of your storage device
02 => read file at position 02
04 => read the following files up to number 04, so read files 03 and 04
00 => this is just a notice "files involved in offline transactions", we don't need this information for our workflow
```

This is the result of reading all 3 files (02, 03 and 04):

```plaintext
The AFL contains 1 entries to read

data from AFL 10020400
read result length: 254 data: 7081fb9081f88893cf85a81325ab8da6a4196eb5787291db7205f61b172b26deb867da427f1d0e438e86400aea81a0f2826b250da618108389bdabe2a75c0168a28bb97645158b57ca8faa1d38d7a56e0a4171ec0d5e048d048dd98106bcadb3b5cac80485ff9c0fc970b4ea95d557fb9dd065bf75eb06f51df5a2c20479058ede6c8a376d9bfbf0c05b9e2b5aac1ec5982e2a9d861573e892da87b68357306e88cb054ab0090e01670a73d23fa239f4ae1283110fca40d46edc6c8021d15b3c147251b3c5e754f0fa9d82b7934ed34a12ef3d0a66c0c2a26a32e9722b10653516b356440aa8eece8d1d023829394adc2f9309ff60fc5baf51c0b24690be
------------------------------------
70 81 FB -- Record Template (EMV Proprietary)
         90 81 F8 -- Issuer Public Key Certificate
                  88 93 CF 85 A8 13 25 AB 8D A6 A4 19 6E B5 78 72
                  91 DB 72 05 F6 1B 17 2B 26 DE B8 67 DA 42 7F 1D
                  0E 43 8E 86 40 0A EA 81 A0 F2 82 6B 25 0D A6 18
                  10 83 89 BD AB E2 A7 5C 01 68 A2 8B B9 76 45 15
                  8B 57 CA 8F AA 1D 38 D7 A5 6E 0A 41 71 EC 0D 5E
                  04 8D 04 8D D9 81 06 BC AD B3 B5 CA C8 04 85 FF
                  9C 0F C9 70 B4 EA 95 D5 57 FB 9D D0 65 BF 75 EB
                  06 F5 1D F5 A2 C2 04 79 05 8E DE 6C 8A 37 6D 9B
                  FB F0 C0 5B 9E 2B 5A AC 1E C5 98 2E 2A 9D 86 15
                  73 E8 92 DA 87 B6 83 57 30 6E 88 CB 05 4A B0 09
                  0E 01 67 0A 73 D2 3F A2 39 F4 AE 12 83 11 0F CA
                  40 D4 6E DC 6C 80 21 D1 5B 3C 14 72 51 B3 C5 E7
                  54 F0 FA 9D 82 B7 93 4E D3 4A 12 EF 3D 0A 66 C0
                  C2 A2 6A 32 E9 72 2B 10 65 35 16 B3 56 44 0A A8
                  EE CE 8D 1D 02 38 29 39 4A DC 2F 93 09 FF 60 FC
                  5B AF 51 C0 B2 46 90 BE (BINARY)
------------------------------------

data from AFL 10020400
read result length: 9 data: 70079f3201038f0109
------------------------------------
70 07 -- Record Template (EMV Proprietary)
      9F 32 01 -- Issuer Public Key Exponent
               03 (BINARY)
      8F 01 -- Certification Authority Public Key Index - card
            09 (BINARY)
------------------------------------

data from AFL 10020400
read result length: 238 data: 7081eb9f4681b07e3b33a489fb75a23643407d2ebf48a808957165aa538d681213d71495b577086e63a24e847ed29d2ceba4bb3b1784361221287607ace4b8bfce09dd8364d4709293ed52b528623472fb6157094b12367534d7cf5c20b810058c817fb87c130111ee53c3855fd2b2a95449d03795541ea7c6ef942b0b069bfa7caa5d0ec6db0e428f18d03adcf7f92fb7e5516403adc629f3ffbd6900a1f308fbe5d28cba795c6c62d7573333abed15ad00a4da4ba8a99f4701035a0811223344556677885f24032609305f280202765f3401009f0702c0809f4a01829f6e04207000009f690701000000000000
------------------------------------
70 81 EB -- Record Template (EMV Proprietary)
         9F 46 81 B0 -- ICC Public Key Certificate
                     7E 3B 33 A4 89 FB 75 A2 36 43 40 7D 2E BF 48 A8
                     08 95 71 65 AA 53 8D 68 12 13 D7 14 95 B5 77 08
                     6E 63 A2 4E 84 7E D2 9D 2C EB A4 BB 3B 17 84 36
                     12 21 28 76 07 AC E4 B8 BF CE 09 DD 83 64 D4 70
                     92 93 ED 52 B5 28 62 34 72 FB 61 57 09 4B 12 36
                     75 34 D7 CF 5C 20 B8 10 05 8C 81 7F B8 7C 13 01
                     11 EE 53 C3 85 5F D2 B2 A9 54 49 D0 37 95 54 1E
                     A7 C6 EF 94 2B 0B 06 9B FA 7C AA 5D 0E C6 DB 0E
                     42 8F 18 D0 3A DC F7 F9 2F B7 E5 51 64 03 AD C6
                     29 F3 FF BD 69 00 A1 F3 08 FB E5 D2 8C BA 79 5C
                     6C 62 D7 57 33 33 AB ED 15 AD 00 A4 DA 4B A8 A9 (BINARY)
         9F 47 01 -- ICC Public Key Exponent
                  03 (BINARY)
         5A 08 -- Application Primary Account Number (PAN)
               11 22 33 44 55 66 77 88(NUMERIC)
         5F 24 03 -- Application Expiration Date
                  26 09 30 (NUMERIC)
         5F 28 02 -- Issuer Country Code
                  02 76 (NUMERIC)
         5F 34 01 -- Application Primary Account Number (PAN) Sequence Number
                  00 (NUMERIC)
         9F 07 02 -- Application Usage Control
                  C0 80 (BINARY)
         9F 4A 01 -- Static Data Authentication Tag List
                  82 (BINARY)
         9F 6E 04 -- Visa Low-Value Payment (VLP) Issuer Authorisation Code
                  20 70 00 00 (BINARY)
         9F 69 07 -- UDOL
                  01 00 00 00 00 00 00 (BINARY)
------------------------------------
```

## step 9: analyze each file read from step 8

The only interesting data is in the last file where we find the tags 0x5a **Application Primary Account Number (PAN)** 
and 0x5f24 **Application Expiration Date**:

```plaintext
5A 08 -- Application Primary Account Number (PAN)
      11 22 33 44 55 66 77 88(NUMERIC)
5F 24 03 -- Application Expiration Date
      26 09 30 (NUMERIC)
```

The data in tag 0x5a is the hex encoded string of a 8 byte long byte array with the Credit Card number we are 
looking for (1122 3344 5566 7788). Tag 0x5f24 gives the 3 bytes long Application Expiration date (format YYMMDD, 26 09 30 
= End of September 2026).

```plaintext
07 get PAN and Expiration date from tag 0x57 (Track 2 Equivalent Data)
data for AID a0000000031010 (VISA credit/debit)
PAN: 1122334455667788
Expiration date (YYMM): 260930

single data retrieved from card
applicationTransactionCounter: 005e (hex), 94 (dec)
pinTryCounter: 02
lastOnlineATCRegister: 0041
logFormat: 9f27019f36029f02069f03069f1a025f2a0295059a039c01

get the application cryptogram
### tag0x8cFound:
no CDOL1 found in files, using an empty one
getApplicationCryptoCommand length: 6 data: 80ae80000000
getApplicationCryptoResponse failure

```

# Useful links

[Complete list of Application Identifiers (AID):](https://www.eftlab.com/knowledge-base/complete-list-of-application-identifiers-aid)

[complete VisaCard logfile:](visacard_logfile.md)

[complete VisaCard logfile (second card):](visacard2_logfile.md)

back to the general readme document: [readme](readme.md)

go the MasterCard document: [MasterCard reading](mastercard.md)

go the GiroCard document: [GiroCard reading](girocard.md)


