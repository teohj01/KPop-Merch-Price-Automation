# K-pop Merch Price Automation

## Background

A **Korean proxy shopping service** helps overseas K-pop fans buy merchandise from Korea, then ships the items to them.

These services are useful because buying directly from Korea is not always straightforward:

1. **Many Korean online stores are difficult to buy from overseas.** Some require a Korean phone number or do not accept foreign cards.
2. **Some merch is only available at pop-up stores.** Limited-edition items sold at Seoul pop-ups may not be available anywhere else.
3. **Fan events are difficult to join from abroad.** Ticketing can require a Korean phone number, while fansign entries usually involve buying albums for a chance to enter through a lucky draw.
4. **Even official global stores can be inconvenient and expensive.** International payment options can be limited, and paying international shipping for every separate order quickly adds up.

For a proxy service in Singapore, quoting a customer is not as simple as converting KRW to SGD. The final price of every item needs to account for several components:

| Component | What it covers |
|---|---|
| **Exchange Rate** | Converting the item's KRW price to SGD |
| **Shipping** | Korea-to-Singapore shipping based on the item's weight |
| **Commission** | The service fee |
| **Purchase Fee** | Handling cost for each item |

---

## The Problem

K-pop merch drops happen frequently, and many are only available for a short period of time. When a new collection drops, a proxy service may need to list dozens of products for pre-order as quickly as possible.

Previously, this entire process was manual.

For every new drop, someone had to:

1. Read the product names and KRW prices from a shop page or promotional poster found on social media
2. Copy each product into a spreadsheet one by one
3. Find the appropriate shipping weight for each item
4. Calculate the final selling price in SGD
5. Prepare the product name and price for listing

Doing this for dozens of products was repetitive and time-consuming. More importantly, it left room for human error.

**One mistyped digit in a KRW price could directly change how much a customer was charged.**

I wanted to reduce the manual work without turning pricing into a black box that nobody could verify.

---

## What I Built

I built an automated workflow that takes a product from its original source and turns it into a ready-to-list SGD price in Google Sheets.

Products can enter the workflow in three ways:

1. **Scan a Weverse Shop page** using the Chrome extension
2. **Right-click a product poster or image** found on X
3. **Drop multiple images into a Google Drive folder** and click `Import Products`

From there, the system extracts each product's **name and KRW price**, assigns the appropriate **shipping category**, and automatically calculates its **final SGD price** in Google Sheets.

Instead of manually processing products one by one, the user mainly needs to provide the source.

---

## Demo

![Scanning a Weverse Shop Page and Sending the Products to Google Sheets](demo_video.mp4)

*Scanning a Weverse Shop page using Chrome extension. Products are added to the
Google Sheet to calculate final price in SGD.*

---

## How It Works

### 1. Extract the Product Names and Prices

I use two different extraction methods depending on where the product comes from.

For **Weverse Shop**, the Chrome extension reads the product information directly from the webpage and automatically skips sold-out items.

For **images**, such as promotional posters from X or Google Drive, GPT-4.1-mini reads the entire image and returns structured product information, including the product name and KRW price.

This gives the workflow structured data regardless of whether the original source is a webpage or an image.

### 2. Assign a Shipping Category

The product name is then matched to one category from a predefined **Item Weight** list.

The AI is deliberately restricted to this list. It cannot create its own categories.

This matters because the category determines the shipping weight used later in the pricing calculation.

### 3. Calculate the Final SGD Price

Once the category is assigned, Google Sheets looks up its corresponding weight and calculates the final price using:

```text
KRW Price
    ↓
Exchange Rate
    ↓
Shipping
    ↓
Commission
    ↓
Purchase Fee
    ↓
Final SGD Price
```

I intentionally kept this part entirely formula-based rather than asking AI to calculate the price.

Every number can therefore be traced, checked and adjusted directly in the spreadsheet.

### 4. Generate Ready-to-Use Listing Text

Finally, each row automatically generates the text needed for the product listing:

```text
💎<Product Name> 🌟$<Final Price>
```

At this point, the product is ready to be listed for customers.

---

## Challenges & Design Decisions

### 1. OCR Worked on the Text, but Not the Layout

**The challenge**

My first approach was to use Google Drive's built-in OCR.

It could recognise the text, but recognising the text was only half the problem.

Real K-pop merch posters come in all kinds of layouts. Product names can span multiple lines, products can be arranged in different grids, and prices may appear above, below or beside the corresponding item.

As a result, OCR could extract the words correctly but still pair the **wrong price with the wrong product**.

**What I changed**

I switched to GPT-4.1-mini for image-based sources.

Instead of treating the poster as disconnected lines of text, the vision model can interpret the image as a whole and return structured data for each product.

**The trade-off**

Using a vision model introduces a small cost per image.

Because of that, I did not use AI everywhere.

For Weverse Shop, where the product information already exists as structured webpage text, the extension reads it directly. This is both **free and more exact**.

AI is only introduced when the source is an image and understanding its layout actually matters.

---

### 2. Product Names Were Too Inconsistent for Exact Matching

**The challenge**

Originally, shipping weight was looked up using the product name itself.

The problem was that different stores could describe essentially the same type of product in completely different ways:

```text
Plush Keyring
Doll Keychain
Mini Plush Charm
```

An exact `VLOOKUP` could not reliably recognise that these products should use the same shipping weight.

I also tested a keyword-based approach using **27 real product names**, but it correctly matched only **17 out of 27 products (63%)**.

Adding more and more keyword rules would make the system increasingly difficult to maintain.

**What I changed**

Instead of using the raw product name for the lookup, I introduced a standardised category in between.

```text
Product Name
    ↓
AI Classification
    ↓
Standardised Category
    ↓
VLOOKUP
    ↓
Shipping Weight
```

The AI's job is simply:

> **Product name → one category from the Item Weight list**

`VLOOKUP` then handles:

> **Category → shipping weight**

The AI is not allowed to invent a new category, which keeps the output consistent with the weight table I control.

---

## Why Not Let AI Calculate Everything?

I deliberately separated the parts where AI is useful from the parts where traditional logic is more reliable.

**AI handles tasks that require interpretation:**

```text
Reading Images → Understanding Product Names → Assigning Categories
```

**Google Sheets handles tasks that need to be deterministic:**

```text
Weight Lookup → Exchange Rate → Shipping → Fees → Final Price
```

I did not want a system where the final price came from an AI model and I simply had to trust the answer.

By keeping pricing inside spreadsheet formulas, every final price remains **predictable, transparent and easy to audit**.
