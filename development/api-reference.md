# Vultisig Plugin API Reference

## Plugin Interface

Every Vultisig plugin must implement this interface:

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

## Method Details

### FrontendSchema() embed.FS
- **Purpose**: Provides UI configuration and assets for the plugin frontend
- **Returns**: Embedded filesystem with UI schema JSON and assets

### ValidatePluginPolicy(policy vtypes.PluginPolicy) error
- **Purpose**: Ensures the plugin policy contains valid configuration
- **Parameters**: `policy` - Plugin policy containing JSON configuration
- **Returns**: Error if policy is invalid, nil if valid

### ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error)
- **Purpose**: Creates unsigned transactions based on policy rules
- **Parameters**: `policy` - Plugin policy containing configuration
- **Returns**: Array of transaction signing requests and error (if any)

### ValidateProposedTransactions(policy vtypes.PluginPolicy, txs []types.PluginKeysignRequest) error
- **Purpose**: Confirms proposed transactions match policy constraints
- **Parameters**: `policy` - Plugin policy, `txs` - Transactions to validate
- **Returns**: Error if transactions don't match policy, nil if valid
- **Example**:
  ```go
  func (p *MyPlugin) ProposeTransactions(policy vtypes.PluginPolicy) ([]types.PluginKeysignRequest, error) {
    var myPolicy MyPluginPolicy
    if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
      return nil, fmt.Errorf("fail to unmarshal policy: %w", err)
    }
    
    // Create transactions based on policy rules
    // ...
    
    return txRequests, nil
  }
  ```

### ValidateProposedTransactions(policy vtypes.PluginPolicy, txs []types.PluginKeysignRequest) error

- **Description**: Validates that proposed transactions conform to policy requirements
- **Parameters**: 
  - `policy` - Plugin policy document
  - `txs` - Array of transaction signing requests to validate
- **Returns**: `error` - Error if transactions are invalid, nil if valid
- **Example**:
  ```go
  func (p *MyPlugin) ValidateProposedTransactions(policy vtypes.PluginPolicy, txs []vtypes.PluginKeysignRequest) error {
    // Validate policy
    if err := p.ValidatePluginPolicy(policy); err != nil {
      return fmt.Errorf("invalid plugin policy: %w", err)
    }
    
    // Parse policy
    var myPolicy MyPluginPolicy
    if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
      return fmt.Errorf("fail to unmarshal policy: %w", err)
    }
    
    // Validate transaction data
    // ...
    
    return nil
  }
  ```

### SigningComplete(ctx context.Context, signature tss.KeysignResponse, signRequest types.PluginKeysignRequest, policy vtypes.PluginPolicy) error

- **Description**: Handles post-signature operations after a transaction has been signed
- **Parameters**: 
  - `ctx` - Context for the operation
  - `signature` - The TSS signature response
  - `signRequest` - The original signing request
  - `policy` - The policy document used for the transaction
- **Returns**: `error` - Error if post-signing operations fail
- **Example**:
  ```go
  func (p *MyPlugin) SigningComplete(ctx context.Context, signature tss.KeysignResponse, signRequest types.PluginKeysignRequest, policy vtypes.PluginPolicy) error {
    // Process the signature
    // Update transaction status
    // Optionally broadcast the transaction
    // ...
    
    return nil
  }
  ```

## Storage API

Plugins have access to a database interface for persistent storage:

### DatabaseStorage Interface

```go
type DatabaseStorage interface {
  // Policy Management
  GetPluginPolicy(ctx context.Context, policyID string) (types.PluginPolicy, error)
  CreatePluginPolicy(ctx context.Context, policy types.PluginPolicy) error
  UpdatePluginPolicy(ctx context.Context, policy types.PluginPolicy) error
  DeletePluginPolicy(ctx context.Context, policyID string) error
  ListPluginPolicies(ctx context.Context, pluginID string, vaultID string, limit, offset int) ([]types.PluginPolicy, error)
  
  // Transaction Management
  CreateTransaction(ctx context.Context, tx types.Transaction) error
  GetTransaction(ctx context.Context, txID string) (types.Transaction, error)
  UpdateTransactionStatus(ctx context.Context, txID string, status string, metadata map[string]interface{}) error
  ListTransactions(ctx context.Context, filterOptions map[string]interface{}, limit, offset int) ([]types.Transaction, error)
}
```

