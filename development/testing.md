# Vultisig Plugin Testing Guidelines

## Key Testing Areas

Focus your Vultisig plugin tests on these critical areas:

1. **Policy Validation**: Test `ValidatePluginPolicy` with various policy inputs
2. **Transaction Generation**: Verify `ProposeTransactions` creates correct transactions
3. **Transaction Validation**: Ensure `ValidateProposedTransactions` properly validates
4. **Signature Handling**: Test `SigningComplete` with mocked TSS signatures
5. **Error Handling**: Verify proper responses to invalid inputs and failures

## Table-Driven Testing

```go
func TestPolicyValidation(t *testing.T) {
    plugin := NewMyPlugin(mockDB, mockLogger)
    tests := []struct {
        name      string
        policy    MyPluginPolicy
        expectErr bool
    }{
        {"valid policy", validPolicy(), false},
        {"invalid recipient", policyWithInvalidRecipient(), true},
    }
    
    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            policyJSON, _ := json.Marshal(tc.policy)
            pluginPolicy := vtypes.PluginPolicy{Policy: policyJSON}
            
            err := plugin.ValidatePluginPolicy(pluginPolicy)
            if tc.expectErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

## Mocking Dependencies

When testing Vultisig plugins, always mock these dependencies:

- **Database**: Mock the DatabaseStorage interface
- **Blockchain RPC**: Mock Ethereum client responses
- **TSS System**: Create mock KeysignResponse objects for testing

## Testing Transaction Properties

Verify your plugin generates transactions with the correct properties:

- Chain ID matches the policy's specified network
- Recipient addresses are properly encoded
- Transaction values match policy specifications
- Gas parameters are within acceptable ranges

## Testing the Plugin Lifecycle

Test the complete Vultisig workflow from policy to execution:

1. Policy validation
2. Transaction generation
3. Transaction validation
4. Signature application
5. Blockchain submission

## Best Practices

- **Focus on Plugin interfaces**: Test all methods in the Plugin interface
- **Test error conditions**: Verify behavior with malformed policies and network failures
- **Use fixtures**: Create reusable policy and transaction templates
- **Test gas estimation**: Ensure gas parameters handle network congestion
- **Test policy constraints**: Verify policy limits on amounts and frequencies

For more information, see the Go testing documentation and examples of existing Vultisig plugins.
