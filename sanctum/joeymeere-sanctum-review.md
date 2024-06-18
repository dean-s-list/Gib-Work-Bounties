# Sanctum Feedback Review
**By: Joey Meere**
<br></br>

## General Use
First impressions: the UI very well designed. Simplistic, but with enough detail for me to understand what the product does, where I need to go, and proper highlighting of call-to-actions. None of the pages try too hard to enamor me with complicated
designs and an overuse of motion. The landing page is highly effective in doing more than enough to convery what I can do with this product, while not being heavy on large text blocks. Additionally, it doesn't assault me with wallet connect requests,
which is nice. Immediately I know where I need to go. Upon launching the app, it brings me to the page to mint INF, which initially confused me. At first, I thought this was the page to swap LSTs, which is the route under it. Once I figured that out,
everything else was relatively simple. I was able to find the page to swap, close stake accounts, and view all of the supported LSTs. The "Wonderland" button stuck out due to the rainbow coloring creating sufficient contrast, which drew me to it. Upon
clicking on it, I see a bunch of cute pets, and click on one. This entire section did an effective job of drawing me in, and getting me interested. This led me to mint 0.1 clockSOL and get the "Clockie" pet. The prompt for the specific pet was good about
giving me the notice that this wouldn't be an NFT, and therefore wouldn't see it in my wallet. All-in-all, the design of Sanctum is effective in providing clarity on where I need to be to do what, and subtle suggestions on where I should be going.

I tried to document the user journey from the perspective of someone who's seldom used Sanctum, and the following notes will reflect that.

The journey I followed:
- Arrive on landing page, read each tile, get an understanding of what I'm dealing with
- Click "Launch App"
- Arrive on "Infinity" page, see swap UI
- Connect Phantom Wallet
- Realize that this isn't the correct page to swap LST for LST, go to "Trade" page
- Peruse different pairs available, look for common LSTs I've heard of (bSOL, mSOL, hSOL etc...)
- Search for them directly, and find them successfully
- Conduct a SOL -> bSOL swap, runs successfully
- Navigate to "Stake Accounts"
- See I have a bunch open, close a few of them, done successfully
- Navigate to "LSTs"
- Immediately see LSTs I have seldom heard of
- Look into them more, checking out Twitter and SolanaFM links, eventually minting dainSOL
- Navigate to "Wonderland", which caught my eye from the beginning
- See many cute pets, click on "Clockie" to learn more
- See I need to mint 0.1 of an LST to get it, so I go to "Trade" and do so

Throughout the course of usage, I didn't find myself needing the documentation, which is optimal. It certainly helped to understand what was going on under the hood however, and will be an asset in the next section.

### Notes
- SquadsX is supported :)
- Closing stake accounts one-by-one can be a bit of a chore if there are a lot of them

## Code Review
First impression: the readme gives me a ton of info. This is absolutely amazing. Within 10 seconds, I know what the program is supposed to do, what other programs are involved and how to pull it down + run it with the correct versions. As a developer,
I found the Ideally manifesto to be a very interesting read (out of the scope of this review, but figured I'd mention it). I reviewed the S Controller program, since that's listed first in the readme, and this would be a very comprehensive review if 
I went through every program provided. 

1. Code Quality: Program was sufficiently readable, to the point where I could understand what was going on with a surface level reading. Clear separation of concerns where needed. Breaking things up into specific crates (outlined in Ideally) is great
on this front.
2. Security Review: No obvious security issues found.
3. Performance: Takes far more performance liberties than the average program. Without directly benchmarking, it's safe to say this program is well optimized

### Notes
- I wish Ideally was more well-known. Found it to be very educational, and an interesting take on writing (more) optimized Solana programs with the client in mind.

## Conclusion & Final Comments
From tech to user-experience, the Sanctum team has done a top tier job. Not many better ways to say it in my opinion.