### Example Usage

```go
// Store a new transaction
func (p *MyPlugin) storeTransaction(ctx context.Context, txID, hashHex string, metadata map[string]interface{}) error {
  tx := types.Transaction{
    ID:       txID,
    Hash:     hashHex,
    Status:   types.StatusPending,
    PolicyID: policyID,
    Metadata: metadata,
  }
  return p.db.CreateTransaction(ctx, tx)
}

// Retrieve a transaction
func (p *MyPlugin) getTransaction(ctx context.Context, txID string) (types.Transaction, error) {
  return p.db.GetTransaction(ctx, txID)
}

// Update transaction status
func (p *MyPlugin) updateTransactionStatus(ctx context.Context, txID, status string, metadata map[string]interface{}) error {
  return p.db.UpdateTransactionStatus(ctx, txID, status, metadata)
}
```

## Data Types

### PluginPolicy

```go
type PluginPolicy struct {
  ID        string          `json:"id"`
  PluginID  string          `json:"plugin_id"`
  VaultID   string          `json:"vault_id"`
  Title     string          `json:"title"`
  Policy    json.RawMessage `json:"policy"`
  CreatedAt time.Time       `json:"created_at"`
  UpdatedAt time.Time       `json:"updated_at"`
}
```

### PluginKeysignRequest

```go
type PluginKeysignRequest struct {
  PolicyID    string   `json:"policy_id"`
  PluginID    string   `json:"plugin_id"`
  VaultID     string   `json:"vault_id"`
  Transaction string   `json:"transaction"` // Hex-encoded transaction
  Messages    []string `json:"messages"`    // Hex-encoded message hashes to sign
}
```

### Transaction

```go
type Transaction struct {
  ID         string                 `json:"id"`
  Hash       string                 `json:"hash"`
  Status     string                 `json:"status"`
  PolicyID   string                 `json:"policy_id"`
  Metadata   map[string]interface{} `json:"metadata"`
  CreateTime time.Time              `json:"create_time"`
  UpdateTime time.Time              `json:"update_time"`
}
```

## External Libraries

The following external libraries are commonly used in plugin development:

### Ethereum Integration

```go
import (
  "github.com/ethereum/go-ethereum/accounts/abi"
  "github.com/ethereum/go-ethereum/common"
  "github.com/ethereum/go-ethereum/core/types"
  "github.com/ethereum/go-ethereum/crypto"
  "github.com/ethereum/go-ethereum/ethclient"
)

// Connect to Ethereum node
rpcClient, err := ethclient.Dial("https://mainnet.infura.io/v3/your-api-key")

// Create transaction
tx := types.NewTransaction(
  nonce,
  common.HexToAddress(recipient),
  amount,
  gasLimit,
  gasPrice,
  data,
)
```

### JSON Handling

```go
import (
  "encoding/json"
  "github.com/mitchellh/mapstructure"
)

// Decode configuration
var cfg MyPluginConfig
if err := mapstructure.Decode(rawConfig, &cfg); err != nil {
  return nil, err
}

// Parse policy JSON
var myPolicy MyPluginPolicy
if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
  return fmt.Errorf("fail to unmarshal policy: %w", err)
}
```

## Error Handling

Standard error handling in plugins using Go's error patterns:

```go
// Return wrapped errors with context
if err := json.Unmarshal(policy.Policy, &myPolicy); err != nil {
  return fmt.Errorf("fail to unmarshal policy: %w", err)
}

// Error handling with meaningful messages
if len(myPolicy.ChainID) == 0 {
  return fmt.Errorf("chain_id is required")
}

if !recipientFound {
  return fmt.Errorf("recipient address %s not found in policy", recipientAddress.Hex())
}
```

## Next Steps

- Review the [Development Guide](guide.md) for implementation details
- Study the actual plugin implementations for examples
- Learn about [Security Model](../architecture/security.md) for secure plugin development
