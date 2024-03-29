<script>
  import { _ } from 'svelte-i18n';
  import images from '../../config/images.json';
  import { formatFiat } from '../helpers.js';
  import * as animateScroll from 'svelte-scrollto';
  import BoostedModal from '../elements/BoostedModal.svelte';
  import UnlockModal from '../elements/UnlockModal.svelte';
  import {
    toNum,
    calculateStakingEnds,
    calculateStakingStarts,
    calculateVeDough,
    boostToMax,
    getLockStatus,
    didLockExpired,
    unstakeDOUGH,
    AVG_SECONDS_MONTH,
    canRestake
  } from '../../helpers/staking.js';
  import { justBoosted, timestampBoosted, stakingData } from '../../stores/eth/writables';
  import { eth } from '../../stores/eth.js';
  
  export let scrollToTop;
  export let itemsNumber;
  
  let isLoading = true;
  let boostedModal;
  let unlockModal;

  $: if($stakingData && $stakingData.hasLoaded) {
    if(!itemsNumber) {
      itemsNumber = $stakingData.accountLocks.length;
    }

    isLoading = false;
  }
</script>

<BoostedModal bind:this={boostedModal}/>
<UnlockModal bind:this={unlockModal}/>

{#if $eth.address}
  <div class="flex flex-col items-center w-full pb-6 bg-lightblu rounded-16 mt-6">
    <div class="font-huge text-center mt-6">Your Staking</div>

    {#if isLoading}
      Loading...
    {:else if $stakingData.accountLocks && $stakingData.accountLocks.length > 0}
      {#each $stakingData.accountLocks.slice(0, itemsNumber) as lock, id}
        <!-- Let's show just the normal stakes, and the boosted ones. The stakes having a boostedPointer are obsolete stakes, so we won't show them -->
        {#if lock.boostedPointer == ''}
          <div
            class={lock.ejected
              ? 'flex flex-col nowrap w-92pc mx-4pc mt-6 swap-from rounded-20px bg-white p-1px opacity-60'
              : $justBoosted[lock.lockId]
              ? 'flex flex-col nowrap w-92pc mx-4pc mt-6 swap-from rounded-20px bg-white p-1px opacity-40'
              : $timestampBoosted[lock.lockId] && $timestampBoosted[lock.lockId] < lock.lockedAt
              ? 'fade-in-1 flex flex-col nowrap w-92pc mx-4pc mt-6 swap-from rounded-20px bg-white p-1px cardbordergradient'
              : 'flex flex-col nowrap w-92pc mx-4pc mt-6 swap-from rounded-20px bg-white p-1px'}
          >
            <div class="p-4">
              <div class="flex items-center justify-between">
                <div class="flex intems-center">
                  <div class="font-thin mr-2">Start:</div>
                  <span
                    >{calculateStakingStarts(lock)
                      .toDateString()
                      .replace(/^\S+\s/, '')}</span
                  >
                </div>
                <div class="flex items-center">
                  <div class="font-thin mr-2">End:</div>
                  <span
                    >{calculateStakingEnds(lock)
                      .toDateString()
                      .replace(/^\S+\s/, '')}</span
                  >
                </div>
              </div>

              <div class="flow-root md:flex nowrap items-center p-1 justify-between mt-2">
                <div class="grid grid-flow-col grid-cols-1 grid-rows-2">
                  <div class="gjVeBU">
                    <div class="font-24px">{formatFiat(toNum(lock.amount), ',', '.', '')}</div>
                    <img class="h-auto w-24px mx-5px" src={images.doughtoken} alt="dough token" />
                    <span class="sc-kXeGPI jeVIZw token-symbol-container">DOUGH</span>
                  </div>
                  <div class="gjVeBU float-left">
                    <div class="font-24px">
                      {formatFiat(
                        calculateVeDough(lock.amount, lock.lockDuration / AVG_SECONDS_MONTH),
                        ',',
                        '.',
                        '',
                      )}
                    </div>
                    <img class="h-auto w-24px mx-5px" src={images.veDough} alt="dough token" />
                    <span class="sc-kXeGPI jeVIZw token-symbol-container">veDOUGH</span>
                  </div>
                </div>
                {#if (!lock.boosted && canRestake(lock.lockedAt)) || (lock.boosted && canRestake(lock.lockedAt))}
                  {#if !lock.ejected && !lock.withdrawn}
                    <button
                      disabled={$justBoosted[lock.lockId]}
                      on:click={() => {
                        // marking the lock as justBoosted...
                        $justBoosted[lock.lockId] = true;

                        // saving the timestampBoosted for further uses...
                        let boostingTimestamp = Math.floor(Number(Date.now()) / 1000);

                        boostToMax(lock.lockId, $eth)
                          .then((updated_data) => {
                            $timestampBoosted[updated_data.accountLocks[0].lockId] = boostingTimestamp;

                            if(scrollToTop) {
                              animateScroll.scrollToTop();
                            }

                            boostedModal.showModalLock(lock);

                            setTimeout(() => {
                              $timestampBoosted[$stakingData.accountLocks[0].lockId] = null;
                              $justBoosted[lock.lockId] = false;
                            }, 15000);
                          })
                          .catch((error) => {
                            $justBoosted[lock.lockId] = false;
                            console.error(error);
                          });
                      }}
                      class="flex items-center cardbordergradient -mr-2 pointer mt-2 ml-15pc md:ml-0"
                    >
                      <div class="flex items-center p-2">
                        {#if $justBoosted[lock.lockId]}
                          <div class="mr-8px">Loading...</div>
                        {:else}
                          <div class="mr-8px">Restake 3 years</div>
                        {/if}
                        <img
                          class="w-30px h-30px"
                          src="https://raw.githubusercontent.com/pie-dao/brand/master/PIE%20Tokens/RewardPie.png"
                          alt="ETH"
                        />
                      </div>
                    </button>
                  {/if}
                {:else}
                  <div class="flex items-center cardbordergradient mt-2 ml-15pc md:ml-0 md:-mr-2 opacity-30">
                    <div class="flex items-center p-2">
                      <div class="mr-8px">Restake 3 years</div>
                      <img
                        class="w-30px h-30px"
                        src="https://raw.githubusercontent.com/pie-dao/brand/master/PIE%20Tokens/RewardPie.png"
                        alt="ETH"
                      />
                    </div>
                  </div>
                {/if}
              </div>

              <div class="flex items-center justify-between">
                <div class="flex intems-center">
                  <div class="mt-2 flex justify-start opacity-30 pointer">
                    <span>{getLockStatus(lock)}</span>
                  </div>
                </div>
                <div class="flex items-center">
                  {#if !lock.withdrawn && !lock.ejected}
                    {#if didLockExpired(lock)}
                      <div
                        on:click={() => {
                          unstakeDOUGH(lock.lockId, toNum(lock.amount), $eth)
                            .then((updated_data) => {
                              console.log("unstaked", updated_data);
                            })
                            .catch((error) => {
                              console.error(error.message);
                            });
                        }}
                        class="mt-2 flex justify-end pointer"
                      >
                        <span>Unstake</span>
                      </div>
                    {:else}
                      <div 
                      on:click={() => {
                        unlockModal.showModalUnock(lock, $stakingData.accountWithdrawnRewards);
                      }}
                      class="mt-2 flex justify-end opacity-30 pointer">
                        <span>Unstake</span>
                      </div>
                    {/if}
                  {/if}
                </div>
              </div>
            </div>
          </div>
        {/if}
      {/each}
    {:else}
      <span class="text-s text-center mx-8">You haven't staked anything yet, what are you waiting for?</span>
    {/if}

    {#if $stakingData.accountLocks.length > itemsNumber}
      <a class="pt-6" href="#/staking_positions"> See all staking positions </a>
    {/if}
  </div>
{/if}
