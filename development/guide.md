# Vultisig Plugin Development Guide

## Plugin Interface

Every Vultisig plugin must implement these methods:

```go
type Plugin interface {
    FrontendSchema() embed.FS
    ValidatePluginPolicy(policy vtypes.PluginPolicy) error
    ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error)
    ValidateProposedTransactions(policy vtypes.PluginPolicy, txs []types.PluginKeysignRequest) error
    SigningComplete(ctx context.Context, signature tss.KeysignResponse, 
                   signRequest types.PluginKeysignRequest, policy vtypes.PluginPolicy) error
}
```

## Development Workflow

1. **Define your policy structure**: Create a struct with configuration fields your plugin needs.

2. **Implement ValidatePluginPolicy**: Validate all policy parameters for correctness.

3. **Implement ProposeTransactions**: Create unsigned transaction(s) based on policy rules.

4. **Implement ValidateProposedTransactions**: Verify that proposed transactions match policy intent.

5. **Implement SigningComplete**: Handle the signed transaction (broadcast, track status).

6. **Provide frontend schema**: Embed UI configuration files for the plugin interface.

## Policy Structure Example

```go
type MyPluginPolicy struct {
    ChainID          string   `json:"chain_id"`     // Network identifier
    TokenAddress     string   `json:"token_address"` // Contract address
    RecipientAddress string   `json:"recipient_address"`
    Amount           string   `json:"amount"`       // Wei amount as string
    MaxGasPrice      uint64   `json:"max_gas_price"` // GWEI
}
```

## Security Best Practices

- **Never handle private keys** - Vultisig manages keys through TSS
- **Validate all user inputs** - Check formats, ranges, and addresses
- **Verify transaction parameters** - Must match policy constraints
- **Implement proper error handling** - Return clear validation errors
- **Set reasonable limits** - Maximum amounts and transaction counts

## Common Database Operations

- **CreateTransaction**: Store a new transaction request
- **UpdateTransactionStatus**: Update after broadcast/confirmation
- **GetTransaction**: Retrieve transaction details
- **CountTransactions**: Track transaction history

## Learn from Examples

- Examine the DCA plugin for token swap implementations
- See the Payroll plugin for scheduled payment patterns

Refer to [API Reference](api-reference.md) for complete interface details.
