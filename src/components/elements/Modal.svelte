<script>
import {mousedownOutside} from '../../helpers/mousedownOutside.js';
import images from '../../config/images.json';

import { createEventDispatcher } from 'svelte';
const dispatch = createEventDispatcher();

export let backgroundColor;
export let title;
export let modalIsOpen = false;

export const open = () => {
    modalIsOpen = true;

    dispatch('modalChanged', {
      data: {isOpen: modalIsOpen},
    });    
}

export const close = () => {
    modalIsOpen = false;
    dispatch('modalChanged', {
      data: {isOpen: modalIsOpen},
    });     
}
</script>

{#if modalIsOpen}
    <div class="genericmodal flex justify-center items-center">
        <div style={ backgroundColor ? `background-color: ${backgroundColor} !important` : "#fff"} 
             class="flex flex-col justify-start modalcontent w-100pc min-h-100pc p-4 overflow-x-hidden overflow-y-auto lg:max-w-600px lg:min-w-30pc lg:max-h-70pc lg:min-h-25pc" 
             use:mousedownOutside 
             on:mousedown_outside={close}>
            <div class="flex">
             {#if title}
                <h1 class="pl-50px text-center text-xl w-100pc"> {title} </h1>
            {/if}
            <button on:click={close} class="w-30px h-30px self-center">
                <img src={images.closebutton} alt="closebutton" />
            </button>
        </div>
            <slot name="content">
                
            </slot>
        </div>
    </div>
{/if}