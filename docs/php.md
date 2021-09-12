---
title: PHP\Laravel Integration for Tigopesa
description: PHP\Laravel package to easy the tigopesa api integration
---

<!-- <p><img src="https://github.com/dbrax/tigopesa-tanzania/blob/master/Tigopesa%20Laravel%20Library.jpeg"></p> -->

# Laravel Tigopesa by [Emmanuel Mnzava](https://github.com/dbrax)

[![Latest Version on Packagist](https://img.shields.io/packagist/v/epmnzava/tigosecure.svg?style=flat-square)](https://packagist.org/packages/epmnzava/tigosecure) [![Total Downloads](https://img.shields.io/packagist/dt/epmnzava/tigosecure.svg?style=flat-square)](https://packagist.org/packages/epmnzava/tigosecure) [![Emmanuel Mnzava](https://img.shields.io/badge/Author-Emmanuel%20Mnzava-green)](mailto:epmnzava@gmail.com)

This package is created to help developers integrate with Tigopesa Tanzania secure online api. More information of this can be found [here](https://github.com/dbrax/tigopesa-tanzania/blob/master/README.md)

## Version Matrix

| Version | Laravel | PHP Version     |
| ------- | ------- | --------------- |
| 1.0.0   | 8.0     | >= 8.0          |
| 1.0.1   | 8.0     | >= 7.3 >= 8.0   |
| 1.0.2   | 8.0     | >= 7.2.5 >= 8.0 |

## Installation

You can install the package via composer:

```bash
composer require epmnzava/tigosecure
```

## Update your config (for Laravel 5.4 and below)

Add the service provider to the providers array in config/app.php:

```php
Epmnzava\Tigosecure\TigosecureServiceProvider::class,
```

Add the facade to the aliases array in config/app.php:

```php
'Tigosecure' =>\Epmnzava\Tigosecure\TigosecureFacade::class,
```

## Publish the package configuration (for Laravel 5.4 and below)

Publish the configuration file and migrations by running the provided console command:

```bash
php artisan vendor:publish --provider="Epmnzava\Tigosecure\TigosecureServiceProvider"
```

### Environmental Variables

- TIGO_CLIENT_ID `your provided tigopesa client id`<br/>
- TIGO_CLIENT_SECRET `your provided tigopesa client secret`<br/>
- TIGO_API_URL `your provided tigopesa api url `<br/>
- TIGO_PIN `your provided tigopesa pin number`<br/>
- TIGO_ACCOUNT_NUMBER `your provided tigopesa account number`<br/>
- TIGO_ACCOUNT_ID `your provided tigopesa account id `<br/>
- TIGO_REDIRECT `your redirect url`<br/>
- TIGO_CALLBACK `your callback url`<br/>
- APP_CURRENCY_CODE `currency put TZS for Tanzanian Shillings`<br/>
- LANG ` language code en for english and sw for swalihi`<br/>

## Usage

This release does not come with database tables for transaction or payments you need to create then After you have filled all necessary variables , providers and facades this is how the package can be used.

On your controller

```php
<?php

namespace App\Http\Controllers;

use Tigosecure;

use Illuminate\Http\Request;
class TransactionController extends Controller
{

    public function customer_transaction(){

        // Tigosecure::make_payment("customerfirstname","customerlastname","customerlastname","amount","transaction_id");
        $tigopesa_response=Tigosecure::make_payment("jacob","laizer","jacob@primeware.co.tz","3000","98778835628");


     return redirect($tigopesa_response->redirectUrl);

    }
```

On your controller you can also call it through this way.

```php
<?php

namespace App\Http\Controllers;

use BillMe;
use Illuminate\Http\Request;
use Epmnzava\LocationDemografia\Models\Country;
use Epmnzava\LocationDemografia\Models\State;
use Epmnzava\Bulksms\Bulksms;
use Epmnzava\Telerivet\Telerivet;
use Epmnzava\Tigosecure\Tigosecure;
use Spatie\SslCertificate\SslCertificate;
use Illuminate\Http\Request;

class TransactionController extends Controller
{

    public function customer_transaction(){

        $payment=new Tigosecure;
        $response=$payment->make_payment("emmanuel","mnzava","epmnzava@gmail.com",4000,"48fhldplofhf".rand(5,100));

        return redirect($response->redirectUrl);
    }
```

### Testing

```bash
composer test
```

### Changelog

Please see [CHANGELOG](https://github.com/dbrax/tigopesa-tanzania/blob/master/CHANGELOG.md) for more information what has changed recently.

## Contributing

Please see [CONTRIBUTING](https://github.com/dbrax/tigopesa-tanzania/blob/master/CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email epmnzava@gmail.com instead of using the issue tracker.

## Credits

- [Emmanuel Mnzava](https://github.com/dbrax)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](https://github.com/dbrax/tigopesa-tanzania/blob/master/LICENSE.md) for more information.
