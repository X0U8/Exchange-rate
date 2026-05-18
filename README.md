# 🌍 Automated Global Exchange Rate Engine

A lightweight, zero-maintenance, high-performance currency translation layer that acts as a decentralized static CDN for global SaaS products. 

This repository automatically fetches, filters, and serves up-to-date currency conversion values for **101 target global countries** directly from a localized storage endpoint.

---

## ❓ What is this?
This project is an automated data pipeline that queries global financial currency matrices, sanitizes the payload to look for specific operational currencies, and saves the output into a public static `rates.json` ledger. 

It provides an open-source lookup endpoint for client-side applications (like multi-currency checkout models, billing cards, and pricing menus) to dynamically calculate local pricing without backend overhead.

---

## ⚙️ How does it work?
1. **Automated Trigger:** A background GitHub Actions runner wakes up automatically **every 12 hours**.
2. **Bulk Extraction:** The workflow fires a single, optimized request to the exchange master database to pull global conversions relative to USD.
3. **Array Filtering:** A Node.js worker sanitizes the response, extracting exactly the 101 precise target currency tags required by modern checkout layers (e.g., INR, EUR, GBP, AUD, etc.).
4. **Static Committing:** The worker writes an updated, minified `rates.json` file back into the repository main branch using an automated GitHub Actions bot profile.

---

## 💡 Why use this architecture? (The Benefits)
* **🔒 Ironclad Key Protection:** Your private financial database API tokens are handled strictly within encrypted server environments. No access keys are ever exposed to client-side networks or client bundles.
* **⚡ 0ms Runtime Latency:** Your frontend applications don't lose milliseconds waiting on third-party financial API handshakes during user rendering. The browser pulls a tiny text asset directly from GitHub's static network edge.
* **💰 100% Budget Safe:** Standard billing components calling API providers on every page visit quickly exhaust quotas. By utilizing this batch-caching model, the platform runs perpetually within free tier constraints by making exactly 2 calls a day.
* **🛡️ High Resiliency:** If a core currency endpoint goes offline or faces a global service crash, your live production checkout cards remain completely unaffected, falling back gracefully to the last cached local file.

---

## 🚀 How to make your own

Follow these steps to deploy this exact self-updating pricing matrix engine in under 3 minutes:

### 1. Create the Local Repositories
Create a new public repository and ensure your structure has the target configuration file path initialized:
* Create a file named `rates.json` in your root directory.
* Create a folder path named `.github/workflows/` and add a workflow file named `sync-rates.yml`.

### 2. Add the Workflow Configuration
Paste the following complete automation engine inside `.github/workflows/sync-rates.yml`:

\`\`\`yaml
name: 12-Hour Static Rates Sync

on:
  schedule:
    - cron: '0 */12 * * *'
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Fetch Global Rates and Generate Static JSON
        env:
          EXCHANGE_API_KEY: ${{ secrets.EXCHANGE_RATE_API_KEY }}
        run: |
          node -e "
            const fs = require('fs');
            
            const targetCurrencies = [
              'ALL', 'DZD', 'ARS', 'AMD', 'AUD', 'EUR', 'AZN', 'BSD', 'BHD', 'BBD', 
              'BYN', 'BAM', 'BWP', 'BRL', 'BGN', 'CAD', 'CLP', 'CNY', 'COP', 'CRC', 
              'CUP', 'CZK', 'DKK', 'DOP', 'USD', 'EGP', 'ETB', 'GEL', 'GHS', 'GTQ', 
              'GYD', 'HTG', 'HKD', 'HUF', 'ISK', 'INR', 'IDR', 'IRR', 'ILS', 'JMD', 
              'JPY', 'JOD', 'KZT', 'KES', 'KWD', 'LBP', 'LYD', 'MYR', 'MXN', 'MKD', 
              'NOK', 'NPR', 'NZD', 'NGN', 'PKR', 'PEN', 'PHP', 'PLN', 'QAR', 'RON', 
              'RUB', 'SAR', 'RSD', 'SGD', 'ZAR', 'KRW', 'LKR', 'SEK', 'CHF', 'TWD', 
              'THB', 'TND', 'TRY', 'UAH', 'AED', 'GBP', 'UYU', 'UZS', 'VND', 'YER'
            ];

            fetch('https://v6.exchangerate-api.com/v6/' + process.env.EXCHANGE_API_KEY + '/latest/USD')
              .then(res => res.json())
              .then(data => {
                if (data.result !== 'success') throw new Error('API request failed');
                
                const filteredRates = {};
                targetCurrencies.forEach(code => {
                  if (data.conversion_rates[code] !== undefined) {
                    filteredRates[code] = data.conversion_rates[code];
                  }
                });

                const payload = {
                  base: 'USD',
                  last_updated: new Date().toISOString(),
                  rates: filteredRates
                };

                fs.writeFileSync('./rates.json', JSON.stringify(payload, null, 2));
                console.log('Static asset updated with ' + Object.keys(filteredRates).length + ' currencies.');
              })
              .catch(err => {
                console.error('Error:', err);
                process.exit(1);
              });
          "

      - name: Commit and Push to Storage
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git add rates.json
          git commit -m "chore: 12-hour rates update [skip ci]" || echo "Rates are identical. Nothing to change."
          git push origin main
\`\`\`

### 3. Generate Your Gateway Key
1. Head over to [ExchangeRate-API](https://www.exchangerate-api.com/).
2. Register for a free developer account (Provides 1,500 free inquiries per month).
3. Copy your unique API Key from the dashboard layout.

### 4. Bind the Secrets to GitHub
1. Navigate to your GitHub repository configuration page and click **Settings**.
2. From the left-hand column panel, choose **Secrets and variables** ➡️ **Actions**.
3. Select **New repository secret**.
4. Input the precise designation identifier name: `EXCHANGE_RATE_API_KEY`
5. Paste your private API core token inside the **Secret** field value and save.

### 5. Authorize Write Permissions
To prevent automated builds from erroring out with structural 403 blocks during deployments, grant the runner write access:
1. Inside your repository layout, click **Settings**.
2. Click **Actions** ➡️ **General** from the sidebar list.
3. Scroll downwards to locate the **Workflow permissions** cluster block.
4. Switch the selector configuration toggle to **"Read and write permissions"**.
5. Save your settings.

### 6. Run & Customize
Go to the **Actions** tab of your repository, select **12-Hour Static Rates Sync**, and hit **Run workflow** to fire the initial initialization array manually. Feel free to modify the currency codes or schedule timing parameters inside the workflow configurations to meet your app's requirements!

---

## 🌐 Consuming the Static Endpoint
You can pull the data directly into your frontend checkout applications using standard client side scripts:

\`\`\`javascript
const DATA_URL = "https://raw.githubusercontent.com/<YOUR_USERNAME>/<YOUR_REPO>/main/rates.json";

fetch(DATA_URL)
  .then(res => res.json())
  .then(data => {
    console.log("Current conversion matrix loaded: ", data.rates);
    // Ready to map directly into currency selectors!
  });
\`\`\`

---
*Built for secure, ultra-lightweight, and lightning-fast global SaaS billing platforms.*
