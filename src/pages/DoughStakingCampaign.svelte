<script>
  import BigNumber from "bignumber.js";
  import images from "../config/images.json";
  import { farming } from '../stores/eth/writables.js';
  import StakingStats from '../components/staking/Stats.svelte';
  import { LottiePlayer } from '@lottiefiles/svelte-lottie-player';
  import moment from 'moment';
  
  import {
    formatFiat,
    subscribeToBalance,
  } from "../components/helpers.js";

  import Meta from '../components/elements/meta.svelte';
   
   import {
    balanceKey,
    balances,
    contract,
  } from "../stores/eth.js";

    // lottie...
    let controlsLayout = [
    'previousFrame',
    'playpause',
    'stop',
    'nextFrame',
    'progress',
    'frame',
    'loop',
    'spacer',
    'background',
    'snapshot',
    'zoom',
    'info',
  ];
  
  const DOUGH = '0xad32A8e6220741182940c5aBF610bDE99E737b2D';
  const DAO = '0x4efD8CEad66bb0fA64C8d53eBE65f31663199C6d';
  const MULTISIG = '0x3bFdA5285416eB06Ebc8bc0aBf7d105813af06d0';

  let daoBalanceKey;
  let msBalanceKey;
  let doughStaked;
  let price = 'n/a';
  let circulatingSupply = 0;
  

  const addToken = () => {
    ethereum.sendAsync({
        method: 'wallet_watchAsset',
        params: {
          "type":"ERC20",
          "options":{
            "address": '0xad32A8e6220741182940c5aBF610bDE99E737b2D',
            "symbol": 'DOUGH',
            "decimals": 18,
            "image": 'https://raw.githubusercontent.com/pie-dao/brand/master/DOUGH%20Token/DOUGH2v.png',
          },
        },
        id: Math.round(Math.random() * 100000),
    }, (err, added) => {
      if (added) {
        console.log('Thanks for your interest!')
      } else {
        alert('Something went wrong. Is Metamask there?')
      }
    })
  };
  
  $: (async () => {

    const doughToken = await contract({ address: DOUGH });
    // DAO
    subscribeToBalance(DOUGH, DAO, true);
    // Multisig
    subscribeToBalance(DOUGH, MULTISIG, true);

    daoBalanceKey = balanceKey(DOUGH, DAO);
    msBalanceKey = balanceKey(DOUGH, MULTISIG);

    const totalSupply = await doughToken.totalSupply();
    const daoBal = $balances[daoBalanceKey] || BigNumber(0);
    const msBal = $balances[msBalanceKey] || BigNumber(0);
    doughStaked = $farming['0xB9a4Bca06F14A982fcD14907D31DFACaDC8ff88E'].doughStaked.toFixed(2) || 0;
    price = $farming['0xB9a4Bca06F14A982fcD14907D31DFACaDC8ff88E'].DOUGHPrice.toFixed(2) || 0;

    if(daoBal > 0 && msBal > 0) {
      const ts = BigNumber(totalSupply.toString()).dividedBy(10**18)
      circulatingSupply = ts.minus(BigNumber(msBal)).minus(BigNumber(daoBal)).toFixed(2);
    }
  })()

  // countdown component...
  let countdown;
  const end = moment('2021-10-04T13:00:00.000Z');

  setInterval(function() {
    countdown = end.fromNow()
  }, 1000);  
</script>

<Meta 
    metadata={{
        title: "Stake DOUGH, PieDAO Governance Token, and get a monthly income!",
        description: "Contribute and be rewarded every month for building a better organization and products",
        image: images.herodough,
        imageAlt: "Full-time Token Holder, the future of work is here."
    }}
/>

<div class="bgtokenholdercontainer">

  <div class="content flex flex-col spl px-4">
    <!-- <div class="flex flex-col items-center text-white font-thin">
      <span class="text-l">Staking started</span>
        <span class="text-xl">October 4th, 2021</span>
    </div> -->
    <img src={images.tokenholderherotype} class="crisp mt-8" alt="dough" data-aos="fade-up" data-aos-delay="150"/>
    <div class="text-lg text-white font-thin text-center mt-4 leading-8" data-aos="fade-up" data-aos-delay="200">Contribute and be rewarded<br />for building a better organization and products.</div>
    <button class="items-center stakinggradient shake text-black text-left mt-4 hover:opacity-80" onclick="location.href='#/swap';" data-aos="fade-up" data-aos-delay="250">
      <div class="w-100pc flex items-center">
      <div class="m-10px"><img class="h-50px inline" src={images.doughtoken} alt="doughtoken" /></div>
      <div class="mr-20px">
        <div class="text-base font-bold leading-5">Buy DOUGH</div>
        <div class="text-sm font-thin">Current price: <strong>${price}</strong></div>
      </div>
    </div>
    </button>
    <!-- <a class="text-bold text-center text-white mt-2 pointer" href="/#/dough-staking">Stake DOUGH</a> -->
    <button onclick="location.href='http://discord.link/PieDAO';" class="add-dough-metamask mt-4" data-aos="fade-up" data-aos-delay="300">
      <span class="text-white">Add to MetaMask 🦊</span>
    </button>
  </div>
</div>

