# DeFiChain Lottery

We came up with the first Lottery system on DefiChain! The initial version is a centralised solution but once the DefiChain Meta Chain will be released, this project will go completely decentralised.

This is how the initial lottery system will work:

We're offering a deposit address. As soon as you're transferring some DFI to this address, lottery tickets will be generated (e.g. 1 DFI each ticket, 1.5 tickets per dUSD). Overpaid amounts (e.g. you transferred 10.55 DFI, 0.55 DFI overpaid) will be refunded automatically.

A ticket consists of a 6 digit random number where each digit can be between 0-9. Once the lottery drawing ended, the winners will be determined and then the funds will be paid directly to the addresses who own a winning ticket.

## Winner categories
For each number, there's a winner bucket:

- Bucket 1: first number is correct
- Bucket 2: first AND second number are correct
- ...
- Bucket 5: numbers 1..5 are correct
- Bucket 6: all numbers are correct

As you see, the numbers and their order are relevant. The payout amount for each bucket is predefined and will be splittet to all winners in this bucket according to the following distribution.

- Bucket 1: 2% of the pot, winning chance 1:10
- Bucket 2: 3% of the pot, winning chance 1:100
- Bucket 3: 5% of the pot, winning chance 1:1.000
- Bucket 4: 10% of the pot, winning chance 1:10.000
- Bucket 5: 20% of the pot, winning chance 1:100.000
- Bucket 6: 40% of the pot, winning chance 1:1.000.000

## How to buy
The buy process is quite simple. You send DFI OR DUSD to a given address that we own. Every minute we check if there are any new transactions and process them according to the following plan:

- If it is DFI we register the sending address, DFI amount, transaction id into a deposit database. A second process picks up new deposits and generates tickets based on the ticket price and the amount. i.e if ticket price is 1 DFI and you send 4 DFI then you will get 4 tickets. Any reminder will be sent back together with any potential win after the drawing.
- If it is DUSD we swap the DUSD to DFI. The dex stabilisation fee will kick in and burn ~30% of DUSD. We store the sender address, the sent DUSD amount, the amount that has been burned, transaction id, as well as the swapped DFI amount into the database. Because there is a burn happening and we want to incentivise you to send DUSD, we offer a 1.5x on the ticket amount. That is if ticket price is 1 DUSD and you send 2 DUSD you will get 3 tickets. In this case we don't process reminder such as if you send 2.1 DUSD you will not get 0.1 DUSD back.

## Special
Think you already added up the % of the pot mentioned above. Until now, only 80% of the pot are reserved. So what about the rest? To offer some more benefit, 2% of each bought ticket will be donated to the DFI Fund #decentralizedHumanity

8% of each bought ticket will be burned - We can decide if we want to burn DFI or swap to DUSD and burn that instead.

The last 10% is the treasury pot, which is required for marketing and development work.

Notice: **if parts of the pot are not paid out, they will be transferred to the next drawing** WITHOUT the burn/donation/treasury reduction!

## Trust, safety and randomness?

Like with any centralised project the number one key of success is customer trust. I try to give some insights on what steps we take to hopefully increase customer trust.

- Every ticket number is generated using the standard php random_int() function which as per documentation generates cryptographic random integers then left padded to 6 digits. This is so that we can also have a number starting with 0. --> `str_pad(rand(0, 999999), 6, 0, STR_PAD_LEFT)`
- Every ticket we create an MD5 checksum over a concatenated string consisting of trx_id, ticket number, block height, timestamp and sending address, deposit amount. Change of any of the value would result in a different checksum.
- The final number is not known until the draw has completed. This happens with the following steps:
	- The draw is closed
	- A new draw is opened. (New transactions to the address will be registered against the new draw)
	- The final number is generated. Here we take an md5 checksum over all ticket checksums that were generated. We then generate a seed with the `crc32()` function which is then used to generate the random number using `srand()`. What this allows us to do is generate a random final number that basically originates from the ticket generated before. This also allows us to verify that no tempering has happened. Specifically it allows YOU to check, if the number is correct as we are going to publish the ticket data, seed and number via REST API. Anyone can check if the tickets and final number matches!

More details how to proof the result can be found in the [validation tutorial](validation_tutorial.md)!