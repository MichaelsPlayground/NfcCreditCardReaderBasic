# Reading a MasterCard

A **MasterCard** is issued by a bank under the licence of MasterCard. There are three flavours of cards 
available on the market: the classic **Credit card**, the **Debit card** and sometimes **Prepaid cards**.

The PAN on the chip may differ from the PAN printed on the card for some internal reasons.

In this document I'm showing the steps to read the PAN and Expiration date of the card by showing 
the commands and responses for each step, a more generalized information is available in the 
[readme](readme.md) document.

A general notice: I could test my program only with the cards I'm obtaining so it might be that your 
card can't get read because of different tags or templates used on your card. If you give me the commands 
and responses up to the point where the reading of your card is failing, I'm trying to help you - 
just open an issue and copy & paste the data from the export file, thank you.

**Warning: the usage of this program reveals secret information from your Credit Card.** 
This program does not send the data anywhere but when exporting the mail and/or the file 
contain these informations. So please be very careful when sending those data using unsecured 
media like Email or simple internet uploading ! Best option is to use the **anonymize PAN**
function available in the menu, it tries to find the PAN and replace it with a dummy PAN
(1122 3344 5566 7788).

In this workflow I'm using a **MasterCard from Augsburger Aktienbank**. The bank was closed mid 2022 
so the PAN and other data shown here are not from an active card.

## step 1: read the main directory using the SELECT PPSE command

```plaintext
01 select PPSE command length 20 data: 00a404000e325041592e5359532e444446303100
01 select PPSE response length 64 data: 6f3c840e325041592e5359532e4444463031a52abf0c2761254f07a000000004101050104465626974204d6173746572436172648701019f0a04000101019000
------------------------------------
6F 3C -- File Control Information (FCI) Template
      84 0E -- Dedicated File (DF) Name
            32 50 41 59 2E 53 59 53 2E 44 44 46 30 31 (BINARY)
      A5 2A -- File Control Information (FCI) Proprietary Template
            BF 0C 27 -- File Control Information (FCI) Issuer Discretionary Data
                     61 25 -- Application Template
                           4F 07 -- Application Identifier (AID) - card
                                 A0 00 00 00 04 10 10 (BINARY)
                           50 10 -- Application Label
                                 44 65 62 69 74 20 4D 61 73 74 65 72 43 61 72 64 (=Debit MasterCard)
                           87 01 -- Application Priority Indicator
                                 01 (BINARY)
                           9F 0A 04 -- [UNKNOWN TAG]
                                    00 01 01 01 (BINARY)
------------------------------------
```
## step 2: analyze the SELECT PPSE response

We are searching for one or more tags **0x4f** that represents the **Application Identifier (AID)**. 
Here we found one AID (A0000000041010), searching in an online directory (see link below) shows it is a MasterCard: 
*MasterCard Credit/Debit (Global)*.
```plaintext
4F 07 -- Application Identifier (AID) - card
      A0 00 00 00 04 10 10 (BINARY)
```
## step 3: select an application on the card

The AID we found ist used for the next step - we are selecting this AID and with this step we are unlocking the card 
for further readings. This important because skipping this step may end in failures when reading data from the card.
```plaintext
03 select AID command length 13 data: 00a4040007a000000004101000
03 select AID response length 84 data: 6f528407a0000000041010a54750104465626974204d6173746572436172649f12104465626974204d6173746572436172648701019f1101015f2d046465656ebf0c119f0a04000101019f6e0702800000303000
------------------------------------
6F 52 -- File Control Information (FCI) Template
      84 07 -- Dedicated File (DF) Name
            A0 00 00 00 04 10 10 (BINARY)
      A5 47 -- File Control Information (FCI) Proprietary Template
            50 10 -- Application Label
                  44 65 62 69 74 20 4D 61 73 74 65 72 43 61 72 64 (=Debit MasterCard)
            9F 12 10 -- Application Preferred Name
                     44 65 62 69 74 20 4D 61 73 74 65 72 43 61 72 64 (=Debit MasterCard)
            87 01 -- Application Priority Indicator
                  01 (BINARY)
            9F 11 01 -- Issuer Code Table Index
                     01 (NUMERIC)
            5F 2D 04 -- Language Preference
                     64 65 65 6E (=deen)
            BF 0C 11 -- File Control Information (FCI) Issuer Discretionary Data
                     9F 0A 04 -- [UNKNOWN TAG]
                              00 01 01 01 (BINARY)
                     9F 6E 07 -- Visa Low-Value Payment (VLP) Issuer Authorisation Code
                              02 80 00 00 30 30 00 (BINARY)
------------------------------------
```
## step 4: search for tag 0x9F38 in the response from step 3

