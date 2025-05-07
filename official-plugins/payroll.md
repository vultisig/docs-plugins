# Payroll Plugin

The Payroll plugin automates cryptocurrency payments using secure, scheduled transactions with Vultisig's threshold signature scheme.

## Features
- Scheduled payments to multiple recipients
- Supports ERC-20 tokens
- Daily, weekly, or monthly schedules
- Secure transaction validation and signing

## Implementation
The plugin is a Go package for Vultisig:

```go
type PayrollPlugin struct {
    db           storage.DatabaseStorage
    rpcClient    *ethclient.Client
    logger       logrus.FieldLogger
}
```

## Configuration
Set the Ethereum RPC endpoint:

```go
type PayrollPluginConfig struct {
    RpcURL string `json:"rpc_url"`
}
```

| Option   | Description           | Example Value                          |
|----------|-----------------------|----------------------------------------|
| rpc_url  | Ethereum RPC URL      | https://mainnet.infura.io/v3/api-key   |

## Policy Structure
Define payment schedules and recipients:

```go
type PayrollPolicy struct {
    ChainID    []string           `json:"chain_id"`
    Recipients []PayrollRecipient `json:"recipients"`
    Schedule   Schedule           `json:"schedule"`
}
```

### Policy Example
```json
{
  "chain_id": ["1"],
  "recipients": [
    {"address": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e", "amount": "100"},
    {"address": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4", "amount": "50"}
  ],
  "schedule": {
    "frequency": "monthly",
    "start_time": "2023-01-01T00:00:00Z"
  }
}
```

## How It Works
1. **Create Policy**: Define recipients, amounts, and schedule.
2. **Schedule Payments**: The plugin checks policies and proposes transactions.
3. **Validate Transactions**: Ensures transactions match policy rules.
4. **Sign Transactions**: Uses Vultisig's secure signing service.
5. **Complete Transactions**: Logs and updates transaction status.

```go
func (p *PayrollPlugin) ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error) {
    var payrollPolicy PayrollPolicy
    json.Unmarshal(policy.Policy, &payrollPolicy)
    // Create transactions
    return txRequests, nil
}
```

## Security
- Validates transactions against policies
- No access to private keys
- Prevents transaction replays
- Logs all actions for audits

## Ethereum Integration
- Connects to Ethereum via RPC
- Supports ERC-20 tokens
- Manages transaction nonces and gas prices

## Troubleshooting
- **Invalid Address**: Check Ethereum address format.
- **Insufficient Funds**: Ensure vault has enough tokens.
- **Transaction Fails**: Verify gas prices or token approvals.
- **Schedule Issues**: Use ISO-8601 format for dates.

Check logs for detailed error information.