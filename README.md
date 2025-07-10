# About the Parsers

These are **two distinct parsers** designed for FortiSIEM to process log
messages related to Electric Vehicle Charging Stations (EVCS) and the Open
Charge Point Protocol (OCPP).

- `FortiGateParser-EVCS.xml`
- `FortiSIEM_7.4.0.0430_FortiGateParser-OCPP.xml`

## Overview

- **EVCS "generic" Parser**:
  - Is triggered as long as the log event contains app_cat=Industrial.
  - Typically remaps OCPP attributes directly to native FortiSIEM event attributes.

- **New "OCPP" Parser**:  
  - Explicitly targets OCPP-related messages (application name must start with `OCPP`).
  - Relies on a set of **additional event attributes** that must be created in
    advance within FortiSIEM. These attributes start with the prefix `ocpp`
    for better visualisation.
  - Instead of remapping OCPP attributes to existing FortiSIEM fields, this
    parser extracts and assigns OCPP data to these pre-defined custom
    attributes.
  - Relies on JSON key/value parsing.

## How the Parser Works

- Specifically processes OCPP messages observed in FortiGate firewall logs,
  especially those captured by UTM application control.
- Extracts key OCPP elements such as `idTag`, `connectorId`, `status`, and
  `errorCode` from the payload for enhanced visibility and mapping.
- Extraction rules are based on conditions and regular expressions applied to
  the `$fileName` field, which typically contains the OCPP payload.
- The parsers are modulars and can be extended to support additional OCPP
  operations as needed.
- The parsers logic is located between the following tags:

  - `FortiGateParser-EVCS.xml` in section `<parsingInstructions/>`
  
    ```xml
    [...]
    <parsingInstructions>
    [...]
    <!--EVCS System Parser-->
    [...]
    <!--EVCS System Parser-->
    [...]
    </parsingInstructions>
    ```

  - `FortiSIEM_7.4.0.0430_FortiGateParser-OCPP.xml` in section
    `<patternDefinitions/>` and `<parsingInstructions/>`.

    ```xml
    <patternDefintions>
    [...]
    <!-- OCPP Parser -->
    [...]
    <!-- OCPP Parser -->
    [...]
    </patternDefintions>
    [...]
    <parsingInstructions>
    [...]
    <!-- OCPP Parser-->
    [...]
    <!-- OCPP Parser-->
    [...]
    </parsingInstructions>
    ```

## Test Events for the Parsers

Adding test events is a mandatory step when uploading a new parser to
FortiSIEM. You can use the following sample events to test and validate the
parsers.

These logs reflect typical OCPP operations observed in EVCS environments and
are compatible with the current parser setup. We recommend including them in
the test section of your parser definition.

The following three events are saved in the file testEvents. This file can be
imported directly. FortiSIEM will automatically separate the file into
individual events, creating one event for each line in the file.

### Test Event 1: OCPP Authorize Request

```text
<190>date=2025-06-26 time=17:00:09 devname="EVCS-FGT" devid="FGXXXXXXXXXXXXX" eventtime=1750946409105229609 tz="+0300" logid="1059028704" type="utm" subtype="app-ctrl" eventtype="signature" level="information" vd="root" appid=52334 srcip=10.0.10.12 srccountry="Reserved" dstip=10.0.20.10 dstcountry="Reserved" srcport=53532 dstport=8180 srcintf="vlan10" srcintfrole="lan" dstintf="IPSEC1" dstintfrole="undefined" proto=6 service="tcp/8180" direction="outgoing" policyid=1 poluuid="79a91dc8-285a-51f0-4cb4-6fad2ac8b80a" policytype="policy" sessionid=76503 applist="AC-all" action="pass" appcat="Operational.Technology" app="OCPP_Authorize" hostname="10.0.20.10" incidentserialno=203593557 msg="Operational.Technology: OCPP_Authorize" clouduser="10.0.10.12" filename="[2,\"1142054\",\"Authorize\",{\"idTag\":\"tagid002\"}]" apprisk="elevated"
```

### Test Event 2: OCPP StartTransaction Request

```text
<190>date=2025-06-26 time=17:00:58 devname="EVCS-FGT" devid="FGXXXXXXXXXXXXX" eventtime=1750946457648432203 tz="+0300" logid="1059028704" type="utm" subtype="app-ctrl" eventtype="signature" level="information" vd="root" appid=52398 srcip=10.0.10.12 srccountry="Reserved" dstip=10.0.20.10 dstcountry="Reserved" srcport=53532 dstport=8180 srcintf="vlan10" srcintfrole="lan" dstintf="IPSEC1" dstintfrole="undefined" proto=6 service="tcp/8180" direction="outgoing" policyid=1 poluuid="79a91dc8-285a-51f0-4cb4-6fad2ac8b80a" policytype="policy" sessionid=76503 applist="AC-all" action="pass" appcat="Operational.Technology" app="OCPP_Start.Transaction" hostname="10.0.20.10" incidentserialno=203593590 msg="Operational.Technology: OCPP_Start.Transaction" clouduser="10.0.10.12" filename="[2,\"1142075\",\"StartTransaction\",{\"connectorId\":1,\"idTag\":\"tagid004\",\"meterStart\":328034,\"timestamp\":\"2025-06-26T14:00:57.483Z\"}]" apprisk="elevated"
```

### Test Event 3: OCPP StopTransaction Request

```text
<190>date=2025-06-26 time=17:01:56 devname="EVCS-FGT" devid="FGXXXXXXXXXXXXX" eventtime=1750946515730257781 tz="+0300" logid="1059028704" type="utm" subtype="app-ctrl" eventtype="signature" level="information" vd="root" appid=52400 srcip=10.0.10.11 srccountry="Reserved" dstip=10.0.20.10 dstcountry="Reserved" srcport=53530 dstport=8180 srcintf="vlan10" srcintfrole="lan" dstintf="IPSEC1" dstintfrole="undefined" proto=6 service="tcp/8180" direction="outgoing" policyid=1 poluuid="79a91dc8-285a-51f0-4cb4-6fad2ac8b80a" policytype="policy" sessionid=76502 applist="AC-all" action="pass" appcat="Operational.Technology" app="OCPP_Stop.Transaction" hostname="10.0.20.10" incidentserialno=203593627 msg="Operational.Technology: OCPP_Stop.Transaction" clouduser="10.0.10.11" filename="[2,\"1085790\",\"StopTransaction\",{\"meterStop\":314394,\"timestamp\":\"2025-06-26T14:01:55.661Z\",\"transactionId\":52533,\"reason\":\"EVDisconnected\"}]" apprisk="elevated"
```

### How to Install the Parsers

1. **Disable** the native parser `FortiGateParser`.
2. **Clone** the parser.
3. **Copy and paste** one of the parser files into your cloned version.

If you are using `FortiSIEM_7.4.0.0430_FortiGateParser-OCPP.xml`, you must create the required FortiSIEM Event Attributes in advance.

Refer to the `EventAttributes.csv` file for a list of the event attributes
that need to be created.
