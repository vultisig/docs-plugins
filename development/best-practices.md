# Vultisig Plugin Best Practices

## Security Best Practices

- **Never handle private keys** - Use only Vultisig's TSS system for signing
- **Validate all policy inputs** - Check ranges, formats, and addresses
- **Verify transaction data** - Ensure transactions match policy constraints
- **Use secure connections** - HTTPS for external APIs, proper TLS verification
- **Implement rate limiting** - Protect against abuse in external API calls

## Transaction Processing

```go
func (p *MyPlugin) ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error) {
    var myPolicy MyPluginPolicy
    if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
        return nil, fmt.Errorf("failed to parse policy: %w", err)
    }
    // Create transaction based on policy parameters...
    return []types.PluginKeysignRequest{{Transaction: txData, Messages: []string{txHash}}}, nil
}
```

## Error Handling

- **Provide detailed errors** - Return specific, actionable error messages
- **Wrap errors** - Use `fmt.Errorf("context: %w", err)` to preserve context
- **Log with context** - Include policy IDs, transaction hashes in log entries
- **Handle external failures** - Gracefully handle RPC failures, API timeouts

## Performance Optimization

- **Cache blockchain data** - Don't query the same data repeatedly
- **Batch database operations** - Use transactions for multiple updates
- **Set timeouts** - Use context timeouts for all external operations
- **Limit resource usage** - Control memory usage for large transaction sets

## Plugin Organization

- **Keep plugin focused** - Each plugin should have a clear, single purpose
- **Separate concerns** - Split policy validation from transaction generation
- **Document policy schema** - Provide clear explanation of all policy fields
- **Use consistent naming** - Follow Vultisig naming conventions

## Testing Strategy

- **Test policy validation** - Ensure invalid policies are rejected
- **Test transaction generation** - Verify correct transaction parameters
- **Test error conditions** - Check handling of network failures, invalid data
- **Use table-driven tests** - Test multiple scenarios efficiently

## Version Management

- **Use semantic versioning** - For plugin and policy versions
- **Support backward compatibility** - Handle older policy formats
- **Document changes** - Maintain changelog for all plugin versions

See [API Reference](api-reference.md) and [Testing Guidelines](testing.md) for more details.
