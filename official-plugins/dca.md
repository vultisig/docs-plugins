# DCA (Dollar Cost Averaging) Plugin

The DCA plugin automates ERC-20 token purchases on Ethereum via Uniswap V2, implementing a dollar-cost averaging investment strategy.

## Key Features

- Scheduled token swaps with configurable frequency (minutely to monthly)
- Automatic distribution of investment amount across multiple orders
- Smart token approvals with allowance checking
- Configurable slippage protection
- Optional price range constraints

## Implementation

```go
type DCAPlugin struct {
    uniswapClient *uniswap.Client
    rpcClient     *ethclient.Client
    db            storage.DatabaseStorage
    logger        *logrus.Logger
}
```

## Configuration

```go
type DCAPluginConfig struct {
    RpcURL  string `mapstructure:"rpc_url" json:"rpc_url"`
    Uniswap struct {
        V2Router string `mapstructure:"v2_router" json:"v2_router"`
        Deadline int64  `mapstructure:"deadline" json:"deadline"`
    } `mapstructure:"uniswap" json:"uniswap"`
}
```

| Option | Description | Example |
|--------|-------------|---------|
| rpc_url | Ethereum RPC URL | https://mainnet.infura.io/v3/your-key |
| uniswap.v2_router | Router contract address | 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D |
| uniswap.deadline | Transaction deadline (minutes) | 20 |

## Policy Structure

```go
type DCAPolicy struct {
    ChainID           string `json:"chain_id"`              // Network chain ID
    TotalAmount       string `json:"total_amount"`          // Total investment (wei)
    SourceTokenID     string `json:"source_token_id"`       // Token to sell
    DestinationTokenID string `json:"destination_token_id"` // Token to buy
    TotalOrders       string `json:"total_orders"`         // Number of purchases
    Schedule          struct {
        Interval  string `json:"interval"`                  // Time value
        Frequency string `json:"frequency"`                 // minutely/hourly/daily/weekly/monthly
    } `json:"schedule"`
    PriceRange *struct {                                   // Optional
        Min string `json:"min,omitempty"`                  // Minimum token price
        Max string `json:"max,omitempty"`                  // Maximum token price
    } `json:"price_range,omitempty"`
}
```

### Example

```json
{
  "chain_id": "1",
  "total_amount": "100000000000000000000",    // 100 ETH
  "source_token_id": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",  // WETH
  "destination_token_id": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",  // USDC
  "total_orders": "5",
  "schedule": { "interval": "1", "frequency": "hourly" },
  "price_range": { "min": "1800", "max": "2000" }
}
```

## Operation Flow

### Key Functions

#### 1. Policy Validation

```go
func (p *DCAPlugin) ValidatePluginPolicy(policyDoc vtypes.PluginPolicy) error {
    // Parse policy JSON
    var dcaPolicy DCAPolicy
    if err := json.Unmarshal(policyDoc.Policy, &dcaPolicy); err != nil {
        return fmt.Errorf("fail to unmarshal DCA policy: %w", err)
    }
    
    // Validate required fields
    if dcaPolicy.ChainID == "" || dcaPolicy.TotalAmount == "" || 
       dcaPolicy.SourceTokenID == "" || dcaPolicy.DestinationTokenID == "" || 
       dcaPolicy.TotalOrders == "" {
        return errors.New("missing required fields")
    }
    
    // Validate numeric values and ranges
    // Validate schedule parameters
    
    return nil
}
```

#### 2. Transaction Generation

```go
func (p *DCAPlugin) ProposeTransactions(policy vtypes.PluginPolicy) ([]vtypes.PluginKeysignRequest, error) {
    // Check if policy is already completed
    completedSwaps, _ := p.getCompletedSwapTransactionsCount(context.Background(), policy.ID)
    totalOrders, _ := new(big.Int).SetString(dcaPolicy.TotalOrders, 10)
    
    if big.NewInt(completedSwaps).Cmp(totalOrders) >= 0 {
        p.completePolicy(context.Background(), policy)
        return nil, ErrCompletedPolicy
    }
    
    // Calculate this order's swap amount
    totalAmount, _ := new(big.Int).SetString(dcaPolicy.TotalAmount, 10)
    swapAmount := p.calculateSwapAmountPerOrder(totalAmount, totalOrders, completedSwaps)
    
    // Generate transactions (both approval if needed + swap)
    txsData, _ := p.generateSwapTransactions(
        chainID, 
        signerAddr, 
        dcaPolicy.SourceTokenID, 
        dcaPolicy.DestinationTokenID, 
        swapAmount
    )
    
    // Convert to keysign requests
    // ...
    
    return requests, nil
}
```

#### 3. Transaction Execution