<div class="flex flex-col items-center text-center mt-4 md:mt-10 mx-4 md:mx-8">
  <div class="flex flex-wrap justify-around w-full max-w-1240px bg-lightgrey rounded pb-12 px-10">
    <div class="min-w-150px flex flex-col items-center leading-5 mt-12"><img class="h-50px inline mb-4" src={images.farmerbenefitcampaign} alt="hourglass" /><span>Automated<br />farming</span></div>
    <div class="min-w-150px flex flex-col items-center leading-5 mt-12"><img class="h-50px inline mb-4" src={images.gem} alt="gem" /><span>Rewarded<br />commitment</span></div>
    <div class="min-w-150px flex flex-col items-center leading-5 mt-12"><img class="h-50px inline mb-4" src={images.pirateflag} alt="pirate flag" /><span>Treasury APR<br />distribution</span></div>
    <div class="min-w-150px flex flex-col items-center leading-5 mt-12"><img class="h-50px inline mb-4" src={images.womanlaptop} alt="woman laptop" /><span>The future of work<br />is DAO</span></div>
    <div class="min-w-150px flex flex-col items-center leading-5 mt-12"><img class="h-50px inline mb-4" src={images.raisedhand} alt="raised hand" /><span>Hybrid governance<br />beyond coin vote</span></div>
  </div>
</div>


<div class="flex flex-col items-center text-center mt-4 md:mt-10 mx-8">
  <div class="flex flex-col items-center w-full max-w-1100px pb-4 px-2 md:px-10">
    <img class="h-150px inline mb-4" src="https://emojipedia-us.s3.dualstack.us-west-1.amazonaws.com/thumbs/240/apple/285/money-mouth-face_1f911.png" alt="hourglass" />
    <div class="font-hero text-center mt-10">Earn rewards<br />in three easy steps</div>
    <div class="font-thin text-base text-justify md:text-center mt-20px">
      Stake your DOUGH, vote on proposals and claim your monthly rewards. 250+ members joined in the first month!
      <br />
      <b>Stake DOUGH to start earning rewards</b>
    </div>
    <button 
    onclick="location.href='/#/dough-staking';"
    class="btnbig mt-8 text-white rounded-8px p-15px">
  Stake DOUGH
  </button>
  </div>
</div>



<div class="flex flex-col items-center text-center mt-0 md:mt-0 mx-8">
  <div class="flex flex-wrap justify-around w-full max-w-1100px pb-12 px-2 md:px-10">
    <div class="font-huge text-center mt-10">A new governance model</div>
    <div class="font-thin text-base text-justify md:text-center mt-20px">
      If you stake DOUGH for a minimum of 6 months, you get in exchange veDOUGH, PieDAO’s new governance token.
      <br /><br />  
      With veDOUGH you can help the community steer the destiny of the DAO and its products, make proposals, vote on issues while <strong>being compensated</strong> for your commitment and effort.
      <br /><br />  
      In fact <strong>PieDAO redistributes 60% of the revenues</strong> generated by its products and treasury management to active community members, proportionally to the amount of veDOUGH they hold.
    </div> 
  </div>
</div>


<div class="w-full flex flex-col items-center text-center">
  <div class="flex w-full justify-center">
    <StakingStats />
  </div>
</div>


<div class="flex flex-col items-center text-center md:mt-2 mx-8">
  <div class="flex flex-wrap justify-around w-full max-w-1100px pb-12 px-10">
    <div class="w-full font-huge text-center mt-10">Token Economics</div>
    <div class="w-full font-thin text-base text-center mt-20px">
      This is how PieDAO (and You!) makes money
    </div> 
    <a href="#/staking-simulator" class="font-bold text-base text-center">
      Learn more about staking >
    </a>   
  </div>
  <div class="hidden md:block">
    <LottiePlayer
    src="https://assets10.lottiefiles.com/private_files/lf30_wksf88hl.json"
    autoplay="{true}"
    loop="{true}"
    controls="{false}"
    renderer="svg"
    background="white"
    height=""
    width="100%"
    controlsLayout="{controlsLayout}"
    />
  </div>
  <div class="block md:hidden">
    <img class="w-100% inline mb-4" src={images.doughconomics} alt="dough economics diagram" />
  </div>
</div>


<div class="flex flex-col items-center text-center mt-6 md:mt-20">
  <div class="w-full max-w-1200px">
    <div class="bg-melanzanafritta min-h-300px flex flex-col md:flex-row items-center text-white rounded p-8 md:p-12 mx-4">
      <img class="w-180px h-180px md:mr-12" src={images.newblack} alt="vedough is the new black"/>
      <div class="font-thin">
        <div class="font-bold text-l md:text-xl text-center md:text-left mt-4 md:mt-0">DOUGH Stabilizer</div>
        <div class="text-justify text-base mt-4 md:mt-0">Price volatility  calls into question the legitimacy of PieDAO. Confidence in DOUGH cannot be established if holders buy and sell it on a whim. Staking removes DOUGH from the open market by locking it for a predetermined amount of time. This allows the price of DOUGH to achieve stability in the face of short term disturbances.
          </div>
      </div>
    </div>
  </div>
  </div>


<div class="flex flex-col items-center text-center mt-4 md:mt-10 mx-8">
  <div class="flex flex-wrap justify-around w-full max-w-1100px pb-12 px-2 md:px-10">
    <div class="w-full font-huge text-center mt-10">Get paid for your work</div>
    <div class="w-full font-thin text-base text-justify md:text-center mt-20px">
      Your time and insight is valuable to our community. It is only fair that you get rewarded for your part in the growth of PieDAO. Staking along with being an active voter entitles you to monthly rewards. Amazing right? Welcome to the future of work, full time token holder.      <br /><br />
    </div> 
    <button 
    onclick="location.href='/#/dough-staking';"
    class="btnbig mt-4 text-white rounded-8px p-15px">
  Stake DOUGH
  </button>

  </div>
</div>








  