# Validate the draw result

A small overview how the algorithmus works can be found at the [README.md](README.MD).

## ticket hash

To reduce the costs of calculation the draw hash, we calculate a md5 hash for each ticket directly after it's creation.

In the hash we're including the deposit transaction id, the sender address, block height, the deposited amount (in DFI), the draw identifier and the created ticket number.

small example how we calculate the md5 hash in PHP:

```
$ticketHash = md5(
    sprintf(
      '%s_%s_%s_%f_%s_%s;',
      $transactionId,
      $senderAddress,
      $blockHeight,
      $amountDeposited,
      $icketNumber,
      $drawIdentifier
    )
  );
```

## finalizing draw

- When finalizing a draw, we concat all ticket hashs ordered by creation block height.
Betweet each hash we add `;` as separator.
The resulting string is hashed again with md5.
- We're calculating the draw seed with the `crc32()`, which calculates the crc32 polynomial of a string and results in an integer.
- With `srand()` the randomizer is now seeded. Everytime we're creating a random number using `rand(0, 999999)`, the result will be the same for the given seed. To fill up the final number to a length of 6 numbers, we're using `str_pad(rand(0, 999999), 6, 0, STR_PAD_LEFT)`

## getting the data via REST API

Choose the environment you want to check the results for:

- mainnet: https://api.defichain-lottery.com
- testnet: https://api.dev.defichain-lottery.com

The path for the verification endpoint is:
`/v1/public/validate/drawing/{drawingIdentifier}`

The drawing identifier can be found on our website.

## demo for verification

To keep it simple for you verifing the result, you can run this script:

```
$drawingId = "********"; // set this value - you find it on https://defichain-lottery.com

/**
 *
 * DONT CHANGE THE CONTENT BELOW!
 *
 **/
$baseUrl = 'https://api.defichain-lottery.com';
$pathUrl = "/v1/public/validate/drawing/";
$requestUrl = sprintf('%s%s%s', $baseUrl, $pathUrl, $drawingId);

$apiResponse = file_get_contents($requestUrl);
$response = json_decode($apiResponse);

$rawString = '';
foreach ($response->tickets as $ticket) {
  $ticketHash = md5(
    sprintf(
      '%s_%s_%s_%f_%s_%s;',
      $ticket->transactionId,
      $ticket->senderAddress,
      $ticket->blockHeight,
      $ticket->amountDeposited,
      $ticket->ticketNumber,
      $response->validationData->drawIdentifier
    )
  );
  if ($ticketHash != $ticket->md5Hash) {
    echo "ERROR ticket hash " . $ticketHash . PHP_EOL . PHP_EOL;
  }
  $rawString .= $ticketHash . ';';
}

$hash = md5(rtrim($rawString, ';')); // md5 hash of all tickets
$IntSeed = crc32($hash);

echo "Hash: " . $hash . PHP_EOL;
echo "Seed: " . $IntSeed . PHP_EOL;

srand($IntSeed); // set the seed to the randomizer

echo "Winning Number: " . str_pad(rand(0, 999999), 6, 0, STR_PAD_LEFT); // this is the final drawing winning number
```