We are searching for the tag **0x9f38** that represents the **Processing Options Data Object List (PDOL)** 
but there is no tag 0x9f38 in the response. As we need to send a Get Processing Option (GPO) command we need  
send an Get Procession Option command with an "empty" PDOL - see step 5)

## step 5: build a Get Processing Option (GPO) command

Below is the structure of a Get Processing Option command (data shown on hex encoding). The description of the 
command can be found in the EMV book 3:
```plaintext
80                CLA, fix value '80'
  A8              INS, fix value 'A8'
    00            P1, fix value '00'
      00          P2, fix value '00'
        02        LC, total length including tag, add 2 to the length of data value below, variabel, here '02'
          83      PDOL related bytestring, fix value '83'
            00    LoD, length of PDOL related data that is following. As no data follows the value is "00"
              00  Le, "length end" separator, fix value '00'

The complete command is
80 A8 00 00 02 83 00 00
```

Note: firing a Get Processing Option command increases the (card internal) **Application Transaction Counter** (ATC). 
This is a non resetable counter that  is 2 byte long, so the **maximum GPO commands are 65535 commands**. 
When the card reaches this value all further readings are blocked and the card gets **unusable or irrevocal 
destroyed**, so be vary careful when programming reader loops including this command.

## step 6: analyze the Get Processing Option response

**Warning**: this step may reveal confidential data like the PAN/CreditCard number - be extreme cautious when publishing 
or posting or sending the response over an unsecure channel like Email !**

The card answers with some useful details, most important is the content of tag 0x94 that is 
the **Application File Locator (AFL)**. This is the directory files to read and retrieve more data 
from the card - proceed with step 8.

```plaintext
05 get the processing options command length: 8 data: 80a8000002830000
05 select GPO response length: 20 data: 771282021980940c080101001001010120010200
------------------------------------
77 12 -- Response Message Template Format 2
      82 02 -- Application Interchange Profile
            19 80 (BINARY)
      94 0C -- Application File Locator (AFL)
            08 01 01 00 10 01 01 01 20 01 02 00 (BINARY)
------------------------------------
```
## step 7: skipped

## step 8: search for tag 0x94 Application File Locator (AFL)

The AFL points to one or more files located on the card are in 4 byte format for each file. After encoding to a 
hex string one entry may look like this: 08 01 01 00. Each byte is a single information in this format:

08 = short file identifier (SFI) | 01 = first record to read | 01 = last record to read | 00 = files included in offline transaction. 

So this is the information for the first entry: read sfi at position 8 and one record (1) (as we should read sector 01 up to sector 01).
```plaintext
94 0C -- Application File Locator (AFL)
      08 01 01 00 10 01 01 01 20 01 02 00 (BINARY)
      
      08 01 01 00: read SFI 08 sector 01 up to sector 01 = 1 sector, data is not included in offline transactions
      10 01 01 01: read SFI 10 sector 01 up to sector 01 = 1 sector, data is included in offline transactions
      20 01 02 00: read SFI 20 sector 01 up to sector 02 = 2 sectors, data is not included in offline transactions
      
```

## step 9: analyze each file read from step 8

