# GS1_128_Encoder
How to use the GS1 128 encoder for crystal reports. 

# BizOneness: Use Code 128 barcodes in Crystal Reports.  

In the [last newsletter](https://www.linkedin.com/pulse/guide-barcodes-sap-part-1-codes-b1-crystal-reports-steven-lipton), we discussed the basics of barcodes, the common types of EAN/UPC, Code 39, and Code 128. We saw how to use Code 39 in SAP and Crystal reports. I want to show you how to use Code 128 and GS1-128 barcodes to print a shipping label in Crystal Reports. 

Crystal Reports does not have a code 128 barcode natively. You have to add it. There are a few options to do so, though most are more expensive than I wanted to pay. I came up with a free solution, which I'll discuss here, and how to use it with documents and labels. You'll need to know your way around Crystal Reports to do this. If you need help understanding Crystal reports, take a look at my course [SAP Business One: Reporting and Customization] (https://www.linkedin.com/learning/sap-business-one-reporting-and-customization)

I'll build an A6(105 mm x 148 mm ) or 4x6 inch standard label using the GS1 Logistic Label guidelines you can find here. We'll include two barcodes at this stage: a Sales order number and a purchase order number. 
# Download the font
Generally, you can find fonts quickly enough, but the applications to run those fonts are licensed and cost a bit. There is a Google font option for barcodes.

 You can [find the font here](https://fonts.google.com/specimen/Libre+Barcode+128). Click the download button and install the font to your system accordingly. 

# Code 128 Encoding
Code 128 does not use a simple 1:1 coding of characters. There are three possible mappings of characters named appropriately A, B, and C. A is single digit numbers, uppercase letters, basic punctuation symbols, and control characters. B contains single-digit numbers, basic punctuation,  uppercase and lowercase characters. Code C maps to the two-digit numbers from 00 to 99. Most often, Code B and Code C work together, using Code C as a compression of long strings of numbers. A Purchase Order number of [CODEB]34145926  which is nine characters, breaks into the characters for [CodeC] 34 14 45 92 [CodeB] 6, saving three characters. As a one-dimensional code, all the space you save is good. 

As you see in this example, you have characters in the bar code to identify the code set. You also have start and stop characters and a checksum character. The checksum is a mathematical formula that the bar code reader can use to confirm this is a well-formed barcode, stored as a single number between 0 and 103.
      
There's one more special charter, FNC1. FNC1 is used in the GS1-128  variation of Code 128. FNC1 indicates the use of Application Identifiers, or AIs for short. AIs tell the bar code reader and any software taking the reader's input that this is a specific kind of number, such as a Purchase order number. For example, (420) is the AI for Zip Code with the same postal authority, and (400) is the Identifier for the Purchase order number.   

So I could make barcodes of 
(420)60010 and (400)31415926 to specify the PO number and zip code. 

# The encode_128 crystal reports function
After finding that I would need a $1000 a year site license, I began to look at how to make my encoding system. My first thought was to take a shortcut and let ChatGPT do it in a more common language than Crystal Reports functions, then translate. That ended up in a mess which, if you want to details, check out [this article](https://www.linkedin.com/pulse/can-chatgpt-replace-developers-code-review-steven-lipton) where I wrote about the debacle. 

I wrote this the old-fashioned way: coding from scratch, first in Swift and then translating to Crystal Reports Basic.
#  A basic Crystal report. 
We'll need a crystal report to start his process. I'll use one from Sales order data. 

```
SELECT 
   T0.[DocNum],
   T0.[CardCode],
   T0.CardName,
   T0.Address2, 
   T0.NumAtCard, 
   T1.PickIdNo, 
   T1.ItemCode,
   T1.Quantity,
(SELECT TOP 1 T4.CompnyName FROM OADM T4) as CompnyName,
(SELECT TOP 1 T4.CompnyAddr FROM OADM T4) as CompnyAddr
FROM 
   ORDR T0 
   INNER JOIN RDR1 T1  ON T1.DocEntry = T0.DocEntry
WHERE T0.DocNum = {?DocNum@}
```

I'll make my Crystal report from this. If you don't know how you'll find directions in [this earlier newsletter](https://www.linkedin.com/pulse/import-sap-b1-sql-crystal-reports-steven-lipton)

Notice this has a parameter to use with forms in SAP. I'll also add a business name and address for easy use on my label. 
# Page Setup for Labels
Once I add the database, I will set my formatting for the label. Label setup in SAP is counterintuitive. You don't select a page size for the label. When printing from Crystal Reports, SAP B1 always uses a letter-size page, then centers the page. If you change the page size in either SAP or Crystal reports, you'll find you have blank labels when you go to print. Instead, you keep a standard letter or A4 page and change the margins to your label. For 8.5 by 11-inch letter size paper, I change my margins to have a 4 x 6 printing area, forcing everything to the leading top edge like this:



# Make Lour Label
The next step is to make your label. I made a label with this layout, grouping by Sales Order number. I included the items ordered in the details. I put the Shipping address,  sales order, and Purchase Order number in the Group header.  


And in the preview, it looks like this: 

# Adding the code to functions
We're ready for the barcodes. You'll need to add code to your report. 

You'll find the code below under [The Whole Code](#The-Whole-Code). However, I used a feature of Crystal Reports many don't know about: Custom Functions.

If you go to the formula workshop in Crystal Reports, there is a heading in the function tree called Custom Functions. Here's mine with the functions loaded. 

You can create functions here to use in formulas. Functions are beneficial if you use them for multiple formulas with different inputs. That's the case here. I'll use **encodeGS1_128**  three times with different values. Formulas don't have that kind of flexibility. 

You'll need to cut and paste all the functions below into the custom functions, But I'll give you an idea of how to do that with the **isDigit** function. 

Go to **Report>Formula Workshop** in the drop-down menu. The top of the navigator on the left has **Report Custom functions** Right. Click that, and in the menu, select **new**.

You'll be asked for a name for your function. I'll use **isDigit**. 

You'll notice two buttons below the name, plus **Cancel**. **Use Extractor** is for extracting a formula to a function. Click **Use Editor** to add and edit the function manually. 

In the toolbar, you'll see **Crystal Syntax**. We will use the Basic Syntax, based on Microsoft's Visual Basic for Applications. Leaving this as Crystal Syntax will give you errors in the next step, so change it to Basic Syntax. 

The code below changes to the template for a Basic function. I'll copy the isDigit code below and replace the template.   

Click the check button or alt-C to make sure there are no errors. Save the functions and repeat for all the other functions below. 

# Using Barcodes 
We'll add a Sales order, Purchase order, and zip code barcode to the label. My shipping system wants only the sales number, so I'll leave that as only the number, But I'll add the AI for the Purchase order number(400). 

First, the formula for the Sales order number. This barcode is a simple Code 128 without an AI, so the formula for **SO_Barcode** is

```
encodeGS1_128 (ToText({Command.DocNum},0,""),false )
``` 

Encode takes two parameters. The first is a string you are encoding. The second is if this is a GS1 or a code 128 barcode. *false* is a code 128, *true* is a GS1, which adds an extra character **FNC1** to alert that an AI is the next string.

**DocNum** is a number that needs conversion to a string without decimals or thousand separators. It is code 128, so I leave the second parameter **false**. 

We have valid strings for the PO number. We have an AI of (400), so we can prefix the PO Number with the AI and use **true** here. 

```
encodeGS1_128 ("400"+{Command.NumAtCard},true)
```

# Add Barcodes to the Layout
After all the preparation, you can add your barcodes. I'll make some space in the group footer and add the barcodes. It is essential to have a minimum of a quarter-inch leading and trailing margin, referred to as the *quiet space* , but optimally you should have more and include space between barcodes to prevent reading the wrong one. For example, here's the layout for the Purchase order and the Sales order numbers. I've formatted these for center alignment horizontally and vertically with a 20-point font.  
 
In the preview, you can see the encoding. 
 
Change the font to **libre code 128**, 36 points for the PO number, and 40 points for the PO number, and the bar codes appear. The font size is changeable, and depending on your specifications, you may need bigger or smaller barcodes. Try for bigger sizes, as they are more readable.   
# Human-Readable Format 
In most barcode specifications, you'll also need a Human readable format. However, it can be barely human-readable. Thus, the value of the barcode will be in a small font under the barcode. 

For A value without an AI, You just put it in there, center-aligned in a font like **Courier New** or **Arial Narrow** at 9 points.  
For the sales order number, I'd do something like this: 
 
With an AI, it's more challenging. There are two approaches you can take. While you can make a formula, I use a text field. Add the literal text and then drag the field into the text field. 

Those give us a final result of this: 

# Printing
If you set up the margins correctly, this will print like any other document. You can give it try on a page of paper. If you have a label printer, it should print. I tried this on a low-end Comer printer with success. 

Once printed try scanning with a barcode scanner or app and see if the results are the expected values. If not, try making a larger font, being careful not to cut off the ends of the barcode. 

# One More Step to Go
While we have a shipping label, we've been concentrating on shipping information. Most labels focus on product information. Next week, in the last of this series, I'll focus on getting Item master data in to barcodes, and fill out the middle of our label. I'll also introduce the GS1 databar, so you can have both  item and lot information in a single code.  
