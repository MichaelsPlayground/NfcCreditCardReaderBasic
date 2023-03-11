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

## step 1: read the main directory using the **SELECT PPSE** command




back to the general readme document: [readme](readme.md)

go the VisaCard document: [VisaCard reading](visacard.md)

go the MasterCard document: [MasterCard reading](mastercard.md)

go the GiroCard document: [GiroCard reading](girocard.md)


