Deploy the current Stylus contract to Arbitrum Sepolia testnet:

1. First run `/check-contract` to ensure everything passes
2. Check that $PRIVATE_KEY is set in .env
3. Run: `cargo stylus deploy --private-key $PRIVATE_KEY --endpoint https://sepolia-rollup.arbitrum.io/rpc`
4. Capture the contract address and transaction hash
5. Verify: `cargo stylus verify --deployment-tx $TX_HASH --endpoint https://sepolia-rollup.arbitrum.io/rpc`
6. Show link to Sepolia Explorer: https://sepolia.arbiscan.io/address/$CONTRACT_ADDRESS

Report deployment summary with all addresses and hashes.
