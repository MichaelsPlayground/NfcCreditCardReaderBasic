# NFC Credit Card reader basic

This is an app that shows how to read a credit card. 

A credit card is not a simple storage device like NTAG21x, Mifare classic or Mifare ultralight tags but a small computer with a 
file directory that needs to get read with the correct commands. Inside of this main directory are subdirectories or 
"applications" because a credit card may serve for special purposes.

Most of the data on the card is in a data structure called **TLV** = **T**ag | **L**ength | **V**alue. To analyze the data 
it is extreme helpful to use a TLV-library, for that reason I'm adding the 


This document will explain the general procedure to get access to the data on the card. In some other documents you get the 
programmaticall commands to read a VisaCard, MasterCard and a (German) GiroCard (see links below).

There are xx steps to read the PAN (primary account number = credit card number) and the card's expiration date that needs to get 
executed in this workflow:

1) read the main directory using the **select PPSE** command. This will retrieve all applications and their **Application ID** available on the card
2) **analyze the select PPSE respond**: search for tag(s) 0x4F (Application Identifier = AID)
3) **select an application**: select an application by its id (AID) found in step 2, after this step the card is unlocked for further readings
4) **search for tag 0x9F38 in the response from step 3**: The tag 0x9F38 is the **Processing Options Data Object List (PDOL)** that we need for 
the next step. The PDOL is a list of tags that the card requests from the reader (usually a payment terminal) and we need to provide the data 
for each element and return the data. Some cards like (some ?) MasterCards do not return a PDOL so we need to proceed with an "empty" PDOL.
5) **build a Get Procession Option command**: The PDOL is a list of tags that the card requests from the reader (usually a payment terminal) 
We need to provide data for each element and return the data in a Get Procession Option (GPO) command. Some cards like (some ?) MasterCards do 
not return 
6) **get the processing options**: providing PDOL-data from step 3 & 4 gives the **AFL application file list** where we can read the data
7) 
8) 
9) 
10) **read the files from card**: think of a file directory and the AFL from step 4 lists all files on the card. Read each file and try
    to find the data we want to show (PAN and expiration date)
11) **search in each file for the tag 0x57**: tag 0x57 is the **Track 2 Equivalent Data** that has the PAN and expiration date as data fields.
12) **get PAN and expiration date** from the content in tag 0x57 value.


## useful links

For response analysis: BER-TLV library: https://github.com/evsinev/ber-tlv

For pretty printing of the response: EMV-NFC-Paycard-Enrollment: https://github.com/devnied/EMV-NFC-Paycard-Enrollment

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
