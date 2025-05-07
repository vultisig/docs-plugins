# Vultisig Plugin Monitoring

This guide covers monitoring implementation specific to Vultisig plugins.

## Vultisig Plugin Critical Paths

Monitor these key plugin operations:

- **Policy Validation**: Track validation errors and validation time
- **Transaction Proposal**: Monitor proposal generation and transaction counts
- **Signature Application**: Record signature application success/failure
- **Blockchain Submission**: Track transaction broadcast confirmation status

## Plugin Logging in Vultisig

Implement structured logging for Vultisig plugin operations:

```go
func (p *MyPlugin) ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error) {
    start := time.Now()
    p.logger.WithFields(logrus.Fields{
        "policy_id": policy.ID,
        "vault_id": policy.VaultID,
        "plugin_id": policy.PluginID,
    }).Info("Starting transaction proposal")
    
    // Implement transaction proposal logic
    
    p.logger.WithField("duration_ms", time.Since(start).Milliseconds()).Info("Proposal complete")
    return txRequests, nil
}
```

## Vultisig-Specific Metrics

Track these plugin-specific metrics:

- **Policy activation counts**: Number of active plugin policies
- **Transaction volume**: Total value of transactions processed
- **Signature success rate**: Percentage of successful signatures
- **Plugin execution time**: Duration of each plugin method
- **Chain-specific metrics**: Performance by blockchain network

## Transaction Status Tracking

During `SigningComplete`, track transaction status:

```go
func (p *MyPlugin) SigningComplete(ctx context.Context, signature tss.KeysignResponse, 
    signRequest types.PluginKeysignRequest, policy vtypes.PluginPolicy) error {
    
    // Apply signature and broadcast transaction
    txHash, err := p.broadcastTransaction(ctx, signature, signRequest)
    if err != nil {
        p.metrics.Increment("tx_broadcast_failures", map[string]string{"plugin_id": policy.PluginID})
        return err
    }
    
    p.metrics.Increment("tx_broadcast_success", map[string]string{"plugin_id": policy.PluginID})
    return nil
}
```

## Vultisig Plugin Health Checks

Implement health checks specific to your plugin:

- Verify RPC connections to required blockchain networks
- Check availability of external APIs needed by your plugin
- Validate policy data structure integrity
- Monitor transaction confirmation times

## Plugin-Specific Observability

- Log all policy validation errors with detailed reason
- Track per-chain metrics separately for multi-chain plugins
- Monitor gas costs for transaction execution
- Implement transaction receipt validation