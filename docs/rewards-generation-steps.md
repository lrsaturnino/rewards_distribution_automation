# Staking rewards process [DAO]
## 1. Generating the new Merkle tree

1. Run the `gen_merkle_dist.js` script setting up the [distribution parameters](https://github.com/threshold-network/merkle-distribution/blob/f06fd162882a1144e45c0effdd857cb4563ee6c6/src/scripts/gen_rewards_dist.js#L12-L23). The `preWeight` and `tbtcv2Weight` parameters can be found [here](https://forum.threshold.network/t/tip-032-council-decision-reward-allocation-between-tbtcv2-pre-overall-target-apy-15/393/4), although it is not expected to change in the future (TACo 25%, tBTC 75%). Redirecting the output to a file `output.log` will make the results easier to check. This file will contain the rewards earned by each stake during this period.
    
    ```bash
    node src/scripts/gen_rewards_dist.js > output.log
    ```
    
2. Check the JSON file generated in `tBTCv2-rewards-details` folder. Some of the checks that can be performed are:
    1. Instances with a `upTimePercent` > 96% must have `isUptimeSatisfied` set to `true`. But those with a value < 96% must have it to `false`.
    2. Instances that run an invalid version of the client must have `isVersionSatisfied` to `false`. Even if they run the invalid version for a brief time.
    3. If int the current rewards period there are more than 1 valid client version, the timestamp  of the versions run by the instances must be less than the deadline timestamp. For example, if a version is valid until 1720000000, the reported timestamp of this version of a node must be lower than this timestamp.
3. Check the calculations for some stakes, calculating manually how much T they should earn. DAO stakes should be checked at this point. To perform this check:
    1. Take the staking provider address of a specific stake.
    2. Check how much T is authorized to the apps (RandomBeacon, tBTC, TACo). For this, you can use this [subgraph query](https://cloud.hasura.io/public/graphiql?endpoint=https%3A%2F%2Fsubgraph.satsuma-prod.com%2F276a55924ce0%2Fnucypher--102994%2Fmainnet%2Fapi&query=query+MyQuery+%7B%0A++stakeData%28id%3A+%220x58d665406cf0f890dad766389df879e84cc55671%22%29+%7B%0A++++id%0A++++authorizations+%7B%0A++++++amount%0A++++++appName%0A++++%7D%0A++%7D%0A%7D%0A). Note that subgraph query only accepts **lowercased** addresses.
    3. Insert the staked amount and adjust the start and end dates on this [DAO Stake rewards check spreadsheet](https://docs.google.com/spreadsheets/d/16qLjST2enH6td0uxzuo7LyWJ-jps63vd_owGhbnVqr4/edit?usp=sharing).
    4. Check if the numbers estimated by the spreadsheet roughly matches with the numbers returned on the `output.log`.
4. Verify the generated Merkle root is valid by calling `node src/scripts/verify_merkle_dist.js <distribution_date>`. For example: `node src/scripts/verify_merkle_dist.js 2024-11-01`. This script should return `true`.
5. Create a new commit with the changes in `distributions` folder and in the `gen_rewards_dist.js` script. It is not necessary to add the `output.log` file. Push the commit to a new branch in remote repository.
6.  Create a new PR on `threshold-network/merkle-distribution` repo. Follow the format of this commit message: [example of PR for the Oct 1st 24 distribution](https://github.com/threshold-network/merkle-distribution/pull/158). Add as reviewer to someone from Keep (@lukasz-zimnoch) and someone from NuCypher (@manumonti). Wait until at least one of each team have approved the PR. Keep people will check Keep stakes and NuCo people will check NuCo stakes.
7. Once the PR is reviewed, just merge it.

## 2. Updating the dashboard

1. Create a new PR in token-dashboard repo like this: https://github.com/threshold-network/token-dashboard/pull/718. **The base branch for this PR must be the latest release branch** `releases/mainnet/v1.yy.zz`. The last release version can be checked here https://github.com/threshold-network/token-dashboard/releases. Also, add people from Keep as reviewers (@michalsmiarowski).
2. When the PR is merged, bump the patch version in package.json. Example: [**Update T Token Dashboard version to v1.15.1**a86c130](https://github.com/threshold-network/token-dashboard/commit/a86c130c42460dac3c50ee57857a57c44880ea24). This must be done directly by pushing the commit to the release branch, **no PR is needed**.
3. Create a tag for this commit that bumps the patch version: `git tag -a v1.15.1 -m "v1.15.1"` and then `git push origin --tags`.
4. Create a [new Milestone](https://github.com/threshold-network/token-dashboard/milestones). The title must be the new release version. No “due date” nor “description” needed. Use this one as reference: https://github.com/threshold-network/token-dashboard/milestone/36. **Add the PR that has been merged to this Milestone.**
5. Create a **DRAFT** release notes. This is by clicking “Draft a new release” in [Releases](https://github.com/threshold-network/token-dashboard/releases). Use [this release](https://github.com/threshold-network/token-dashboard/releases/tag/v1.15.1) as reference. The commit hash must be the one of the version bump. The milestone link is the link to the milestone created earlier. **Save** the draft release (**DON’T PUBLISH YET, just save it**).
6. Use the tool Crypto on Keybase app to **Sign** the hash of the commit. Go to `threshold_org` chat on Keybase and share the signature with this message:
    
    ```
    Starting release signing party for v1.15.2 Threshold Token Dashboard.
    
    The commit hash is `5a652fb199fdc5e80d9475094238c4a9c2e177e0`
    
    My signature for `5a652fb199fdc5e80d9475094238c4a9c2e177e0`:
    
    ```
    BEGIN KEYBASE SALTPACK SIGNED MESSAGE. kXR7VktZdyH7rvq v5weRa0zkY6Uyez YqdubbtbSlVyMrt 6aOprCwfq8LShUp LS1yoDz2B3DrQhZ tnSlEnB52Z3Yftw wUc1FLg8K7Evp77 CUIslWgBmY3bkKM bsH1XAIjSIwPVcO HBGPDrwu1xNg2lQ a8roE7LVpcB71Ff ffJ9h1hTmc82Awa opfjJvxfMOoNRqg rHeHwflmKLNZAOF YHzhV0P1O53Md7s LilnkunBMtL4f9E . END KEYBASE SALTPACK SIGNED MESSAGE.
    ```
    ```
    
    Also, wait for someone to confirm your signature and also to sign the message. Confirm its signature as well.
    
7. Add these signatures to the release draft. The usernames (@manumonti, @michalsmiarowski) must be a link to Keybase users (https://keybase.io/manumonti).
8. Once the release message is ready, **publish it**.
9. Someone with permissions (Michal, Lukasz, David or Maclane), must approve the deploy. You can check the status here: https://github.com/threshold-network/token-dashboard/actions
10. When the production dashboard is released, and ASAP, the Merkle root must be updated in the Merkle Distribution contract. Use the method setMerkleRoot for this: [setMerkleRoot](https://etherscan.io/address/0xeA7CA290c7811d1cC2e79f8d706bD05d8280BD37#writeContract#F4).
11. Tell Luna to post an announcement in Threshold Discord. The new distribution of rewards is available.
12. Do a backport. For this, `git switch` to `main` branch, pull the last changes, and create a new branch with the name `backport-<backported-branch>`. For instance: `backport-rewards-jan-1-2024`. Do a `git cherry-pick` of the commit with the change in `rewards.json`. Create a PR like this: https://github.com/threshold-network/token-dashboard/pull/720. Add this PR to the Milestone.

## 3. The Council authorizes the spending of rewards

> Note: this process should be carried out bimonthly instead of monthly to avoid burdening the council frequently.
> 

Using the [staking rewards proxy](https://github.com/threshold-network/staking-rewards-proxy) architecture we want to authorize the spending of the T rewards by the MerkleDistribution contract.

So we have to create a transaction batch in Safe Multisig using the Transaction Builder.

<aside>
➡️

Since Nov 23, the Safe MS used is one owned by all the council members plus an address for the Threshold Foundation: https://app.safe.global/home?safe=eth:0xf642Bd6A9F76294d86E99c2071cFE2Aa3B61fBDa

</aside>

We have to create a transaction with two actions:

- Action 1: This will approve the spending of an amount of T stored in [FutureRewardsProxy](https://etherscan.io/address/0xbe3e95dc12c0ae3fac264bf63ef89ec81139e3df) by [ClaimableRewardsProxy](https://etherscan.io/address/0xec8183891331a845e2dcf5cd616ee32736e2baa4). This amount is the reward generated in this distribution. This is necessary because, in Action 2, `ClaimableRewardsProxy` will transfer this T from `FutureRewardsProxy` to `ClaimableRewardsProxy`.
- Action 2: This action will: 1. transfer the T from `FutureRewardsProxy` to `ClaimableRewardsProxy`, and 2. approve the spending of this T by the Merkle Distribution contract.

A good example of this authorization is Transaction #2 in the multisig: https://app.safe.global/transactions/tx?safe=eth:0xf642Bd6A9F76294d86E99c2071cFE2Aa3B61fBDa&id=multisig_0xf642Bd6A9F76294d86E99c2071cFE2Aa3B61fBDa_0xbbc3521ae25eb5988b9a10a54a3b6e1b579e3fbe8e6698b184f5b495967172e5

1. Calculate the total amount of rewards calculated for this distribution. This is the difference between the new rewards distribution and the last one: https://github.com/threshold-network/merkle-distribution/blob/main/distributions/distributions.json
2. Open the [Transaction Builder](https://app.safe.global/apps/open?safe=eth:0x9F6e831c8F8939DC0C830C6e492e7cEf4f9C2F5f&appUrl=https%3A%2F%2Fapps.gnosis-safe.io%2Ftx-builder) app in Safe.
3. In the first transaction, enter the address of FutureRewardsProxy contract: `0xbe3e95Dc12C0aE3FAC264Bf63ef89Ec81139E3DF`. The web should be updated and show the contract’s ABI. Fill the following fields:
    
    
    | To Address | 0xbe3e95Dc12C0aE3FAC264Bf63ef89Ec81139E3DF |
    | --- | --- |
    | ETH Value | 0 |
    | Contract Method Selector | execute |
    | _target (address) | 0xa604C363d44e04da91F55E6146D09ecDD004f678 |
    | _data (bytes)* | 0x45e8bcf8 […] |
    - The `_target` field is the address of [ProxyLogicV1 contract](https://etherscan.io/address/0xa604c363d44e04da91f55e6146d09ecdd004f678).
    - The `_data` field is the calldata to `approveT` method in ProxyLogicV1 contract. To get it we can use `cast` CLI (part of Foundry):
    
    ```bash
    cast calldata "approveT(address,uint256)" 0xec8183891331a845e2dcf5cd616ee32736e2baa4 <T distribution rewards>
    ```
    
4. In the second transaction, enter the address of ClaimableRewardsProxy contract: `0xec8183891331a845E2DCf5CD616Ee32736E2BAA4`. The web should be updated and show the contract’s ABI. Fill the following fields:
    
    
    | To Address | 0xec8183891331a845E2DCf5CD616Ee32736E2BAA4 |
    | --- | --- |
    | ETH Value | 0 |
    | Contract Method Selector | execute |
    | _target (address) | 0xE9ec5e1c6956625D2F3e08A46D9f5f4c62B563f7 |
    | _data (bytes)* | 0xece8f2de […] |
    - The `_target` field is the address of [ProxyLogicV2 contract](https://etherscan.io/address/0xe9ec5e1c6956625d2f3e08a46d9f5f4c62b563f7).
    - The `_data` field is the calldata to `topUpClaimableRewards` method in ProxyLogicV2 contract. To get it we can use `cast` CLI.
    
    ```bash
    cast calldata "topUpClaimableRewards(uint256)" <T distribution rewards>
    ```
    
5. Click on `Create Batch`.
6. Click on `Simulate` to run a Tenderly simulation. Check everything worked fine in the simulation.
7. Save the JSON with the transactions batch and send it together with the simulation to the Council.