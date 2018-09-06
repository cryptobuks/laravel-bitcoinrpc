# Bitcoin JSON-RPC Service Provider for Laravel

[![Latest Stable Version](https://poser.pugx.org/denpa/laravel-bitcoinrpc/v/stable)](https://packagist.org/packages/denpa/laravel-bitcoinrpc) [![License](https://poser.pugx.org/denpa/laravel-bitcoinrpc/license)](https://packagist.org/packages/denpa/laravel-bitcoinrpc) [![Build Status](https://travis-ci.org/denpamusic/laravel-bitcoinrpc.svg)](https://travis-ci.org/denpamusic/laravel-bitcoinrpc) [![Code Climate](https://codeclimate.com/github/denpamusic/laravel-bitcoinrpc/badges/gpa.svg)](https://codeclimate.com/github/denpamusic/laravel-bitcoinrpc) <a href="https://codeclimate.com/github/denpamusic/laravel-bitcoinrpc/coverage"><img src="https://codeclimate.com/github/denpamusic/laravel-bitcoinrpc/badges/coverage.svg" /></a>

## About
This package allows you to make JSON-RPC calls to Bitcoin Core JSON-RPC server from your laravel project.
It's based on [php-bitcoinrpc](https://github.com/denpamusic/php-bitcoinrpc) project - fully unit-tested Bitcoin JSON-RPC client powered by GuzzleHttp.

## Installation
Run ```php composer.phar require denpa/laravel-bitcoinrpc``` in your project directory or add following lines to composer.json
```json
"require": {
    "denpa/laravel-bitcoinrpc": "^1.1"
}
```
and run ```php composer.phar update```.

Add `Denpa\Bitcoin\Providers\ServiceProvider::class,` line to the providers list somewhere near the bottom of your /config/app.php file.
```php
'providers' => [
    ...
    Denpa\Bitcoin\Providers\ServiceProvider::class,
];
```

Publish config file by running
`php artisan vendor:publish --provider="Denpa\Bitcoin\Providers\ServiceProvider"` in your project directory.

You also might want to add facade to $aliases array in `/config/app.php`.
```php
'aliases' => [
    ...
    'Bitcoind' => Denpa\Bitcoin\Facades\Bitcoind::class,
    'BitcoindFactory' => Denpa\Bitcoin\Facades\BitcoindFactory::class,
];
```

You can use [Environment Configuration](https://laravel.com/docs/master/configuration#environment-configuration) file to configure bitcoind client.
You must have at least following parameters defined:
```
BITCOIND_USER=(rpcuser from bitcoin.conf)
BITCOIND_PASSWORD=(rpcpassword from bitcoin.conf)
```
See `config/bitcoind.php` and [.env.example](https://github.com/denpamusic/laravel-bitcoinrpc/blob/master/.env.example) files to learn more about supported parameters.
You can also directly define your configurations in `config/bitcoind.php`:
```
<?php

return [
    ...
        'bitcoin' => [
            'scheme'   => 'http',
            'host'     => 'localhost',
            'port'     => 8332,
            'user'     => '(rpcuser from bitcoin.conf)',     // required
            'password' => '(rpcpassword from bitcoin.conf)', // required
            'ca'       => null,
        ],

        'bitcoin2' => [
            'scheme'   => 'http',
            'host'     => 'example.com',
            'port'     => 8332,
            'user'     => '(rpcuser at example.com server)',     // required
            'password' => '(rpcpassword at example.com server)', // required
            'ca'       => null,
        ],
    ...
];
```

## Requirements
* PHP 7.0 or higher (should also work on 5.6, but this is unsupported)
* Laravel 5.1 or higher

## Usage
You can perform request to Bitcoin Core using any of methods listed below:
### Helper Function
```php
<?php

namespace App\Http\Controllers;

class BitcoinController extends Controller
{
  /**
   * Get block info.
   *
   * @return object
   */
   public function blockInfo()
   {
      $blockHash = '000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f';
      $blockInfo = bitcoind()->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }
}
```

### Trait
```php
<?php

namespace App\Http\Controllers;

use Denpa\Bitcoin\Traits\Bitcoind;

class BitcoinController extends Controller
{
  use Bitcoind;

  /**
   * Get block info.
   *
   * @return object
   */
   public function blockInfo()
   {
      $blockHash = '000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f';
      $blockInfo = $this->bitcoind()->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }
}
```

### Facade
```php
<?php

namespace App\Http\Controllers;

use Bitcoind;

class BitcoinController extends Controller
{
  /**
   * Get block info.
   *
   * @return object
   */
   public function blockInfo()
   {
      $blockHash = '000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f';
      $blockInfo = Bitcoind::getBlock($blockHash);
      return response()->json($blockInfo->get());
   }
}
```

### Automatic Injection
```php
<?php

namespace App\Http\Controllers;

use Denpa\Bitcoin\Client as BitcoinClient;

class BitcoinController extends Controller
{
  /**
   * Get block info.
   *
   * @param  BitcoinClient  $bitcoind
   * @return \Illuminate\Http\JsonResponse
   */
   public function blockInfo(BitcoinClient $bitcoind)
   {
      $blockHash = '000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f';
      $blockInfo = $bitcoind->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }
}
```

### Multiple Instances
You can use multiple configurations to connect to different bitcoin or even altcoin daemons.
To do this you'll need to open `config/bitcoind.php` file and add new keys containing connection data (see litecoind example in [config file](https://github.com/denpamusic/laravel-bitcoinrpc/blob/multi-instance/config/config.php#L85))
You can then call specific configuration by passing it's name as parameter with usual methods (see examples below)
```php
<?php

namespace App\Http\Controllers;

use Denpa\Bitcoin\ClientFactory;

class CoinController extends Controller
{
  /**
   * Get bitcoin block info using helper.
   * No configuration name passed means using default configuration.
   *
   * @return object
   */
   public function blockInfoUsingHelper()
   {
      $blockHash = '000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f';
      $blockInfo = bitcoind()->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }

  /**
   * Get litecoin block info using helper.
   *
   * @return object
   */
   public function litecoinBlockInfoUsingHelper()
   {
      $blockHash = 'a0c6bf6e1744b30954c41c1269af7b8045d07724333e2f6e3c9a31349d6d3f42';
      $blockInfo = bitcoind('litecoin')->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }

  /**
   * Get litecoin block info using facade.
   * Note BitcoindFactory facade used to get specific client configuration.
   *
   * @return object
   */
   public function litecoinBlockInfoUsingFacade()
   {
      $blockHash = 'a0c6bf6e1744b30954c41c1269af7b8045d07724333e2f6e3c9a31349d6d3f42';
      $litecoind = \BitcoindFactory::get('litecoin');
      $blockInfo = $litecoind->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }

  /**
   * Get litecoin block info using injection.
   * Note ClientFactory hint as we need factory to
   * get specific client configuration.
   *
   * @return object
   */
   public function litecoinBlockInfoUsingInjection(ClientFactory $bitcoindFactory)
   {
      $blockHash = 'a0c6bf6e1744b30954c41c1269af7b8045d07724333e2f6e3c9a31349d6d3f42';
      $litecoind = $bitcoindFactory->get('litecoin');
      $blockInfo = $litecoind->getBlock($blockHash);
      return response()->json($blockInfo->get());
   }
}
```

## License

This product is distributed under MIT license.

## Donations

If you like this project, please consider donating:<br>
**BTC**: 3L6dqSBNgdpZan78KJtzoXEk9DN3sgEQJu<br>
**Bech32**: bc1qyj8v6l70c4mjgq7hujywlg6le09kx09nq8d350

❤Thanks for your support!❤
