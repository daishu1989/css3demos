Charting Library consumes your own data so symbology is 100% up to you. You may use whatever naming convention you want. Just return symbol info in Charting Library format and use arbitrary symbols names. Virtually, symbol name may be an arbitrary string containing any characters.

But there are some fine points you should know about:

1. Our own symbology assumes symbols names have `EXCHANGE:SYMBOL` format. Surely Library supports this by default. So if you like it -- just keep calm and use it.
2. Already having other symbology or just going to have one ? There is `ticker` term defined expecially for you. `Ticker` is symbol's unique identifier which is used **only** inside of Library. Your users will never see it. So just put `ticker` values in all of your SymbolInfo and Symbol Search results and expect Charting Library will ask you for data using those values.

## SymbolInfo Structure

**This section is extremely important. 72.2% of all issues experienced by Charting Library users are caused by wrong/malformed SymbolInfo data.**

SymbolInfo is an object containing symbol metadata. This object is the result of resolving the symbol. SymbolInfo has following fields:

##### name
It's name of a symbol. It is a string which your users will see. Also, it will be used for data requests if you are not using **tickers**.

##### ticker
It's an unique identifier for this symbol in your symbology. If you specify this property then its value will be used for all data requests for this symbol. `ticker` is treated to be equal to `symbol` if not specified explicitly.

##### description
Description of a symbol. Will be printed in chart legend for this symbol.

##### session
Trading hours for this symbol. See the [[Trading Sessions|Trading Sessions]] article to know more details.

##### exchange, listed_exchange
For now both of this fields are expected to have short name of exchange where this symbol is traded. This name will be printed in chart's legend for this symbol. This field is not used for other purposes now.

##### timezone
Exchange timezone for this symbol. We expect to get name of time zone in olsondb format. Supported timezones are:
```
UTC
America/New_York
America/Los_Angeles
America/Chicago
America/Toronto
America/Vancouver
America/Argentina/Buenos_Aires
America/Bogota
America/Sao_Paulo
Europe/Moscow
Europe/Athens
Europe/Berlin
Europe/London
Europe/Madrid
Europe/Paris
Europe/Warsaw
Australia/Sydney
Asia/Bangkok
Asia/Tokyo
Asia/Taipei
Asia/Singapore
Asia/Shanghai
Asia/Seoul
Asia/Kolkata
Asia/Hong_Kong
```
##### pricescale, minmov
1. Minimal possible price change is determined by those values.
2. PriceScale parameter determines interval between price lines on chart's price scale.
```
PriceScaleInterval  = MinimalPossiblePriceChange = minmov / pricescale
```

##### fractional <false>
Boolean showing whether this symbol wants to have complex price formatting (see `minmove2` below) or not.

##### minmove2 <0>
It's a magic number for complex price formatting cases. Here are some samples:
```
typical stock with 0.01 price increment: minmov = 1, pricescale = 100, minmove2 = 0
ZBM2014 (T-Bond) with 1/32: minmov = 1, pricescale = 32, minmove2 = 0
ZCM2014 (Corn) with 2/8: minmov = 2, pricescale = 8, minmove2 = 0
ZFM2014 (5 year t-note) with 1/4 of 1/32: minmov=1, pricescale=128, minmove2= 4
```

##### has_intraday <false>
Boolean showing whether symbol has intraday (minutes) history data. If it's false then all buttons for intradays resolutions will be disabled when this symbol is active in chart.
If it is set to true, all resolutions that are supplied directly by the datafeed must be provided in `intraday_multipliers` array.

##### supported_resolutions
An array of resolutions which should be enabled in resolutions picker for this symbol. Each item of an array is expected to be a string.

Resolutions treated as supported by datafeed (see datafeed configuration data) but not supported by the current symbol will be disabled in Resolution picker widget. If one changes the symbol and new symbol does not support the selected resolution then resolution will be switched to first one in supported resolutions list. Resolution availability logic (pseudocode):
```
resolutionAvailable  = 
    resolution.isIntraday ?
       symbol.has_intraday && symbol.supports_resoluiton(resolution) :
    symbol.supports_resoluiton(resolution);
```

In case of absence of `supported_resolutions` in a symbol info all DWM resolutions will be available. Intraday resolutions will be available if `has_intraday` is `true`.

Supported resolutions affect available time frames too. The timeframe will not be available if it requires the resolution which is not supported.

##### intraday_multipliers <[]>
It is an array containing intraday resolutions (in minutes) the datafeed wants to build by itself. E.g., if the datafeed reported he supports resolutions ["1", "5", "15"], but in fact it has only 1 minute bars for symbol X, it should set intraday_multipliers of X = [1]. This will make Charting Library to build 5 and 15 resolutions by itself.

##### has_seconds <false>
Boolean showing whether symbol has sedonds history data. If it's false then all buttons for seconds resolutions will be disabled when this symbol is active in chart.
If it is set to true, all resolutions that are supplied directly by the datafeed must be provided in `seconds_multipliers` array.

##### seconds_multipliers <[]>
It is an array containing seconds resolutions (in seconds without a postfix) the datafeed wants to build by itself. E.g., if the datafeed reported he supports resolutions ["1S", "5S", "15S"], but in fact it has only 1 second bars for symbol X, it should set seconds_multipliers of X = [1]. This will make Charting Library to build 5S and 15S resolutions by itself.

##### has_daily <false>
The boolean value showing whether datafeed has its own D resolution bars or not. If `has_daily` = false then Charting Library will build respective resolutions from intraday by itself. If not, then it will request those bars from datafeed.

##### has_weekly_and_monthly <false>
The boolean value showing whether datafeed has its own W and M resolution bars or not. If has_weekly_and_monthly = false then Charting Library will build respective resolutions from D by itself. If not, then it will request those bars from datafeed.
 
##### has_empty_bars <false>
The boolean value showing whether library should generate empty bars in session when there is no data from datafeed for this time. I.e., if your session is `0900-1600` and your real data lacks of trades between `11:00` and `12:00` and your `has_empty_bars` is true, then Library will paste degenerate bars in this time.

##### force_session_rebuild <true>
The boolean value showing whether library should filter bars with current session. If false, bars will filtered when library builds data from other resolution or if has_empty_bars was set to true. If true, Library will remove from your data those bars who does not belong to trading session.

##### has_no_volume <false>
Boolean showing whether symbol has volume data or not.

##### has_fractional_volume <false> | obsolete (1.1 - 1.5), use volume_precision instead
If has_fractional_volume=true, `Volume` indicator values will not be rounded to integer values.

##### volume_precision <0>
Integer showing typical volume value decimal places for this symbol. 0 means volume always in an integer. 1 means there may be 1 numeric character after comma and so on.

##### data_status
The status code of a series with this symbol. The status is shown in upper right corner of a chart. Supported values:
* streaming
* endofday
* pulsed
* delayed_streaming

##### expired <false>
Boolean showing whether this symbol is expired futures contract or not.

##### expiration_date
Unix timestamp of expiration date. One must set this value if `expired` = true. Charting Library will request data for this symbol starting from that time point instead of actual moment.

##### sector
Sector for stocks to be displayed in Symbol Info.

##### industry
Industry for stocks to be displayed in Symbol Info.