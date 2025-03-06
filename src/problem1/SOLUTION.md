# Problem 1: Too Many Things To Do

## Objective

Submit HTTP GET requests to `https://example.com/api/:order_id` with Order IDs that are selling `TSLA`, writing the output to `./output.txt`

## Input File

The input file `./transaction-log.txt` contains JSON records of trading transactions, one per line.

## Solution

```bash
grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt | jq -r '.order_id' | xargs -I{} curl -s https://example.com/api/{} > ./output.txt
```


## Explanation

This command works in four steps:

1. **Filter TSLA sell orders**:

```bash
grep -E '"symbol": "TSLA".*"side": "sell"' ./transaction-log.txt
```

Searches the transaction log file for lines containing both "TSLA" as the symbol and "sell" as the side.
2. **Extract order IDs**:

```bash
jq -r '.order_id'
```

Parses the JSON from each matching line and extracts just the order_id value.
3. **Make API requests**:

```bash
xargs -I{} curl -s https://example.com/api/{}
```

Takes each order ID and makes a GET request to the API endpoint, substituting the order ID in the URL.
4. **Save output**:

```bash
> ./output.txt
```

Redirects all API responses to the specified output file.

## Expected Results

The command will identify orders with IDs 12346 and 12362 (the two TSLA sell orders in the log) and make API requests for them, saving the responses to output.txt.

## Requirements

- grep (with regex support)
- jq (JSON processor)
- curl (HTTP client)
- xargs (command builder)

