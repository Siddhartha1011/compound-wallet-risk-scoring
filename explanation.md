# Compound Wallet Risk Scoring - Explanation

This document outlines the methodology used for assessing the risk of wallets interacting with the Compound V2/V3 protocol.

## Data Collection Method

Transaction data was collected for each wallet address using the Covalent API, focusing on interactions with the Compound protocol. The steps followed were:

1. **Wallet Addresses**: A list of wallet addresses was provided.
2. **Protocol Transactions**: Covalent’s `/v1/{chain_id}/address/{address}/transactions_v2/` endpoint was used to fetch historical transaction data.
3. **Filtering**: Only transactions involving Compound V2/V3 (identified by method signatures and protocol addresses) were retained.

## Feature Selection Rationale

| Feature               | Derived From                    | Insight Provided                      |
|----------------------|----------------------------------|----------------------------------------|
| `total_borrowed_usd` | method == 'borrow', value_usd    | Indicates how much credit the wallet utilized. |
| `total_repaid_usd`   | method contains 'repay'          | Measures how disciplined the user is with debt. |
| `liquidation_count`  | method == 'liquidateBorrow'      | Reflects risk mismanagement behavior. |
| `activity_days`      | date(first_tx) to date(last_tx)  | Shows long-term engagement or abuse.  |
| `avg_gas_price`      | Mean of gas_price                | Bots often show abnormal gas patterns. |
| `avg_gas_offered`    | Mean of gas_offered              | Helps detect automation patterns.     |
| `tx_count`           | Count of all transactions        | Indicates engagement level.           |


## Scoring Method

Each wallet was assigned a **risk score between 0 and 1000**, where **higher scores indicate lower risk**. The scoring process includes:

1. **Normalization**:
   - All numeric features were normalized using Min-Max scaling.
2. **Weight Assignment**:
   - `total_repaid_usd`: +20%
   - `total_borrowed_usd`: +15%
   - `activity_days`: +15%
   - `tx_count`: +10%
   - `avg_gas_price` & `avg_gas_offered`: -20% (combined)
   - `liquidation_count`: -20%

3. **Score Aggregation**:
   - Positive indicators increase the score.
   - Negative indicators penalize the score.
   - Final score scaled to 0–1000.

## Justification of Risk Indicators

- **Repayment Discipline**: Users who repay loans indicate financial responsibility.
- **Borrowing Volume**: High-volume borrowers are assumed to be sophisticated users.
- **Liquidation Events**: Frequent liquidations are strong indicators of poor risk management.
- **Activity Span**: Consistent long-term use suggests reliability.
- **Gas Usage Patterns**: Abnormal values may indicate bots or exploitative behaviors.
- **Transaction Frequency**: Very low or extremely high frequency can be red flags.

## Scalability

This risk scoring approach is:
- **Modular** – easily extendable with more DeFi protocols.
- **Automatable** – can run daily/weekly using pipelines.
- **Explainable** – relies on interpretable features and clear logic.

