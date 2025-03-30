+++
title = "Query Financial Data Conveniently with vistafetch"
date = "2023-10-20"

toc = true

[taxonomies]
tags=[
    "open-source",
    "python",
]
categories = ["Tech"]
+++

{% note(header="TL;DR") %}
`vistafetch` is a small and simple Python library to fetch financial data for stocks, ETFs, funds, etc. from Onvista.
{% end %}

Within this blog post, I'd like to introduce [`vistafetch`](https://github.com/bossenti/vistafetch): a small and simple
library to fetch financial data for stocks,
ETFs, funds, etc. from Onvista. `vistafetch` makes retrieving data for financial data a blast by providing an easy and
simple-to-remember syntax. It is available on [PyPI](https://pypi.org/project/vistafetch/) and can therefore be easily
installed by using pip:

```bash
pip install vistafetch
```

## Discover Financial Assets 🔎

The first step to work with `vistafetch` is to create an instance of the `VistaFetchClient`:

```python
from vistafetch import VistaFetchClient

client = VistaFetchClient()
```

This client enables you to search for various financial assets on [Onvista](https://www.onvista.de/). You can simply use
the search for keywords,
like 'Apple':

```python
result = client.search_asset(
    search_term="Apple",
)
result.visualize()
```

This visualizes the found items as a tabular structure:

```bash
┏━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━┳━━━━━━━━━━━━━━┓
┃ Index ┃              Name               ┃ Asset Type ┃     ISIN     ┃
┡━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━╇━━━━━━━━━━━━━━┩
│   0   │              Apple              │   STOCK    │ US0378331005 │
│   1   │     Apple Hospitality REIT      │   STOCK    │ US03784Y2000 │
│   2   │  Apple International Co. Ltd.   │   STOCK    │ JP3121170009 │
│   3   │ Apple Inc. DL-Notes 2021(21/61) │    BOND    │ US037833EL06 │
│   4   │ Apple Inc. DL-Notes 2023(23/53) │    BOND    │ US037833EW60 │
└───────┴─────────────────────────────────┴────────────┴──────────────┘
```

If you want to continue working with one of these assets you simply select your desired financial asset by its index
shown in the table:

```python
apple_reit = result.get(1)
```

As an alternative, you can also directly search with identifiers, e.g., the ISIN or WKN:

```python
result = client.search_asset(
    search_term="IE00B3XXRP09",
)
vanguard_etf = result.get()
```

Since we pass a unique identifier, we can be sure to get only one result. For this case, we can simply get the
corresponding financial asset by using `get()`.

<br>

A financial asset has the following metadata available:

* `entity_type`: The type of the financial asset, e.g., stock or bond
* `isin`: The [ISIN](https://en.wikipedia.org/wiki/International_Securities_Identification_Number) of the financial
  asset
* `name`: The full name of the financial asset
* `tiny_name`: The short name of the financial asset
* `wkn`: The [WKN](https://en.wikipedia.org/wiki/Wertpapierkennnummer) of the financial asset

All attributes can directly be accessed on the financial asset, e.g., `vanguard_etf.wkn`.
The full financial asset record looks the following:

```python
print(vanguard_etf.as_json())
# or alternatively as dictionary
vanguard_etf.model_dump()
```

```bash
{
  "display_type":"ETF",
  "entity_type":"FUND",
  "isin":"IE00B3XXRP09",
  "name":"Vanguard S&P 500 UCITS ETF USD Dis.",
  "tiny_name":"Vanguard S&P 500 UCITS ETF USD Dis.",
  "wkn":"A1JX53",
  "market":null
}
```

## Retrieve Price Data 🤑

`vistafetch` provides two different ways to retrieve price data for a financial asset.
The first one simply returns the most recent available price data. This can be queried by running:

```python
price_data = vanguard_etf.get_latest_price_data()
```

This returns an instance of PriceData which encompasses several price-related values and their corresponding timestamp.
The full structure looks the following (via `price_data.as_json()`):

```bash
{
  "currency_symbol":"EUR",
  "datetime_high":"2023-10-19T14:00:30.131000Z",
  "datetime_last":"2023-10-19T19:15:10.383000Z",
  "datetime_low":"2023-10-19T18:51:26.540000Z",
  "datetime_open":"2023-10-19T06:00:01.227000Z",
  "high":77.692,
  "last":76.668,
  "low":76.648,
  "open":77.48
}
```

Every value can directly be accessed as attribute on `price_data`, e.g., `price_data.open`.
Second, we can also query the price data for a specific date:

```python
> vanguard_etf.get_day_price_data(day=datetime.date(2023, 10, 10)).last

77.891
```

## Future Roadmap

In next future, more time intervals are planned to be supported for price data retrieval so that data can also be
queried for a month, a year, etc.

If you have any ideas or wishes beyond that, please feel to create
a [feature request](https://github.com/bossenti/vistafetch/issues/new).

---

## Problems? 🐛

Feel free to open an [issue](https://github.com/bossenti/vistafetch/issues/new) if you experience strange behavior or bugs when using `vistafetch`. If you are not sure if your
problem should be considered a bug or if you have a question in general, reach out via [discussions](https://github.com/bossenti/vistafetch/discussions).