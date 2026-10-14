# K-pop Merch Price Automation

I'm helping a small Korean proxy shopping service for K-pop fans in Singapore.
Every new merch drop meant hours of copying prices into a spreadsheet by hand,
so I built an automation to do it for them.

## Background

A **Korean proxy shopping service** buys K-pop merchandise in Korea for overseas
fans who can't buy it themselves, then ships it to them.

Working with the service, I've seen fans come to it for four main reasons:

1. **Korean online stores are hard to buy from overseas.** Many need a Korean
   phone number or won't accept foreign cards.
2. **Pop-up stores sell exclusives.** Limited-edition items at Seoul pop-ups are
   often not sold anywhere else.
3. **Fan events are hard to join from abroad.** Ticketing may need a Korean phone
   number, and fansign spots are usually won by buying albums for a lucky draw.
4. **Weverse Shop is hard to pay for and costly to ship.** The official merch store
   doesn't accept foreign cards directly, and paying international shipping on
   every single order adds up fast.

Every price the service quotes a customer is made up of four parts:

| Component | What it covers |
| --- | --- |
| Exchange rate | Converting the KRW price to SGD |
| Shipping | Delivery from Korea to Singapore, based on the item's weight |
| Commission | The service's fee |
| Purchase fee | Handling cost per item |

## The Problem

K-pop drops happen often and sell out fast. To open pre-orders in time, the service
has to list new items quickly, sometimes dozens of products in a single drop.

Before this project, the team did every listing by hand:

1. Read each product name and KRW price off a shop page or a poster on social media
2. Type them into a spreadsheet, one by one
3. Look up each item's shipping weight
4. Work out the final SGD price

It was slow and repetitive, and it was easy to make mistakes. One wrong digit in a
KRW price meant quoting a customer the wrong amount.

## What I Built

Now the team can add products in three ways:

1. **Scan a Weverse Shop page** with a Chrome extension I built
2. **Right-click a picture** on X or another website
3. **Drop pictures into a Google Drive folder** and click *Import Products*

From there, the tool pulls out each product's name and KRW price, works out its
shipping category, and calculates the final SGD price in Google Sheets.

## Demo

<video src="demo_video.mp4" controls width="100%"></video>

If the video doesn't play, [download demo_video.mp4](demo_video.mp4).

## How It Works

1. **Extract names and prices**
   - **Weverse Shop:** the extension reads product text straight from the page
     and skips anything sold out.
   - **Pictures** (right-click or Drive): GPT-4.1-mini reads the image and returns
     each product's name and KRW price.
2. **Assign a category.** The AI matches each product to one category from the
   service's *Item Weight* list. It can only choose from that list, never invent its own.
3. **Calculate the price.** Spreadsheet formulas look up the category's weight,
   then add up the exchange rate, shipping, commission and purchase fee.
4. **Create the listing text.** Each row produces a ready-to-post line like
   `💎<Product Name> 🌟$<Final Price>`.

## Challenges and What I Decided

### 1. OCR couldn't read real merch posters

**What went wrong:** My first version used Google Drive's built-in OCR. It could
read the text, but every poster had a different layout. Long product names were
split across lines, and prices ended up matched to the wrong products.

**What I changed:** I switched to a vision model (GPT-4.1-mini), which looks at
the whole image and returns clean, structured data: name, price and category.

**The trade-off:** Each image costs a little, so I didn't use AI everywhere. For
Weverse Shop, I read the page text directly instead, which is free and accurate.
AI only does the jobs that actually need it.

### 2. Product names never matched the weight list

**What went wrong:** I originally looked up shipping weight with VLOOKUP on the
product name. But every shop names things differently ("Plush Keyring",
"Doll Keychain", "Mini Plush Charm"), so most lookups failed.

**What I changed:** I kept VLOOKUP, but now it looks up a *category* instead of a
product name. The AI turns each product name into one category from the fixed
*Item Weight* list, and VLOOKUP finds that category's weight.

**Why not just use keyword rules?** On 27 real product names, keyword
rules matched only 17 (63%). Limiting the AI to a fixed list gave me consistent
answers, while the weights stay in a table the team controls.

**Why not let the AI calculate prices too?** Prices have to be predictable and
easy to check, so all the maths stays in spreadsheet formulas. The AI only reads
and extracts. It never decides what a customer pays.