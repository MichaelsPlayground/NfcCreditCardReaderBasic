# MasterCard complete logfile

**Note: the Credit Card number shown below is an anonymize Number.**

NFC tag discovered
TagId: 028eedb17074b0
TechList found with these entries:
android.nfc.tech.IsoDep
android.nfc.tech.NfcA

*** Tech ***
Technology IsoDep
try to read a payment card with PPSE


*********************************
************ step  1 ************
* select PPSE                   *
*********************************
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


*********************************
************ step  2 ************
* search applications on card   *
*********************************
02 analyze select PPSE response and search for tag 0x4F (applications on card)
Found tag 0x4F 1 time(s):
application Id (AID): a0000000041010


*********************************
************ step  3 ************
* select application by AID     *
*********************************
03 select application by AID a0000000041010 (number 1)
card is a MasterCard

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


*********************************
************ step  4 ************
* search for tag 0x9F38         *
*********************************
04 search for tag 0x9F38 in the selectAid response

### processing the MasterCard path ###

No PDOL found in the selectAid response
try to request the get processing options (GPO) with an empty PDOL

*********************************
************ step  5 ************
* get the processing options    *
*********************************
05 get the processing options command length: 8 data: 80a8000002830000
05 select GPO response length: 20 data: 771282021980940c080101001001010120010200
------------------------------------
77 12 -- Response Message Template Format 2
82 02 -- Application Interchange Profile
19 80 (BINARY)
94 0C -- Application File Locator (AFL)
08 01 01 00 10 01 01 01 20 01 02 00 (BINARY)
------------------------------------

06 read the files from card and search for tag 0x57 in each file

*********************************
************ step  6 ************
* read files & search PAN       *
*********************************
tag 0x57 not found, try to find in tag 0x94 = AFL
The AFL contains 3 entries to read
data from AFL 08010100
read result length: 119 data: 70759f6c0200019f6206000000000f009f63060000000000fe563442353337353035303030303136303131305e202f5e323430333232313237393433323930303030303030303030303030303030309f6401029f65020f009f660200fe9f6b131122334455667788d24032210000000000000f9f670102
------------------------------------
70 75 -- Record Template (EMV Proprietary)
9F 6C 02 -- Mag Stripe Application Version Number (Card)
00 01 (BINARY)
9F 62 06 -- Track 1 bit map for CVC3
00 00 00 00 0F 00 (BINARY)
9F 63 06 -- Track 1 bit map for UN and ATC
00 00 00 00 00 FE (BINARY)
56 34 -- Track 1 Data
42 35 33 37 35 30 35 30 30 30 30 31 36 30 31 31
30 5E 20 2F 5E 32 34 30 33 32 32 31 32 37 39 34
33 32 39 30 30 30 30 30 30 30 30 30 30 30 30 30
30 30 30 30 (BINARY)
9F 64 01 -- Track 1 number of ATC digits
02 (BINARY)
9F 65 02 -- Track 2 bit map for CVC3
0F 00 (BINARY)
9F 66 02 -- Terminal Transaction Qualifiers
00 FE (BINARY)
9F 6B 13 -- Track 2 Data
11 22 33 44 55 66 77 88D2 40 32 21 00 00 00 00
00 00 0F (BINARY)
9F 67 01 -- Track 2 number of ATC digits
02 (BINARY)
------------------------------------
data from AFL 10010101
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
11 22 33 44 55 66 77 88D2 40 32 21 27 94 32 90
00 00 0F (BINARY)
------------------------------------
data from AFL 20010200
read result length: 187 data: 7081b89f4701039f4681b03cada902afb40289fbdfea01950c498191442c1b48234dcaff66bca63cbf821a3121fa808e4275a4e894b154c1874bddb00f16276e92c73c04468253b373f1e6a9a89e2705b4670682d0adff05617a21d7684031a1cdb438e66cd98d591dc376398c8aab4f137a2226122990d9b2b4c72ded6495d637338fefa893ae7fb4eb845f8ec2e260d2385a780f9fda64b3639a9547adad806f78c9bc9f17f9d4c5b26474b9ba03892a754ffdf24df04c702f86
------------------------------------
70 81 B8 -- Record Template (EMV Proprietary)
9F 47 01 -- ICC Public Key Exponent
03 (BINARY)
9F 46 81 B0 -- ICC Public Key Certificate
3C AD A9 02 AF B4 02 89 FB DF EA 01 95 0C 49 81
91 44 2C 1B 48 23 4D CA FF 66 BC A6 3C BF 82 1A
31 21 FA 80 8E 42 75 A4 E8 94 B1 54 C1 87 4B DD
B0 0F 16 27 6E 92 C7 3C 04 46 82 53 B3 73 F1 E6
A9 A8 9E 27 05 B4 67 06 82 D0 AD FF 05 61 7A 21
D7 68 40 31 A1 CD B4 38 E6 6C D9 8D 59 1D C3 76
39 8C 8A AB 4F 13 7A 22 26 12 29 90 D9 B2 B4 C7
2D ED 64 95 D6 37 33 8F EF A8 93 AE 7F B4 EB 84
5F 8E C2 E2 60 D2 38 5A 78 0F 9F DA 64 B3 63 9A
95 47 AD AD 80 6F 78 C9 BC 9F 17 F9 D4 C5 B2 64
74 B9 BA 03 89 2A 75 4F FD F2 4D F0 4C 70 2F 86 (BINARY)
------------------------------------
data from AFL 20010200
read result length: 227 data: 7081e08f01059f3201039224abfd2ebc115c3796e382be7e9863b92c266ccabc8bd014923024c80563234e8a11710a019081b004cc60769cabe557a9f2d83c7c73f8b177dbf69288e332f151fba10027301bb9a18203ba421bda9c2cc8186b975885523bf6707f287a5e88f0f6cd79a076319c1404fcdd1f4fa011f7219e1bf74e07b25e781d6af017a9404df9fd805b05b76874663ea88515018b2cb6140dc001a998016d28c4af8e49dfcc7d9cee314e72ae0d993b52cae91a5b5c76b0b33e7ac14a7294b59213ca0c50463cfb8b040bb8ac953631b80fa85a698b00228b5ff44223
------------------------------------
70 81 E0 -- Record Template (EMV Proprietary)
8F 01 -- Certification Authority Public Key Index - card
05 (BINARY)
9F 32 01 -- Issuer Public Key Exponent
03 (BINARY)
92 24 -- Issuer Public Key Remainder
AB FD 2E BC 11 5C 37 96 E3 82 BE 7E 98 63 B9 2C
26 6C CA BC 8B D0 14 92 30 24 C8 05 63 23 4E 8A
11 71 0A 01 (BINARY)
90 81 B0 -- Issuer Public Key Certificate
04 CC 60 76 9C AB E5 57 A9 F2 D8 3C 7C 73 F8 B1
77 DB F6 92 88 E3 32 F1 51 FB A1 00 27 30 1B B9
A1 82 03 BA 42 1B DA 9C 2C C8 18 6B 97 58 85 52
3B F6 70 7F 28 7A 5E 88 F0 F6 CD 79 A0 76 31 9C
14 04 FC DD 1F 4F A0 11 F7 21 9E 1B F7 4E 07 B2
5E 78 1D 6A F0 17 A9 40 4D F9 FD 80 5B 05 B7 68
74 66 3E A8 85 15 01 8B 2C B6 14 0D C0 01 A9 98
01 6D 28 C4 AF 8E 49 DF CC 7D 9C EE 31 4E 72 AE
0D 99 3B 52 CA E9 1A 5B 5C 76 B0 B3 3E 7A C1 4A
72 94 B5 92 13 CA 0C 50 46 3C FB 8B 04 0B B8 AC
95 36 31 B8 0F A8 5A 69 8B 00 22 8B 5F F4 42 23 (BINARY)
------------------------------------


