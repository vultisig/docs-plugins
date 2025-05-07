# Official Plugins Overview

This document provides an overview of the official plugins available for Vultisig. These reference implementations demonstrate best practices for plugin development.

## Available Official Plugins

| Plugin | Description | Use Case |
|--------|-------------|----------|
| [Payroll](payroll.md) | Automated crypto payments | Recurring payments to multiple recipients |
| [DCA](dca.md) | Dollar Cost Averaging | Scheduled crypto purchases |

## Implementation Pattern

All official plugins follow a consistent implementation pattern that you can emulate in your own plugins:

```go
type MyPlugin struct {
    db     database.Storage
    logger *logrus.Logger
}

func (p *MyPlugin) ValidatePluginPolicy(policy vtypes.PluginPolicy) error {
    // Validate policy JSON against custom structure
    var myPolicy MyPluginPolicy
    if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
        return fmt.Errorf("invalid policy format: %w", err)
    }
    // Validate required fields
    return validateFields(myPolicy)
}
```

## Key Learning Points

Study these official plugins to learn about:

- **Proper policy validation**: Thorough validation of all user inputs
- **Error handling**: Structured and detailed error responses
- **Time-based transactions**: Scheduling and triggering transactions based on time conditions
- **Database integration**: Persisting policy state between transactions
- **Frontend integration**: Schema design for user configuration

## Policy Structure

Each plugin defines its own policy structure. For example, the Payroll plugin uses:

```json
{
  "chain_id": "1",
  "token_address": "0x6B175474E89094C44Da98b954EedeAC495271d0F",
  "recipients": [
    {"address": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e", "amount": "100"}
  ],
  "schedule": {
    "frequency": "monthly",
    "start_date": "2023-01-01T00:00:00Z"
  }
}
```

## Common Functions

Study how these plugins implement these common functions:

1. **Schedule parsing**: Converting user-friendly formats to cron or time-based triggers
2. **Transaction building**: Constructing valid blockchain transactions
3. **Balance checking**: Validating sufficient funds before proposing transactions
4. **State management**: Tracking completed transactions to prevent duplicates

For more details on specific implementations, see the individual plugin documentation pages:
- [Payroll Plugin](payroll.md)
- [DCA Plugin](dca.md)
