# Validate the draw result

A small overview about the project and how the validation algorithmus works can be found at the [README.md](README.MD).

## ticket hash

To reduce the costs of calculation the draw hash, a md5 hash is calculated for each ticket directly after it's creation.

In the hash includes the deposit transaction id, the sender address, block height, the deposited amount (in DFI), the draw identifier and the random ticket number.

md5 hash calculation (in PHP):

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

- When finalizing a draw, all ticket hashes are concatenated ordered by creation block height.
Betweet each hash the separator `;` is added.
The resulting string is hashed again with md5.
- Calculating the draw seed with the `crc32()`, which calculates the crc32 polynomial of a string and results in an integer.
- With `srand()` the randomizer is seeded. Everytime creating a random number using `rand(0, 999999)`, the result will be the same for the given seed. To fill up the final number to a length of 6 numbers, the function `str_pad(rand(0, 999999), 6, 0, STR_PAD_LEFT)` is used.

## getting the data via REST API

Choose the environment you want to check the results for:

- mainnet: https://api.defichain-lottery.com
- testnet: https://api.dev.defichain-lottery.com

The path for the verification endpoint is:
`/v1/public/validate/drawing/{drawingIdentifier}`

The drawing identifier can be found on our website.

## demo for verification

To keep it simple for you verifing the result, you can run this script (PHP):

```
$drawingId = "********"; // set this value - you find it on https://defichain-lottery.com
$runningOnMainnet = false;

/**
 *
 * DONT CHANGE THE CONTENT BELOW!
 *
 **/
function validateDrawing(string $drawingId, bool $isMainnet)
{
  $baseUrlMainnet = 'https://api.defichain-lottery.com';
  $baseUrlTestnet = 'https://api.dev.defichain-lottery.com';
  $pathUrl = "/v1/public/validate/drawing/";
  $requestUrl = sprintf(
    '%s%s%s',
    $isMainnet ? $baseUrlMainnet : $baseUrlTestnet,
    $pathUrl,
    $drawingId
  );

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

  echo "Drawing: " . $drawingId . PHP_EOL;
  echo "Winning Number: " . str_pad(rand(0, 999999), 6, 0, STR_PAD_LEFT); // this is the final drawing winning number
}

validateDrawing($drawingId, $runningOnMainnet);
```

If there are less tickets bought for a draw, e.g. the tool [paiza.io](https://paiza.io/en/projects/new?language=php) can be used to run this snippet. Otherwise you will see a timeout. In this case you need to run this code locally.