*********************************
************ step  7 ************
* print PAN & expire date       *
*********************************
07 get PAN and Expiration date from tag 0x57 (Track 2 Equivalent Data)
data for AID a0000000041010 (MasterCard)
PAN: 1122334455667788
Expiration date (YYMM): 240331

single data retrieved from card
applicationTransactionCounter: NULL
pinTryCounter: 03
lastOnlineATCRegister: NULL
logFormat: 0000000000000000000000000000000000000000000000000000

internalAuthResponse: 137 data: 7781849f4b81805726a7e1ea742ea5e3fba3fa2773192c730cedbb8eb3f57e6b8e376d78f710a10c30480d6dd77b8533df7bef25a73e8fcdb9ca7b1ca8c1ecdb62201c9701bc4acad5afa8270485568fb5b1824e924abfae53ebc5e5f430dd58a02e467e8c7cc48c95e069721e9ec0e047627cf6dd75c6a4b77748259bdddd2c902386316b79af9000
------------------------------------
77 81 84 -- Response Message Template Format 2
9F 4B 81 80 -- Signed Dynamic Application Data
57 26 A7 E1 EA 74 2E A5 E3 FB A3 FA 27 73 19 2C
73 0C ED BB 8E B3 F5 7E 6B 8E 37 6D 78 F7 10 A1
0C 30 48 0D 6D D7 7B 85 33 DF 7B EF 25 A7 3E 8F
CD B9 CA 7B 1C A8 C1 EC DB 62 20 1C 97 01 BC 4A
CA D5 AF A8 27 04 85 56 8F B5 B1 82 4E 92 4A BF
AE 53 EB C5 E5 F4 30 DD 58 A0 2E 46 7E 8C 7C C4
8C 95 E0 69 72 1E 9E C0 E0 47 62 7C F6 DD 75 C6
A4 B7 77 48 25 9B DD DD 2C 90 23 86 31 6B 79 AF (BINARY)
90 00 -- Command successfully executed (OK)
------------------------------------


get the application cryptogram
### tag0x8cFound: 9f02069f03069f1a0295055f2a029a039c019f37049f35019f45029f4c089f34039f21039f7c14
getApplicationCryptoCommand length: 72 data: 80ae80004200000000100000000000000009780000000000097823030100383930312200000000000000000000000000111009000000000000000000000000000000000000000000
getApplicationCryptoResponse length: 43 data: 77299f2701809f360205579f2608fe90705657463b689f10120110a08003240000000000000000000000ff
------------------------------------
77 29 -- Response Message Template Format 2
9F 27 01 -- Cryptogram Information Data
80 (BINARY)
9F 36 02 -- Application Transaction Counter (ATC)
05 57 (BINARY)
9F 26 08 -- Application Cryptogram
FE 90 70 56 57 46 3B 68 (BINARY)
9F 10 12 -- Issuer Application Data
01 10 A0 80 03 24 00 00 00 00 00 00 00 00 00 00
00 FF (BINARY)
------------------------------------





