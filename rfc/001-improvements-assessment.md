# Proposed Improvements to the Monthly Rewards Distribution Process
## Introduction
This document outlines a series of potential improvements to the current monthly rewards distribution workflow. These improvements are intended to streamline operations, reduce manual intervention, and improve reliability, security, and auditability of the system. The goal is to help other developers in the team understand the rationale behind these enhancements and prepare to review and potentially implement them.
## Current Challenges
- Manual Execution: The entire process—from generating the Merkle tree to updating the dashboard—requires manual steps each month.
- Time-Consuming Verifications: Frequent recalculations and checks are needed to ensure rewards data is accurate.
- Complex Dashboard Updates: Updating the dashboard and ensuring the front-end reflects the latest rewards involves multiple manual steps and versioning processes.
## Proposed Improvements
### 1. Automated Script Execution via a VPS and Background Service
Set up a dedicated Virtual Private Server (VPS) or a cloud-based instance (e.g., AWS EC2) to run the rewards generation script (gen_rewards_dist.js) at scheduled intervals. Use a cron job or a systemd service to ensure automatic execution, logging, and alerting.

Implementation Details:
```
   Systemd Service Unit Example:
   [Unit]
   Description=Threshold Monthly Rewards Generation Service
   After=network.target

   [Service]
   WorkingDirectory=/opt/threshold-rewards
   ExecStart=/usr/bin/node /opt/threshold-rewards/src/scripts/gen_rewards_dist.js
   Restart=on-failure
   User=threshold

   [Install]
   WantedBy=multi-user.target
```

Cron Job:
```
   //Run at 00:00 UTC on the 1st of each month
   0 0 1 * * /usr/bin/node /opt/threshold-rewards/src/scripts/gen_rewards_dist.js >> /var/log/threshold-rewards.log 2>&1
```
Estimate:
- VPS setup & configuration: ~6 hours (including environment setup)
- CI/CD integration & monitoring: ~8 hours
- Total: ~14 hours

### 2. Additional Script for Automated Rewards Calculations Validation
Introduce a separate script to validate the correctness of the generated rewards. This script checks the JSON output file to ensure reward conditions (uptime, version compliance) are met, and spot-checks some calculations. It could also compare values against a known formula or prior month’s data.

Implementation Details:
```
   // validate_rewards.js
   const fs = require('fs');

   function validateRewards(filePath) {
     const data = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
     
     let isValid = true;
     for (const stake of data.stakes) {
       // Check uptime condition
       if (stake.upTimePercent > 96 && !stake.isUptimeSatisfied) {
         console.error(`Stake ${stake.id} should have isUptimeSatisfied = true`);
         isValid = false;
       }
       if (stake.upTimePercent < 96 && stake.isUptimeSatisfied) {
         console.error(`Stake ${stake.id} should have isUptimeSatisfied = false`);
         isValid = false;
       }

       // Check version compliance (example logic)
       if (!stake.isVersionSatisfied) {
         console.warn(`Stake ${stake.id} is running invalid version.`);
       }

       // Spot-check reward amounts (example calculation)
       const expectedReward = (stake.authorizedT / 1_000_000) * 0.25; // Example formula
       if (Math.abs(stake.reward - expectedReward) > expectedReward * 0.05) {
         console.error(`Stake ${stake.id} reward differs significantly from expected.`);
         isValid = false;
       }
     }

     return isValid;
   }

   // Usage: node validate_rewards.js path/to/rewards.json
   const filePath = process.argv[2];
   if (!filePath) {
     console.error("Please provide the path to the rewards JSON file.");
     process.exit(1);
   }

   const result = validateRewards(filePath);
   process.exit(result ? 0 : 1);
```
Estimate:
- Script development & integration: ~12 hours
- Testing & refinement: ~6 hours
- Total: ~18 hours

### 3. API-Based Data Distribution for the Dashboard
Provide a simple API endpoint that always returns the latest rewards distribution data. The dashboard front-end then queries this API instead of manually updating files and versions. This can be done via a simple Node.js server, serverless function, or static file hosting on S3 with a known URL.

Implementation Details:
```
   const express = require('express');
   const fs = require('fs');
   const path = require('path');

   const app = express();

   // Suppose the latest rewards file is always named 'latest_rewards.json'
   app.get('/rewards/latest', (req, res) => {
     const rewardsData = fs.readFileSync(path.join(__dirname, 'latest_rewards.json'), 'utf-8');
     res.setHeader('Content-Type', 'application/json');
     res.send(rewardsData);
   });

   app.listen(3000, () => {
     console.log('Rewards API server running on port 3000');
   });
```
Steps:
- Run gen_rewards_dist.js on VPS -> Generates new_rewards.json.
- After validation, rename new_rewards.json to latest_rewards.json.
- The front-end dashboard fetches GET /rewards/latest and displays the data.

Estimate:
- API server setup & deployment: ~12 hours
- Front-end integration: ~8 hours
- Testing & rollout: ~4 hours
- Total: ~24 hours

## Conclusion
By implementing these three main improvements—automated script execution on a VPS, a dedicated validation script for rewards calculations, and an API-based approach for serving reward data to the dashboard—we can significantly reduce manual labor, minimize human error, and ensure timely, accurate, and consistent updates for our stakeholders.


