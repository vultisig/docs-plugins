# Vultisig Plugin Blockchain Integration

This guide explains how Vultisig plugins connect to blockchain networks for transaction signing and execution.

## Vultisig-Supported Networks

Vultisig plugins can interact with:

- Ethereum Mainnet (chain ID: 1)
- Ethereum Testnets (Goerli, Sepolia)
- EVM-compatible chains (Polygon, Arbitrum, Optimism)

## Plugin Policy Configuration

Your plugin should accept these blockchain parameters in the policy JSON:

```go
type MyPluginPolicy struct {
    ChainID            string  `json:"chain_id"`
    RpcURL             string  `json:"rpc_url"`
    TokenAddress       string  `json:"token_address"`
    RecipientAddress   string  `json:"recipient_address"`
    AmountWei          string  `json:"amount_wei"`
    GasLimit           uint64  `json:"gas_limit"`
}
```

## Vultisig Transaction Flow

1. **Policy Validation**: Validate blockchain parameters in `ValidatePluginPolicy`
2. **Transaction Preparation**: Create unsigned transaction in `ProposeTransactions`
3. **Transaction Validation**: Verify proposed transactions in `ValidateProposedTransactions`
4. **Signature Application**: Apply TSS signatures in `SigningComplete`
5. **Transaction Broadcast**: Send signed transactions to the blockchain

## Example: Preparing Transactions for Vultisig Signing

```go
func (p *MyPlugin) ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error) {
    var myPolicy MyPluginPolicy
    if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
        return nil, err
    }
    
    // Create transaction data for signing
    txData := createTransactionData(myPolicy)
    
    return []types.PluginKeysignRequest{
        {Transaction: txData, Messages: []string{txHash}},
    }, nil
}
```

## Vultisig-Specific Considerations

- Store `ChainID` in policy for signature verification
- Use the blockchain data from policy for transaction creation
- Calculate transaction hashes correctly for TSS signing
- Apply TSS signatures to transactions in the correct format
- Verify transaction success in `SigningComplete`