<script>
  import { connectWeb3, eth, isChachedProvider } from '../stores/eth.js';
  import { shortenAddress } from "@pie-dao/utils";
  const address = null;

  $: shortAddress = address ? shortenAddress(address) : '';

  $: ( async () => {
    if (isChachedProvider()) {
      connectWeb3();
    }
  })()

  const handleClick = () => {
    if (!$eth.address) {
      connectWeb3();
    }

    if ($eth.address) {
      window.location.hash = '#/piefolio';
      return;
    }      
  }
  
</script>

<button
  class="btn connect-button-container min-w-150px md:min-w-0"
  on:click={handleClick}
  role="button"
  tabIndex="-100"
>
  {#if $eth.address}
    {#if $eth.ens}
      <p>{$eth.ens}</p>
    {:else}
      <p>{$eth.shortAddress}</p>
    {/if}
    
    <div class="icon-container">
      <div class="image-container">
        {@html $eth.icon}
      </div>
    </div>
  {:else}
    {#if address}
      <p>{shortAddress}</p>
      🔌
    {:else}
      Connect Wallet
    {/if}
  {/if}
</button>