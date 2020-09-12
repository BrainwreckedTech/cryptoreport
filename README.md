# Cryptoreport

A BASH script to track and report on your crypto currency holdings.

## Supported Exchanges

| Exchange                | API | CSV | TSV Only |
|-------------------------|:---:|:---:|:--------:|
| Binance_US              |  ✔  |  ✖  |          |
| Binance (International) |  ✔  |  ✖  |          |
| BlockFi                 |  ✖  |  ✖  |    ✔     |
| Celsius                 |  ✖  |  ✔  |          |
| Coinbase                |  ✔  |  ✖  |          |
| Coinbase_Pro            |  ✔  |  ✖  |          |
| Kraken                  |  ✔¹ |  ✖  |          |
| KuCoin                  |  ✔  |  ✖  |          |
| StakeCube               |  ✖  |  ✖  |    ✔     |

***Any*** exchange that is not supported by API or CSV may be tracked manually.  Exchanges listed here as TSV-Only are known to have neither CSV export nor API access.  

¹ Kraken ticker symbols aren't fully supported yet.  Kraken has a habit of putting an X in front of the ticker symbol.  XXBT, XETH, and XXLM are already corrected back to the standard ticker symbols.  You can open an issue to add corrections for any coins that need it.

## Configuration and Data Storage Locations

For now, `cryptoreport` stores all configs and data in its own directory.  Because of this, you'll most likely want to put the script into a sub-directory of your home directory.

## Configuration

Configuration is handled with the `cryptoreport.json` file.

### General Reference

```JSON
{
  "Exchange_Name": {
    "API_Key": "Your_API_Key",
    "API_Secret": "Your_Secret/Private_Key",
    "API_Password": "Your_API_Password",
    "CSV_File": true/false,
    "TSV_File": true/false,
    "Type": "Data/Exchange",
    "Rewards": true/false,
    "Value_Source": "Exchange_Name"
  }
}
```
- **API_Key, API_Secret, API_Password**  
  When using API access, set the relevant JSON keys.  Relevant for both `Data` and `Exchange` types.

- **CSV_File**  
  When `Type`=`Exchange` and you are using exported CSV files from the exchange, set to `true`.

- **TSV_File**  
  When `Type`=`Exchange` and you are tracking transactions manually, set to `true`.

- **Type**  
  Either `Data` for Data APIs or `Exchange` for exchanges.

- **Rewards**  
  When `Type`=`Exchange` and you are receiving rewards (interest, etc.), set to `true`.

- **Value_Source**  
  When `Type`=`Exchange` and you don't have API access, set to `Exchange_Name` to get coin values from there.

### Examples

#### Set up the CoinMarketCap Data API

*Requires registration.  Registration is cost-free as long as you don't plan on making more than 10k calls per month.*  It's highly recommended to set this up over at https://pro.coinmarketcap.com/login as this is used for both the `convert` command and as a fallback for coin values when API calls to native exchanges fail.

```JSON
{
  "CoinMarketCap": {
  "API_Key": "Your API Key",
  "Type": "Data"
  }
}
```

#### Exchange with API access

```JSON
{
  "Coinbase_Pro": {
  "API_Key": "Your API Key",
  "API_Secret": "Your Secret Key",
  "API_Password": "Your Password",
  "Type": "Exchange",
  "Rewards": false
  }
}
```

#### Exchange with CSV Export

```json
{
  "Celsius": {
  "CSV_File": true,
  "Type": "Exchange",
  "Rewards": true,
  "Value_Source": "Coinbase_Pro"
  }
}
```

#### Manually-Tracked Exchange

```JSON
{
  "StakeCube": {
  "TSV_File": true,
  "Type": "Exchange",
  "Rewards": true,
  "Value_Source": "Binance"
  }
}
```

## Usage

Once you have stored your configuration in `cryptoreport.json` you can start using `cryptoreport`

- `cryptoreport import <exchange> <url>`  
  Imports a CSV file from `<exchange>`

- `cryptoreport add <exchange> <currency> <trans_type> <amount> <date> <trans_id>`  
  Adds a transaction to a manually-tracked `<exchange>`

- `cryptoreport update`  
  Gathers up all of your coin balances

- `cryptoreport value <grouping> [format:csv]`
  Generates a report on the value of your holdings.  The report can be grouped by `detail`, `coins`, or `exchange`.  The optional `format:csv` option outputs a comma-delimited list rather than a console-pretty table.

- `cryptoreport convert <value-from> <currency-from> <value-to> <currency-to>`  
  Convert values between currencies.

- `cryptoreport rewards <time_range>`  
  Report on rewards earned during `<time_range>`, which can be one of `today`, `yesterday`, `day-before`, `this-month`, `last-month`, `this-year`, `last-year`, or a specific date in `YYYY-MM-DD` format.

- `cryptoreport test-api <exchange> <endpoint> [query_string]`
  Have fun poking around the APIs yourself.

Just remember that `help` is your friend.

## Virtual Exchanges

Exchanges may be split into virtual exchanges depending on how they are set up.

### KuCoin

KuCoin has a lot of separation of balances.  For their main site (kucoin.com) you have a Main account balance and a Trade account balance.  These funds are indeed stored separately as the API returns two sets of account balances.  Then you have yet another balance on their staking site (pool-x.io).  These balances are also stored separately and are returned as a third set of balances in the API.

But the last balance on KuCoin -- the coins you have locked up in Fixed Deposit staking -- are nowhere to be found via standard API calls.  Instead, this balance must be calculated from the ledgers of your staking balance where you put crypto into and take crypto out of locks.

So there are four virtual exchanges when you set up KuCoin API access.

**KuCoin_Main:**  Your main account balance  
**KuCoin_Trade:**  Your trade account balance  
**KuCoin_Pool:**  Your Pool-X account balance  
**KuCoin_Lock:**  The balance you have locked up on Pool-X

***Warning:*** KuCoin_Trade is not set up yet.  As a result, any balance you have in your trading account will be ignored.  Until this is rectified, please keep your coins out of the trading account until needed for trade so they can be counted.

### Kraken

Kraken separates staking and trading balances as noted on their main web site.  Both balances are returned by the API with staking balances denoted with a ".S" at the end of the ticker symbol.  (E.g., DOT is the balance in the trade account while DOT.S is the balance in the staking account.)

**Kraken_Stake:** Your staking balance  
**Karken_Trade:** Your trading balance

## Design Decisions

TSV (tab-separated values) is the internal format of `cryptoreport`'s operation.

Capitalization doesn't matter, except when it does.  :joy:  The only time it matters is inside the JSON configuration file.  Internal operations are case-insensitive, but `cryptoreport` will maintain the capitalization of exchange names you place inside of the JSON configuration file.