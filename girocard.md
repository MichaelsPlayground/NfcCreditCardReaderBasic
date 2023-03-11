# Reading a (German) GiroCard

A **GiroCard** is issued by German banks and can be used locally in shops or at ATMs. As most banks 
work together with Credit Card issuers most cards do have an additional application on the card 
that enabled them for usage in other European countries for shopping and ATM usage.

The cooperation's between the banks and MasterCard ("Maestro") and Visa ("VPay") will stop in the 
next years as MasterCard announced to stop the Maestro service in the next years and probably Visa 
will follow.

A GiroCard is always a **Debit card** and the **PAN** is not the real account number for debiting 
but  combination of a "Bank number" and a part of the account number, so don't use this "PAN" for 
any real bank transaction.

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