Only the data in the second entry of the AFL contains data useful for us. There are 3 tags of interest:
- tag 0x5f24 - Application Expiration Date
- tag 0x5a - Application Primary Account Number (PAN)
- tag 0x57 - Track 2 Equivalent Data

```plaintext
data from AFL 10010101
read command length: 5 data: 00b2011400
read result length: 169 data: 7081a69f420209785f25032203015f24032403315a0811223344556677885f3401009f0702ffc09f080200028c279f02069f03069f1a0295055f2a029a039c019f37049f35019f45029f4c089f34039f21039f7c148d0c910a8a0295059f37049f4c088e0e000000000000000042031e031f039f0d05b4508400009f0e0500000000009f0f05b4708480005f280202809f4a018257131122334455667788d24032212794329000000f
------------------------------------
70 81 A6 -- Record Template (EMV Proprietary)
         9F 42 02 -- Application Currency Code
                  09 78 (NUMERIC)
         5F 25 03 -- Application Effective Date
                  22 03 01 (NUMERIC)
         5F 24 03 -- Application Expiration Date
                  24 03 31 (NUMERIC)
         5A 08 -- Application Primary Account Number (PAN)
               11 22 33 44 55 66 77 88(NUMERIC)
         5F 34 01 -- Application Primary Account Number (PAN) Sequence Number
                  00 (NUMERIC)
         9F 07 02 -- Application Usage Control
                  FF C0 (BINARY)
         9F 08 02 -- Application Version Number - card
                  00 02 (BINARY)
         8C 27 -- Card Risk Management Data Object List 1 (CDOL1)
               9F 02 06 -- Amount, Authorised (Numeric)
               9F 03 06 -- Amount, Other (Numeric)
               9F 1A 02 -- Terminal Country Code
               95 05 -- Terminal Verification Results (TVR)
               5F 2A 02 -- Transaction Currency Code
               9A 03 -- Transaction Date
               9C 01 -- Transaction Type
               9F 37 04 -- Unpredictable Number
               9F 35 01 -- Terminal Type
               9F 45 02 -- Data Authentication Code
               9F 4C 08 -- ICC Dynamic Number
               9F 34 03 -- Cardholder Verification (CVM) Results
               9F 21 03 -- Transaction Time (HHMMSS)
               9F 7C 14 -- Merchant Custom Data
         8D 0C -- Card Risk Management Data Object List 2 (CDOL2)
               91 0a -- Issuer Authentication Data
               8A 02 -- Authorisation Response Code
               95 05 -- Terminal Verification Results (TVR)
               9F 37 04 -- Unpredictable Number
               9F 4C 08 -- ICC Dynamic Number
         8E 0E -- Cardholder Verification Method (CVM) List
               00 00 00 00 00 00 00 00 42 03 1E 03 1F 03 (BINARY)
         9F 0D 05 -- Issuer Action Code - Default
                  B4 50 84 00 00 (BINARY)
         9F 0E 05 -- Issuer Action Code - Denial
                  00 00 00 00 00 (BINARY)
         9F 0F 05 -- Issuer Action Code - Online
                  B4 70 84 80 00 (BINARY)
         5F 28 02 -- Issuer Country Code
                  02 80 (NUMERIC)
         9F 4A 01 -- Static Data Authentication Tag List
                  82 (BINARY)
         57 13 -- Track 2 Equivalent Data
               11 22 33 44 55 66 77 88 D2 40 32 21 27 94 32 90
               00 00 0F (BINARY)
------------------------------------
```

```plaintext

```


```plaintext
07 get PAN and Expiration date from tag 0x57 (Track 2 Equivalent Data)
data for AID a0000000041010 (MasterCard)
PAN: 1122334455667788
Expiration date (YYMM): 240331
```











# Useful links

[Complete list of Application Identifiers (AID):](https://www.eftlab.com/knowledge-base/complete-list-of-application-identifiers-aid)


back to the general readme document: [readme](readme.md)

go the VisaCard document: [VisaCard reading](visacard.md)

go the GiroCard document: [GiroCard reading](girocard.md)

