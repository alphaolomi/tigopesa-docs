---
title: Python Integration for Tigopesa
description: |
  Python package to easy the tigopesa api integration
---

# [tigopesa](https://kalebu.github.io/tigopesa) python package by - [Kalebu Jordan](https://kalebu.github.io/)

[![Downloads](https://pepy.tech/badge/tigopesa)](https://pepy.tech/project/tigopesa) [![Downloads](https://pepy.tech/badge/tigopesa/month)](https://pepy.tech/project/tigopesa) [![Downloads](https://pepy.tech/badge/tigopesa/week)](https://pepy.tech/project/tigopesa)

<!-- ![Become a Patron](pictures/become_a_patron_button.png) -->

## Getting started

To get started with Tigopesa, you firstly need to install it as show below;

```bash
pip install tigopesa
```

## Authorization and Configuration

Before you begin making transaction with tigopesa module, you firstly need to initialize your tigopesa api credentials _client_id_ and _client_secret_ you were given.

Whether are sandbox keys or production keys, all of them are in great use with the tigopesa package

Here how to initialize;

```python
from tigopesa import Tigopesa
tigopesa = Tigopesa(
            client_secret='xxxx',
            client_id ='xxxx'
            environment="sandbox"
        )
```

# OR

# You can do this;

```python
from tigopesa import Tigopesa
tigopesa = Tigopesa(environment='production')
tigopesa.client_id = 'xxxx'
tigopesa.client_secret = "xxxx'
```

Once you initialize your module, you might need still need to configure your module with couple of more information ready to begin making transactions, there are required parameters and optional parameters while configuring as shown below;

```python
# Master merchant (Required parameters)

account: str
pin: str
account_id: str

# Merchant Information

mechant_reference: Optional[str] = ''
mechant_fee: Optional[str] = '0.0'
mechant_currency_code: Optional[str] = ''

# Other_information
language: Optional[str] = 'eng'
terminal_id: Optional[str] = ''
currency_code: Optional[str] = 'TZS'

tax: Optional[str] = '0.0'
fee: Optional[str] = '0.0'

exchange_rate: Optional[str] = '1'

# Callbacks and Redirects

callback_url: Optional[str] = 'https://kalebujordan.dev/'
redirect_url: Optional[str] = 'https://kalebu.github.io/pypesa/'

# Subscribers default Information

subscriber_country_code: Optional[str] = '255'
subscriber_country: Optional[str] = 'TZA'
```

As you can see there about 3 required parameters while the rest being optional parameters, so in our example we are going to configure using only 3 required parameters and the rest will just take the default values;

```python
from tigopesa import Tigopesa
tigopesa.configure(
            account = '255xxxxx',
            pin = 'xxxxx'
            account_id = 'xxxxxx'
            # .........
        )
```

## Authorizing Payments

Now once we are done with the authentication and the authorization part, we can start making authorizing tigopesa payment, Here an example you would authorize a secure tigopesa payment with tigopesa library;

```python
response = tigopesa.authorize_payment({
            "amount": 4999,
            "first_name": "Kalebu",
            "last_name": "Gwalugano",
            "customer_email": "kalebjordan.kj@gmail.com",
            "mobile": "255757294146",}
        )

print(response)
```

```python
# Response output

{'transactionRefId': 'f9995a1ab5d04235a2aeeef37baad129', 'redirectUrl': 'https://secure.tigo.com/v1/tigo/payment-auth/transactions?auth_code=CgFsXfSZRL&transaction_ref_id=f9995a1ab5d04235a2aeeef37baad129&lang=eng', 'authCode': 'CgFsXfSZRL', 'creationDateTime': 'Sat, 1 May 2021 20:50:34 UTC', 'SessionLife': 600}
```

## Issues

If you're facing issue with the use of the package, raise an issue and I will be working to fixing it as soon as I can;

## Contributing

Tigopesa has a lot of modules to integrate, it can be overwhelming doing all of them by myself together with other responsibilities so I warmly welcome contributors (code + documentation) to contribute to this package.

## Credits

- [Kalebu Jordan](https://kalebu.github.io/)
- [Contributors](https://kalebu.github.io/tigopesa/graphs/contributors)