```go
func (p *DCAPlugin) SigningComplete(ctx context.Context, signature tss.KeysignResponse, 
    signRequest vtypes.PluginKeysignRequest, policy vtypes.PluginPolicy) error {
    
    // Get chain ID from policy
    chainID, _ := new(big.Int).SetString(dcaPolicy.ChainID, 10)
    
    // Sign transaction using TSS signature
    signedTx, _, _ := sigutil.SignLegacyTx(
        signature, 
        signRequest.Messages[0], 
        signRequest.Transaction, 
        chainID
    )
    
    // Broadcast transaction
    p.rpcClient.SendTransaction(context.Background(), signedTx)
    
    // Wait for confirmation
    receipt, _ := bind.WaitMined(context.Background(), p.rpcClient, signedTx)
    
    return nil
}
```

### Smart Features

#### Amount Distribution

The plugin distributes the total investment amount across orders:

```go
func (p *DCAPlugin) calculateSwapAmountPerOrder(totalAmount, totalOrders *big.Int, completedSwaps int64) *big.Int {
    baseAmount := new(big.Int).Div(totalAmount, totalOrders)
    remainder := new(big.Int).Mod(totalAmount, totalOrders)
    
    swapAmount := new(big.Int).Set(baseAmount)
    if big.NewInt(completedSwaps+1).Cmp(remainder) <= 0 {
        swapAmount.Add(swapAmount, big.NewInt(1))
    }
    return swapAmount
}
```

#### Token Approval Management

The plugin automatically checks if token approvals are needed:

```go
// Check if approval is needed
allowance, _ := p.uniswapClient.GetAllowance(*signerAddress, srcTokenAddress)
if allowance.Cmp(swapAmount) < 0 {
    // Create approval transaction
    txHash, rawTx, _ := p.uniswapClient.ApproveERC20Token(
        chainID, signerAddress, srcTokenAddress, 
        *p.uniswapClient.GetRouterAddress(), swapAmount, 0
    )
    // Add to transaction batch
}
```
```

## Uniswap Integration

The DCA plugin integrates specifically with Uniswap V2 for token swaps:

```go
// Setting up Uniswap Client
routerAddress := gcommon.HexToAddress(cfg.Uniswap.V2Router)
uniswapCfg := uniswap.NewConfig(
    rpcClient,
    &routerAddress,
    2000000, // Gas limit
    50000,   // Gas price
    time.Duration(cfg.Uniswap.Deadline)*time.Minute,
)

uniswapClient, err := uniswap.NewClient(uniswapCfg)
```

The integration handles:

1. **Token Approvals**: Checking allowances and creating approval transactions when needed
2. **Swap Execution**: Constructing and executing swap transactions with appropriate parameters
3. **Slippage Protection**: Setting minimum output amounts with a configured slippage tolerance (default 1%)
4. **Deadline Management**: Setting transaction deadlines to protect against pending transactions

## Smart Order Execution

The plugin implements several key algorithms:

### Swap Amount Calculation

The plugin intelligently distributes the total investment amount across all orders:

```go
func (p *DCAPlugin) calculateSwapAmountPerOrder(totalAmount, totalOrders *big.Int, completedSwaps int64) *big.Int {
    baseAmount := new(big.Int).Div(totalAmount, totalOrders)
    remainder := new(big.Int).Mod(totalAmount, totalOrders)
    
    // Determine swap amount for the next order
    swapAmount := new(big.Int).Set(baseAmount)
    if big.NewInt(completedSwaps+1).Cmp(remainder) <= 0 {
        swapAmount.Add(swapAmount, big.NewInt(1)) // Add 1 to distribute remainder
    }
    return swapAmount
}
```

### Transaction Batching

The plugin automatically creates transaction sequences:

1. **Approval Transactions**: When token allowance is insufficient
2. **Swap Transactions**: For the actual token exchange
3. **Smart Nonce Management**: Ensuring proper transaction ordering

### Price Optimization

For each swap, the plugin:
1. Queries current exchange rates
2. Calculates expected output amounts
3. Sets appropriate minimum output with slippage tolerance
4. Optionally enforces price range constraints (min/max price)

## Security Considerations

- **Transaction Validation**: All transactions are validated against policy constraints
- **Policy Verification**: Source/destination tokens and amounts are strictly verified
- **Smart Contract Safety**: Only interacts with verified Uniswap V2 router contracts
- **Order Protection**: Transactions include deadline parameters to prevent stale executions
- **Slippage Protection**: Minimum output amounts are enforced on all swaps
- **Policy Completion**: Policies automatically mark themselves complete after all orders execute

## Troubleshooting

Common issues:

- **Token Approval Failures**: Ensure the vault has approved the Uniswap Router to spend source tokens
- **Slippage Errors**: Increase slippage tolerance for volatile token pairs
- **Gas Estimation Errors**: May indicate issues with token contracts or insufficient ETH for gas
- **Scheduling Errors**: Verify schedule configuration is valid according to constraints
- **Price Range Issues**: If using price range constraints, ensure they are properly configured

For detailed troubleshooting, check the plugin logs for specific error messages and transaction details.
