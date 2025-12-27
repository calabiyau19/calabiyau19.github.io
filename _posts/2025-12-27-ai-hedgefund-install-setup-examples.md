---
layout: post
draft: false
title: "AI HedgeFund installation and setup guide"
date: 2025-12-27
description: "This guide documents how to install and set up AI HedgeFund from GitHub and requires an API key that must be purchased for a small amount.  Since we were using Claude, we bought our API key from Anthropic. AI Hedgefund takes stock symbols input and then allows you to pick from one or more or all of 18 investors to analyze your stock picks and recommend buy, sell, short, or hold strategies."

---

## AI Hedge Fund Installation and Setup Guide

This guide walks through installing and configuring the AI Hedge Fund tool on an Ubuntu VM, using Anthropic's Claude API to analyze stocks with multiple AI investment analysts.

## System Requirements

- Ubuntu 24.04 (or similar Debian-based Linux)
- Python 3.12+
- Git, curl (pre-installed on Ubuntu 24.04)
- Anthropic API key (from console.anthropic.com)

## Step 1: Verify Python Installation

Check that Python 3 is installed:
```sh
python3 --version
```

Expected output: Python 3.12.3 (or similar)

## Step 2: Verify Git Installation

Confirm git is available:
```sh
git --version
```

Expected output: git version 2.43.0 (or similar)

## Step 3: Verify curl Installation

Check curl is installed:
```sh
curl --version
```

Expected output: curl 8.5.0 (or similar)

## Step 4: Install Poetry

Poetry is a Python package manager that handles dependencies for the AI Hedge Fund project.

Install Poetry:
```sh
curl -sSL https://install.python-poetry.org | python3 -
```

Add Poetry to your PATH for Fish shell:
```sh
fish -c 'set -U fish_user_paths /home/mark/.local/bin $fish_user_paths'
```

For Bash shell, add this line to your ~/.bashrc:
```sh
export PATH="/home/mark/.local/bin:$PATH"
```

Verify Poetry installation:
```sh
poetry --version
```

Expected output: Poetry (version 2.2.1)

## Step 5: Clone the AI Hedge Fund Repository

Navigate to your home directory and clone the project:
```sh
cd ~
git clone https://github.com/virattt/ai-hedge-fund.git
cd ai-hedge-fund
```

Verify you're in the correct directory:
```sh
pwd
```

Expected output: /home/mark/ai-hedge-fund

## Step 6: Install Project Dependencies

Use Poetry to install all required Python packages:
```sh
poetry install
```

This will install approximately 125 packages including:
- anthropic (for Claude API)
- langchain (AI agent framework)
- pandas (data analysis)
- matplotlib (visualization)
- And many dependencies

Installation takes 2-3 minutes.

## Step 7: Configure API Keys

Create your environment configuration file:
```sh
cp .env.example .env
```

Open the .env file in your preferred editor:
```sh
nano .env
```

Add your Anthropic API key. Find this line:
```
ANTHROPIC_API_KEY=your-anthropic-api-key
```

Replace `your-anthropic-api-key` with your actual API key from console.anthropic.com.

Save and exit (in nano: Ctrl+O, Enter, Ctrl+X)

Verify your API key was saved:
```sh
grep ANTHROPIC_API_KEY .env
```

## Understanding the Free Stock Data

The Financial Datasets API provides free data for 5 stocks:
- AAPL (Apple)
- GOOGL (Google)
- MSFT (Microsoft)
- NVDA (Nvidia)
- TSLA (Tesla)

All other stocks require a paid Financial Datasets API key.

## Step 8: Run Your First Analysis

Analyze a single stock with all 18 analysts:
```sh
poetry run python src/main.py --ticker AAPL
```

When prompted:
1. Press 'a' to select all analysts
2. Press Enter
3. Use arrow keys to select "Claude Sonnet 4.5"
4. Press Enter

The analysis will take 3-5 minutes and cost approximately $0.14 in API usage.

## Step 9: Run Focused Analysis with Selected Analysts

Analyze multiple stocks with specific analysts:
```sh
poetry run python src/main.py --ticker NVDA,GOOGL,MSFT
```

When prompted:
1. Use SPACE to select: Stanley Druckenmiller, Warren Buffett, Peter Lynch
2. Press Enter
3. Select "Claude Sonnet 4.5"
4. Press Enter

This analyzes 3 stocks with 3 analysts, costing approximately $0.11 in API usage.

## Understanding the Output

The analysis provides:

### Agent Analysis Table
Shows each analyst's recommendation:
- Signal: BULLISH, BEARISH, or NEUTRAL
- Confidence: Percentage (0-100%)
- Reasoning: Detailed explanation of their recommendation

### Trading Decision
- Action: BUY, SHORT, or HOLD
- Quantity: Suggested number of shares
- Confidence: Overall confidence level
- Reasoning: Why this decision was made

### Portfolio Summary
Aggregates all stock recommendations with vote counts (Bullish/Bearish/Neutral).

## Interpreting Signals

### BUY Signal
The analysts recommend purchasing the stock. Higher confidence (70%+) indicates stronger consensus.

