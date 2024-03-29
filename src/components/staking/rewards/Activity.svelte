<script>
  import { eth } from '../../../stores/eth.js';
  import { formatFiat } from '../../../components/helpers.js';
  import { toNum } from '../../../helpers/staking.js';
  import images from '../../../config/images.json';
  import epochsJSON from '../../../config/epochs.json';
  import BigNumber from 'bignumber.js';

  export let timestamp;
  let currentAddress;
  let epoch;
  let currentAccount;
  let votingPower;
  let accountWithdrawnRewards;

  $: if ($eth.address) {
    // if address is first setup, or is changed...
    if (currentAddress !== $eth.address) {
      currentAddress = $eth.address;

      // TODO: this should be changed, fetch the epochs from backend
      epoch = epochsJSON.epochs.find(
        (epoch) => epoch.startDate <= timestamp && epoch.endDate >= timestamp,
      );

      currentAccount = epoch.merkleTree.leafs.find((leaf) => leaf.staker.id == $eth.address);

      if(currentAccount) {
        currentAccount.participant = epoch.participants.find(
          (participant) => participant.address == $eth.address,
        );  
        
        let accountVeTokenBalance = new BigNumber(currentAccount.staker.accountVeTokenBalance);
        let veTokenTotalSupply = new BigNumber(epoch.stakingStats.veTokenTotalSupply);
        votingPower = accountVeTokenBalance.times(100).div(veTokenTotalSupply).toFixed(2);

        accountWithdrawnRewards = new BigNumber(currentAccount.staker.accountWithdrawnRewards);
        accountWithdrawnRewards = accountWithdrawnRewards.times(epoch.slice.usd);        
      }

      console.log("account has changed, currentAccount is now", currentAccount);
    }
  }
</script>

{#if $eth.address}
  <div class="flex flex-col items-center w-full md:w-1/2 p-1px bg-lightgrey rounded-16 m-10px">
    <div class="flex flex-col nowrap w-96pc m-2pc swap-from rounded-20px bg-white p-16px">
      <div class="font-huge text-left pb-4">Your Activity</div>

      {#if currentAccount}
        <div class="flex flex-row p-1 justify-between items-center">
          <div class="flex items-center">
            <span class="font-thin">Rewards value USD</span>
          </div>
          <div class="flex flex-col items-right">
            <div class="font-24px">
              {formatFiat(toNum(accountWithdrawnRewards), ',', '.', '$')}
            </div>
          </div>
        </div>

        <div class="text-l text-left pt-4">Slice Breakdown</div>
        {#each epoch.slice.underlying as underlying}
          <div class="flex flex-row p-1 justify-between items-center">
            <div class="flex items-center">
              <img
                class="h-auto w-24px mr-10px"
                src={images.logos[underlying.address]}
                alt={underlying.symbol}
              />
              <span class="token-symbol-container font-thin">{underlying.symbol}</span>
            </div>
            <div class="flex flex-col items-right">
              <div class="">
                {underlying.amount}
              </div>
            </div>
          </div>
        {/each}

        <div class="text-l text-left pt-4">Your Participation</div>
        <div class="flex flex-row p-1 justify-between items-center">
          <div class="flex items-center">
            <img class="h-auto w-24px mr-10px" src={images.veDough} alt="dough token" />
            <span class="token-symbol-container font-thin">Your veDOUGH</span>
          </div>
          <div class="flex flex-col items-right">
            <div class="">
              {formatFiat(toNum(currentAccount.staker.accountVeTokenBalance), ',', '.', '')}
            </div>
          </div>
        </div>

        <div class="flex flex-row p-1 justify-between items-center">
          <div class="flex items-center">
            <span class="token-symbol-container font-thin">Voting Power</span>
          </div>
          <div class="flex flex-col items-right">
            <div class="">
              {Number(votingPower)}%
            </div>
          </div>
        </div>

        {#each epoch.proposals as proposal}
          <div class="flex flex-row p-1 justify-between items-center">
            <div class="flex items-center">
              <span class="token-symbol-container font-thin">{proposal.title}</span>
            </div>
            <div class="flex flex-col items-right">
              <div class="">
                {#if currentAccount.participant.votes.find((vote) => vote.proposal == proposal.id)}
                  Voted
                {:else}
                  N/A
                {/if}
              </div>
            </div>
          </div>
        {/each}
      {:else}
        <div class="text-center p-1">Sorry, there's no data for this account.</div>
      {/if}
    </div>
  </div>
{/if}
