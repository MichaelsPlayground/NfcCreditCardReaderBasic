# Reading a VisaCard

A **VisaCard** is issued by a bank under the licence of Visa. The are three flavours of cards
available on the market: the classic **Credit card**, the **Debit card** and sometimes **Prepaid cards**.

The PAN on the chip may differ from the PAN printed on the card for some internal reasons.

In this document I'm showing the steps to read the PAN and Expiration date of the card by showing 
the commands and responses for each step, a more generalized information is available in the 
[readme](readme.md) document.

A general notice: I could test my program only with the cards I'm obtaining so it might be that your 
card can't get read because of different tags or templates used on your card. If you give me the commands 
and responses up to the point where the reading of your card is failing I'm trying to help you - 
just open an issue and copy & paste the data from the export file, thank you.

**Warning: the usage of this program reveals secret information from your Credit Card.** 
This program does not send the data anywhere but when exporting the mail and/or the file 
contain these informations. So please be very careful when sending those data using unsecured 
media like Email or simple internet uploading !

In this workflow I'm using a **VisaCard from Lloyds Bank** that was cancelled some months ago so the PAN and 
other data shown here not from an active card.

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

```plaintext
9F 66 04 -- Terminal Transaction Qualifiers
```
The card is asking for "Terminal Transaction Qualifiers" (tag 0x9f66) and the length has to be 4 bytes long ("04").

```plaintext
9A 03 -- Transaction Date
```
The card is asking for the "Transaction Date" (tag 9a) that has to by 3 bytes long ("03").

When adding all length informations (4+6+..+1+4) we get a total length of 33 bytes that has to be returned to the card.

Unfortunately we now have to examine each tag and find useful data to provide. The transaction date e.g. needs to be a 
date in the format YYMMDD (byte array in hex string encoding). Luckily most of the data fields can be zeroed so there is 
not much work to do but - so data fields need to have "accepted" data.

For this purpose I'm using a lookup class were some tags are stored with a working dataset 
(DolValues.java class nfccreditcards package), e.g.:
```plaintext
tag 0x9f66 Terminal Transaction Qualifiers: A0 00 00 00
tag 0x9f1a Terminal Country Code: 09 78
...
```

Note: each card issuer can request an individual set of tags so it is possible that your card is requesting some "exotic" data.

## step 5 build a Get Procession Option command

The PDOL data are embedded in a command and here are all data elements in one row: `a00000000000000010000000000000000978000000000009782303010038393031`.

```plaintext
05 get the processing options command length: 41 data: 80a80000238321a0000000000000001000000000000000097800000000000978230301003839303100
05 run GPO response length: 73 data: 77478202200057134921828094896752d25022013650000000000f5f3401009f100706040a03a020009f2608cc297291b6d757e49f2701809f360203639f6c0216009f6e0420700000
------------------------------------
77 47 -- Response Message Template Format 2
      82 02 -- Application Interchange Profile
            20 00 (BINARY)
      57 13 -- Track 2 Equivalent Data
            49 21 82 80 94 89 67 52 D2 50 22 01 36 50 00 00
            00 00 0F (BINARY)
      5F 34 01 -- Application Primary Account Number (PAN) Sequence Number
               00 (NUMERIC)
      9F 10 07 -- Issuer Application Data
               06 04 0A 03 A0 20 00 (BINARY)
      9F 26 08 -- Application Cryptogram
               CC 29 72 91 B6 D7 57 E4 (BINARY)
      9F 27 01 -- Cryptogram Information Data
               80 (BINARY)
      9F 36 02 -- Application Transaction Counter (ATC)
               03 63 (BINARY)
      9F 6C 02 -- Mag Stripe Application Version Number (Card)
               16 00 (BINARY)
      9F 6E 04 -- Visa Low-Value Payment (VLP) Issuer Authorisation Code
               20 70 00 00 (BINARY)
------------------------------------
```



# Useful links

[Complete list of Application Identifiers (AID):](https://www.eftlab.com/knowledge-base/complete-list-of-application-identifiers-aid)


back to the general readme document: [readme](readme.md)

go the VisaCard document: [VisaCard reading](visacard.md)

go the MasterCard document: [MasterCard reading](mastercard.md)

go the GiroCard document: [GiroCard reading](girocard.md)