### SHORT Signal
The analysts believe the stock is overvalued or will decline. This is essentially an "avoid" signal for most individual investors, as shorting requires margin accounts and carries unlimited risk.

### HOLD Signal
Insufficient data or no clear trading opportunity. Common for stocks without full financial data.

## Understanding Analyst Perspectives

### Stanley Druckenmiller
Focuses on momentum, asymmetric risk-reward, and macro trends. Looks for growth with favorable entry points.

### Warren Buffett
Emphasizes business quality, competitive moats, and margin of safety. Won't overpay even for great companies.

### Peter Lynch
Seeks "ten-baggers" with reasonable valuations. Uses PEG ratio (P/E divided by growth rate). Values under 1.0 are attractive.

## Cost Management

Actual costs per run:
- 18 analysts on 1 stock: ~$0.14
- 3 analysts on 4 stocks: ~$0.11
- Costs scale with number of analysts and stocks analyzed

With $5 in API credit, you can run approximately 35-45 analyzes.

## Backtesting Warning

The backtester runs AI analysis for every business day in your date range:
```sh
poetry run python src/backtester.py --ticker AAPL --start-date 2024-01-01 --end-date 2024-12-31
```

This would cost approximately $27 (252 trading days x ~$0.11 per day) for a full year.

Only run backtests on short time periods (1 month = ~$2.31) or when you're ready to invest in validation.

## Common Use Cases

### Analyzing Your Index Fund Holdings

If you own an S&P 500 index fund like FXAIX, you can analyze the top holdings to identify potential individual stock picks:
```sh
poetry run python src/main.py --ticker NVDA,AAPL,MSFT,AMZN,GOOGL
```

### Comparing Multiple Stocks

To choose between several investment candidates:
```sh
poetry run python src/main.py --ticker GOOGL,MSFT,TSLA
```

Use 3-5 focused analysts (Druckenmiller, Buffett, Lynch) for faster, cheaper analysis.

### Validating a Stock Pick

Before buying a stock, get multiple perspectives:
```sh
poetry run python src/main.py --ticker NVDA
```

Select all 18 analysts for comprehensive analysis (~$0.14).

## File Structure
```
ai-hedge-fund/
├── src/
│   ├── main.py           # Main analysis script
│   ├── backtester.py     # Backtesting engine
│   ├── agents/           # Individual analyst implementations
│   └── tools/            # Financial data APIs
├── .env                  # Your API keys (DO NOT commit to git)
├── .env.example          # Template for .env file
├── pyproject.toml        # Poetry dependencies
└── poetry.lock           # Locked dependency versions
```

## Troubleshooting

### Poetry command not found

Add Poetry to your PATH:
```sh
export PATH="/home/mark/.local/bin:$PATH"
```

Make it permanent by adding to ~/.bashrc or Fish config.

### API key errors

Verify your key is set correctly:
```sh
cat .env | grep ANTHROPIC_API_KEY
```

Ensure there are no extra spaces or quotes around the key.

### Insufficient data errors

You're trying to analyze a stock outside the free tier (AAPL, GOOGL, MSFT, NVDA, TSLA). Either:
- Stick to the 5 free stocks
- Get a Financial Datasets API key from financialdatasets.ai

### Poetry install fails

Update Poetry:
```sh
poetry self update
```

Then retry:
```sh
poetry install
```

## Additional Resources

### Finding Top Performing Stocks

Use SlickCharts to see S&P 500 performance rankings:
- Visit: https://www.slickcharts.com/sp500/performance
- Sort by YTD Return to find the real performance drivers
- Many top performers are NOT the largest holdings

### Official Documentation

- AI Hedge Fund GitHub: https://github.com/virattt/ai-hedge-fund
- Anthropic API Docs: https://docs.anthropic.com
- Financial Datasets: https://financialdatasets.ai

## Best Practices

1. Start with the 5 free stocks to learn the system
2. Use 3-5 analysts for routine analysis (cheaper, faster)
3. Use all 18 analysts only when making major decisions
4. Focus on stocks with STRONG signals (70%+ confidence)
5. Ignore SHORT signals unless you understand shorting
6. Track recommendations over time to validate accuracy
7. Never invest based solely on AI recommendations
8. Use the tool as one input among many

## Limitations

- This is an EDUCATIONAL tool, not investment advice
- Past performance does not guarantee future results
- The system does not execute real trades
- AI agents are simulations of investment philosophies, not the actual investors
- Financial data may lag by days or weeks
- The tool cannot predict black swan events or breaking news

## Security Notes

The .env file contains your API keys. Never:
- Commit .env to git (it's in .gitignore by default)
- Share your .env file
- Post API keys publicly
- Use production keys for testing

Anthropic charges based on actual usage. Monitor your API usage at console.anthropic.com.

## Next Steps

After installation and testing:
1. Document findings in your own system (Jekyll site, notes, etc.)
2. Track AI recommendations against actual performance
3. Develop your own screening criteria based on lessons learned
4. Use the tool to validate stocks you're already considering
5. Consider getting Financial Datasets API key if analyzing many stocks

Remember: The AI Hedge Fund is a tool for learning and validation, not a replacement for your own research and judgment.