As an end host of 0x Instant, you can charge users a fee on all trades made through Instant with the `affiliateFee` option. Simply specify an ethereum address and feePercentage (up to 5%), and a percentage of each transaction will be deposited into the specified address (denominated in ETH).

Example: Earning affiliate fees

3% of transaction volume (in ETH) will de deposited into 0x50ff5828a216170cf224389f1c5b0301a5d0a230

```html
<head>
    ...
    <script src="http://0x-instant-staging.s3-website-us-east-1.amazonaws.com/main.bundle.js"></script>
    ...
</head>
```

```javascript
zeroExInstant.render(
    {
        orderSource: 'https://api.relayer.com/sra/v2/',
        affiliateInfo: {
            feeRecipient: '0x50ff5828a216170cf224389f1c5b0301a5d0a230',
            feePercentage: 0.03,
        },
    },
    'body',
);
```
