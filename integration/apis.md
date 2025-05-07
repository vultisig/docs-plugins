# Vultisig Plugin API Integration

This guide explains how Vultisig plugins integrate with external APIs for transaction management.

## Vultisig Plugin API Requirements

Vultisig plugins need to interact with specific external services to function:

- **Token Price Services**: For DCA (Dollar Cost Average) plugins to calculate token purchase amounts
- **Blockchain Networks**: To query balances and broadcast signed transactions
- **Contract State Readers**: To check contract state and calculate parameters

## Integration with Vultisig Core

Plugins consume these Vultisig system APIs:

- **KeysignResponse**: Signature result from the TSS system
- **PluginKeysignRequest**: Describes transactions to be signed
- **PluginPolicy**: Contains policy configuration for validation

## Example: Policy Validation with External API

```go
func (p *MyPlugin) ValidatePluginPolicy(policy vtypes.PluginPolicy) error {
    var myPolicy MyPluginPolicy
    if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
        return fmt.Errorf("invalid policy format: %w", err)
    }
    
    // Validate token address on-chain
    valid, err := validateTokenAddress(myPolicy.TokenAddress, myPolicy.ChainID)
    if err != nil || !valid {
        return fmt.Errorf("invalid token address")
    }
    return nil
}
```

## Security in Vultisig Plugin APIs

- Store API credentials in the plugin policy or environment variables
- Validate all external data before using in transactions
- Implement signature verification for critical data sources
- Limit permissions of API credentials used by plugins

## Handling Transaction Completion

When implementing `SigningComplete` for your plugin:

- Broadcast signed transactions to the blockchain
- Update external systems with transaction status
- Store transaction hashes for later verification

## Vultisig Plugin API Best Practices

- Query token prices at transaction creation time, not signing time
- Cache blockchain state data where appropriate to minimize RPC calls
- Include chain IDs in all external API requests for cross-chain plugins
- Add request IDs to trace API calls throughout the plugin lifecycle