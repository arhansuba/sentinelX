# MultiversX AI Smart Contract Sentinel

AI-powered security monitoring for MultiversX smart contracts that analyzes on-chain transactions in real-time to detect suspicious patterns and potential exploits before they cause damage.

## Overview

This project is an AI-driven security monitoring solution for the MultiversX blockchain that:

- Monitors smart contracts in real-time for suspicious transaction patterns.
- Detects potential vulnerabilities using Google's Gemini AI and pattern recognition.
- Alerts developers and users about potential security risks.
- Provides a visual dashboard for monitoring contract security health.

[Backend Repository](https://github.com/arhansuba/sentimentX-backend)  
[Frontend Repository](https://github.com/arhansuba/SentimentX-frontend)

## Features

- **AI-Powered Analysis**: Uses Google's Gemini AI to analyze contract behavior and identify potential vulnerabilities.
- **Real-time Transaction Monitoring**: Track on-chain activity as it happens.
- **Pattern Recognition**: Identify common attack vectors (reentrancy, flash loans, etc.).
- **Visual Dashboard**: Display contract security health and potential issues.
- **Alert System**: Notify users of suspicious activities.
- **Risk Scoring**: Rate contracts based on vulnerability potential.

## Gemini AI Integration Features

The project leverages Google's Gemini 1.5 Flash model for advanced security analysis capabilities:

- **Smart Contract Analysis**: Deep analysis of contract code to identify vulnerabilities, security risks, and best practice violations.
- **Transaction Pattern Recognition**: AI-powered detection of suspicious transaction patterns that might indicate an attack.
- **Contextual Vulnerability Assessment**: Understanding the context and semantics of smart contract logic beyond simple pattern matching.
- **Risk Scoring**: Advanced risk evaluation algorithm informed by AI analysis.
- **Remediation Suggestions**: AI-generated suggestions for fixing identified security issues.
- **Natural Language Explanations**: Clear, human-readable explanations of detected vulnerabilities.
- **Security Best Practices**: AI-assisted recommendations for improving overall contract security.

## Working with Gemini AI

This project integrates Google's Gemini 1.5 Flash model for advanced AI-powered security analysis. Here's how the integration works:

### Setup

1. **Get a Gemini API Key**: Visit Google AI Studio to create a free API key. The free tier provides sufficient quota for testing during the hackathon.
2. **Configure the Environment**: Add your Gemini API key to the `.env` file in the backend directory:

   ```sh
   GEMINI_API_KEY=your_api_key_here
   ```

### How the AI Integration Works

The MultiversX AI Smart Contract Sentinel uses Gemini AI in several key ways:

- **Contract Code Analysis**:
  - Smart contract code is sent to Gemini for semantic analysis.
  - The AI identifies potential vulnerabilities beyond simple pattern matching.
  - Results include risk scores, vulnerability details, and recommended fixes.

- **Transaction Analysis**:
  - Transaction data is analyzed for suspicious patterns.
  - The AI correlates transaction data with known attack patterns.
  - Anomalous transaction patterns are flagged for review.

- **Security Recommendations**:
  - AI-generated recommendations help developers improve contract security.
  - Specific code-level guidance for fixing identified issues.

### Benefits of Gemini Integration

- **Contextual Understanding**: Gemini understands the semantics and context of code, not just patterns.
- **Natural Language Interface**: Security findings are explained in clear, understandable language.
- **Continuous Learning**: The model benefits from Google's continuous model improvements.
- **Multimodal Capabilities**: The model can analyze both code and textual descriptions.

The integration is designed to be modular, allowing the system to function with basic pattern recognition if the Gemini API is unavailable, while providing enhanced capabilities when activated.

## Example: AI Analysis of a Smart Contract

### Sample Smart Contract with Vulnerability

```rust
#![no_std]

elrond_wasm::imports!();
elrond_wasm::derive_imports!();

#[elrond_wasm::contract]
pub trait TokenSwap {

    #[init]
    fn init(&self) {
        self.owner().set(&self.blockchain().get_caller());
    }

    #[view(getOwner)]
    #[storage_mapper("owner")]
    fn owner(&self) -> SingleValueMapper<ManagedAddress>;

    #[endpoint]
    fn swap_tokens(
        &self,
        #[payment_token] payment_token: TokenIdentifier,
        #[payment_amount] payment_amount: BigUint,
        token_out: TokenIdentifier,
        min_amount_out: BigUint,
    ) -> BigUint {
        require!(payment_amount > 0, "Zero payment amount");
        
        // Get the exchange rate from price oracle
        let exchange_rate = self.get_exchange_rate(&payment_token, &token_out);
        
        // Calculate output amount
        let amount_out = &payment_amount * exchange_rate / BigUint::from(1000u32);
        require!(amount_out >= min_amount_out, "Slippage too high");
        
        // Send tokens to user - VULNERABILITY: No checks before transfer
        self.send().direct(
            &self.blockchain().get_caller(),
            &token_out,
            0,
            &amount_out,
        );
        
        // Update state after transfer
        self.update_reserves(&payment_token, &payment_amount, &token_out, &amount_out);
        
        amount_out
    }

    // Get exchange rate from oracle
    fn get_exchange_rate(
        &self,
        token_in: &TokenIdentifier,
        token_out: &TokenIdentifier,
    ) -> BigUint {
        // In real implementation, this would call an oracle or use reserves
        BigUint::from(1200u32) // Mock rate for demo
    }

    // Update reserves after swap
    fn update_reserves(
        &self,
        token_in: &TokenIdentifier,
        amount_in: &BigUint,
        token_out: &TokenIdentifier,
        amount_out: &BigUint,
    ) {
        // Update state after transfer is done
        // In a real implementation, this would update liquidity reserves
    }
}
```

### Gemini AI Analysis Response

```json
{
  "vulnerabilities": [
    {
      "type": "Reentrancy Vulnerability",
      "risk_level": "critical",
      "location": "swap_tokens function",
      "explanation": "The contract updates state after performing an external call (send().direct). This creates a potential reentrancy vulnerability where an attacker could reenter the contract before the state is updated.",
      "recommendation": "Implement the checks-effects-interactions pattern by updating state before making external calls. Move the update_reserves() call to before the send().direct() call."
    },
    {
      "type": "Missing Access Control",
      "risk_level": "medium",
      "location": "update_reserves function",
      "explanation": "The update_reserves function doesn't implement access controls to ensure it's only called by the swap_tokens function or other authorized functions.",
      "recommendation": "Consider making the function private or implementing proper access controls to prevent unauthorized calls."
    },
    {
      "type": "Hardcoded Exchange Rate",
      "risk_level": "medium",
      "location": "get_exchange_rate function",
      "explanation": "The exchange rate is hardcoded rather than derived from actual reserves or an external price oracle, which could lead to front-running attacks or price manipulation.",
      "recommendation": "Implement a proper price discovery mechanism based on liquidity reserves or integrate with a reliable price oracle service."
    }
  ],
  "risk_score": 85,
  "overall_assessment": "This contract has a critical reentrancy vulnerability and several medium-risk issues that should be addressed before deployment."
}
```

