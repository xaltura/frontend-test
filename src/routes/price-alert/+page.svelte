<script lang='ts'>
import { numberOfUnreadNotification } from '$lib/store';
//import { enhance } from '$app/forms';
import toast from 'svelte-french-toast';
import { goto } from '$app/navigation';
import {screenWidth } from '$lib/store';
import MiniPlot from '$lib/components/MiniPlot.svelte';
import { onMount } from 'svelte';
import ArrowLogo from "lucide-svelte/icons/move-up-right";


let cloudFrontUrl = import.meta.env.VITE_IMAGE_URL;

export let data;

const rawData = data?.getMiniPlotsIndex;

function getCurrentDateFormatted() {
    // Get current date
    let date = new Date();
    
    // If today is Saturday or Sunday, move to the previous Friday
    if (date.getDay() === 6) { // Saturday
        date.setDate(date.getDate() - 1);
    } else if (date.getDay() === 0) { // Sunday
        date.setDate(date.getDate() - 2);
    }
    
    // Define months array for formatting
    const months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
    
    // Get formatted date components
    const month = months[date.getMonth()];
    const day = date.getDate();
    const year = date.getFullYear();
    
    // Return formatted date
    return `${month} ${day}, ${year}`;
}

let editMode = false;
let numberOfChecked = 0;
let deletePriceAlertList = [];

let priceDataSP500;
let priceDataNasdaq;
let priceDataDowJones;
let priceDataRussel2000;

let changeSP500, changeNasdaq, changeDowJones, changeRussel2000;
let previousCloseSP500, previousCloseNasdaq, previousCloseDowJones, previousCloseRussel2000;
// Assign values based on the symbol
rawData?.forEach(({ symbol, priceData, changesPercentage, previousClose }) => {
    switch (symbol) {
    case "SPY":
        priceDataSP500 = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
        priceDataSP500 = priceDataSP500?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
        changeSP500 = changesPercentage;
        previousCloseSP500 = previousClose;
        break;
    case "QQQ":
        priceDataNasdaq = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
        priceDataNasdaq = priceDataNasdaq?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
        changeNasdaq = changesPercentage;
        previousCloseNasdaq = previousClose;
        break;
    case "DIA":
        priceDataDowJones = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
        priceDataDowJones = priceDataDowJones?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
        changeDowJones = changesPercentage
        previousCloseDowJones = previousClose;
        break;
    case "IWM":
        priceDataRussel2000 = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
        priceDataRussel2000 = priceDataRussel2000?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
        changeRussel2000 = changesPercentage;
        previousCloseRussel2000 = previousClose;
        break;
    default:
        // Handle unknown symbol
        break;
    }
});

let isLoaded = false;
let priceAlertList = data?.getPriceAlert;
        



function stockSelector(symbol, assetType) {
    if (editMode) {
    }
    else {
        goto(`/${assetType === 'stock' ? 'stocks' : assetType === 'etf' ? 'etf' : 'crypto'}/${symbol}`)
    }
}


async function handleFilter(priceAlertId) {

  const filterSet = new Set(deletePriceAlertList);

  // Check if the new filter already exists in the list
  if (filterSet?.has(priceAlertId)) {
    // If it exists, remove it from the list
    filterSet?.delete(priceAlertId);
  } else {
    // If it doesn't exist, add it to the list
    filterSet?.add(priceAlertId);

  }
  deletePriceAlertList = Array?.from(filterSet);
  numberOfChecked = deletePriceAlertList?.length;
}

async function handleDelete() {

    if (numberOfChecked === 0) {
        toast.error(`You need to select symbols before you can delete them`, {
            style: 'border-radius: 10px; background: #333; color: #fff;  padding: 12px; margin-top: 10px; box-shadow: 0 25px 50px -12px rgb(0 0 0 / 0.25);',
        });
    }
    else {

        priceAlertList = priceAlertList?.filter(item => !deletePriceAlertList?.includes(item?.id));

        priceAlertList = [...priceAlertList];

        const postData = {
            'priceAlertIdList': deletePriceAlertList,
            'path': 'delete-price-alert'
        }

        const response = await fetch('/api/fastify-post-data', {
            method: 'POST',
            headers: {
             "Content-Type": "application/json"
            },
            body: JSON.stringify(postData)
        });
        deletePriceAlertList = [];
        numberOfChecked = 0;
        editMode = !editMode

    }

}


onMount( async () => {


isLoaded = true;



});




$: charNumber = $screenWidth < 640 ? 15 : 40;
    
    
</script>


<svelte:head>
    <title> {$numberOfUnreadNotification > 0 ? `(${$numberOfUnreadNotification})` : ''} Price Alert · stocknear</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />

    <meta name="description" content="Set a price alert and get instant notification.">
    <!-- Other meta tags -->
    <meta property="og:title" content="Price Alert · stocknear"/>
    <meta property="og:description" content="Set a price alert and get instant notification.">
    <meta property="og:type" content="website"/>
    <!-- Add more Open Graph meta tags as needed -->

    <!-- Twitter specific meta tags -->
    <meta name="twitter:card" content="summary_large_image"/>
    <meta name="twitter:title" content="Price Alert · stocknear"/>
    <meta name="twitter:description" content="Set a price alert and get instant notification.">
    <!-- Add more Twitter meta tags as needed -->
</svelte:head>
        
    
          
        
        
            
        
        
<section class="w-full max-w-3xl sm:max-w-screen-2xl overflow-hidden min-h-screen pt-5 pb-40 lg:px-3">
          
  <div class="text-sm sm:text-[1rem] breadcrumbs ml-4">
    <ul>
      <li><a href="/" class="text-gray-300">Home</a></li>
      <li class="text-gray-300">Price Alert</li>
    </ul>
  </div>
          
  <div class="w-full overflow-hidden m-auto mt-5">
    
    <div class="sm:p-0 flex justify-center w-full m-auto overflow-hidden ">
        <div class="relative flex justify-center items-start overflow-hidden w-full">


            <main class="w-full lg:w-3/4 lg:pr-5">


              <div class="w-full  m-auto sm:bg-[#27272A] sm:rounded-xl h-auto pl-10 pr-10 pt-5 sm:pb-10 sm:pt-10 mt-3 mb-8">
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-10">
      
          <!-- Start Column -->
          <div>
            <div class="flex flex-row justify-center items-center">
              <h1 class="text-4xl sm:text-5xl text-white font-bold mb-5">
                Price Alert
              </h1>
            </div>
    
            <span class="text-white text-md font-medium text-center flex justify-center  items-center ">
              Get email notifications instantly when your alert goes off, so you never miss out!
            </span>
          </div>
          <!-- End Column -->
      
          <!-- Start Column -->
          <div class="hidden sm:block relative m-auto mb-5 mt-5 sm:mb-0 sm:mt-0">
            <svg class="w-40 -my-5" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
              <defs>
                <filter id="glow">
                  <feGaussianBlur stdDeviation="5" result="glow"/>
                  <feMerge>
                    <feMergeNode in="glow"/>
                    <feMergeNode in="SourceGraphic"/>
                  </feMerge>
                </filter>
              </defs>
              <path fill="#1E40AF" d="M57.6,-58.7C72.7,-42.6,81.5,-21.3,82,0.5C82.5,22.3,74.7,44.6,59.7,60.1C44.6,75.6,22.3,84.3,0,84.3C-22.3,84.2,-44.6,75.5,-61.1,60.1C-77.6,44.6,-88.3,22.3,-87.6,0.7C-86.9,-20.8,-74.7,-41.6,-58.2,-57.7C-41.6,-73.8,-20.8,-85.2,0.2,-85.4C21.3,-85.6,42.6,-74.7,57.6,-58.7Z" transform="translate(100 100)" filter="url(#glow)" />
            </svg>
            
            
            <div class="z-1 absolute top-2">
              <img class="w-[120px] h-fit ml-10" src={cloudFrontUrl+"/assets/price_alert_logo.png"} alt="logo" loading='lazy'>
            </div>
          </div>
          <!-- End Column -->
      
        </div>
      </div>
        
    
        {#if isLoaded}
      <div class="sm:hidden">
        <div class="text-white text-xs sm:text-sm pb-5 sm:pb-2 pl-3 sm:pl-0">
          Stock Indexes - {getCurrentDateFormatted()}
        </div>
    
        <div class="w-full -mt-4 sm:mt-0 mb-8 m-auto flex justify-start sm:justify-center items-center p-3 sm:p-0">
          <div class="w-full grid grid-cols-2 md:grid-cols-4 gap-y-3 lg:gap-y-0 gap-x-3 ">
          <MiniPlot title="S&P500" priceData = {priceDataSP500} changesPercentage={changeSP500} previousClose={previousCloseSP500}/>
          <MiniPlot title="Nasdaq" priceData = {priceDataNasdaq} changesPercentage={changeNasdaq} previousClose={previousCloseNasdaq}/>
          <MiniPlot title="Dow" priceData = {priceDataDowJones} changesPercentage={changeDowJones} previousClose={previousCloseDowJones}/>
          <MiniPlot title="Russel" priceData = {priceDataRussel2000} changesPercentage={changeRussel2000} previousClose={previousCloseRussel2000}/>
          </div>
        </div>

        </div>
      
      
        
    
            {#if priceAlertList?.length === 0}
            <div class="flex flex-col justify-center items-center m-auto pt-8">
                <span class="text-white font-bold text-white text-xl sm:text-3xl">
                    No Alerts set
                </span>
    
                <span class="text-white text-sm sm:text-[1rem] m-auto p-4 text-center">
                    Create price alerts for your stocks that have the most potential in your opinion.
                </span>
                {#if !data?.user}
                <a class="w-64 flex mt-10 justify-center items-center m-auto btn text-white bg-purple-600 hover:bg-purple-500 transition duration-150 ease-in-out group" href="/register">
                    Get Started 
                    <span class="tracking-normal group-hover:translate-x-0.5 transition-transform duration-150 ease-in-out">
                        <svg class="w-4 h-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><g transform="rotate(90 12 12)"><g fill="none"><path d="M24 0v24H0V0h24ZM12.593 23.258l-.011.002l-.071.035l-.02.004l-.014-.004l-.071-.035c-.01-.004-.019-.001-.024.005l-.004.01l-.017.428l.005.02l.01.013l.104.074l.015.004l.012-.004l.104-.074l.012-.016l.004-.017l-.017-.427c-.002-.01-.009-.017-.017-.018Zm.265-.113l-.013.002l-.185.093l-.01.01l-.003.011l.018.43l.005.012l.008.007l.201.093c.012.004.023 0 .029-.008l.004-.014l-.034-.614c-.003-.012-.01-.02-.02-.022Zm-.715.002a.023.023 0 0 0-.027.006l-.006.014l-.034.614c0 .012.007.02.017.024l.015-.002l.201-.093l.01-.008l.004-.011l.017-.43l-.003-.012l-.01-.01l-.184-.092Z"/><path fill="white" d="M13.06 3.283a1.5 1.5 0 0 0-2.12 0L5.281 8.939a1.5 1.5 0 0 0 2.122 2.122L10.5 7.965V19.5a1.5 1.5 0 0 0 3 0V7.965l3.096 3.096a1.5 1.5 0 1 0 2.122-2.122L13.06 3.283Z"/></g></g></svg>
                    </span>
                </a>
                {/if}
    
            </div>
            {:else}

            <div class="flex flex-row justify-end items-center pr-4 sm:pr-0 pb-2">
                {#if editMode}
                <label on:click={handleDelete} class="border text-sm border-gray-600 ml-3 cursor-pointer inline-flex items-center justify-center space-x-1 whitespace-nowrap rounded-md py-2 pl-3 pr-4 font-semibold text-white shadow-sm bg-[#09090B] sm:hover:bg-[#09090B]/60 ease-out">
                    <svg class="inline-block w-5 h-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="white" d="M10 5h4a2 2 0 1 0-4 0M8.5 5a3.5 3.5 0 1 1 7 0h5.75a.75.75 0 0 1 0 1.5h-1.32l-1.17 12.111A3.75 3.75 0 0 1 15.026 22H8.974a3.75 3.75 0 0 1-3.733-3.389L4.07 6.5H2.75a.75.75 0 0 1 0-1.5zm2 4.75a.75.75 0 0 0-1.5 0v7.5a.75.75 0 0 0 1.5 0zM14.25 9a.75.75 0 0 1 .75.75v7.5a.75.75 0 0 1-1.5 0v-7.5a.75.75 0 0 1 .75-.75m-7.516 9.467a2.25 2.25 0 0 0 2.24 2.033h6.052a2.25 2.25 0 0 0 2.24-2.033L18.424 6.5H5.576z"/></svg>
                    <span class="ml-1 text-white text-sm">
                        {numberOfChecked}
                    </span>
                </label>
                {/if}
                <label on:click={() => editMode = !editMode} class="border text-sm border-gray-600 ml-3 cursor-pointer inline-flex items-center justify-center space-x-1 whitespace-nowrap rounded-md py-2 pl-3 pr-4 font-semibold text-white shadow-sm bg-[#09090B] sm:hover:bg-[#09090B]/60 ease-out">
                    <svg class="inline-block w-5 h-5" xmlns="http://www.w3.org/2000/svg"  viewBox="0 0 1024 1024"><path fill="white" d="M832 512a32 32 0 1 1 64 0v352a32 32 0 0 1-32 32H160a32 32 0 0 1-32-32V160a32 32 0 0 1 32-32h352a32 32 0 0 1 0 64H192v640h640z"/><path fill="white" d="m469.952 554.24l52.8-7.552L847.104 222.4a32 32 0 1 0-45.248-45.248L477.44 501.44l-7.552 52.8zm422.4-422.4a96 96 0 0 1 0 135.808l-331.84 331.84a32 32 0 0 1-18.112 9.088L436.8 623.68a32 32 0 0 1-36.224-36.224l15.104-105.6a32 32 0 0 1 9.024-18.112l331.904-331.84a96 96 0 0 1 135.744 0z"/></svg>
                    {#if !editMode}
                    <span class="ml-1 text-white text-sm">
                        Edit
                    </span>
                    {:else}
                    <span class="ml-1 text-white text-sm">
                      Cancel
                    </span>
                    {/if}
                </label>
            </div>
            <!--Start Table-->
            <div class="w-screen sm:w-full rounded-lg overflow-hidden overflow-x-scroll no-scrollbar">
                <table class="table table-sm table-compact rounded-none sm:rounded-md w-full bg-[#09090B] border-bg-[#09090B] m-auto mt-4 ">
                  <!-- head -->
                  <thead>
                    <tr class="">
                      <th class="text-white font-semibold text-[1rem] ">Symbol</th>
                      <th class="text-white font-semibold text-[1rem] ">Company</th>
                      <th class="text-white font-semibold text-end text-[1rem] ">Volume</th>
                      <th class="text-white font-semibold text-end text-[1rem] ">Price when Created</th>
                      <th class="text-white font-semibold text-end text-[1rem] ">Price Target</th>
                      <th class="text-white font-semibold text-end text-[1rem] ">Current Price</th>
                      <th class="text-white font-semibold text-end text-[1rem] ">Change</th>
                    </tr>
                  </thead>
                  <tbody class="p-3">
                    {#each priceAlertList as item, index}
                    <!-- row -->
                    <tr on:click={() => stockSelector(item?.symbol, item?.assetType)} class="sm:hover:bg-[#245073] sm:hover:bg-opacity-[0.2] odd:bg-[#27272A] border-b-[#09090B] cursor-pointer">
                      
                      <td on:click={() => handleFilter(item?.id)} class="text-blue-400 font-medium text-sm sm:text-[1rem] whitespace-nowrap text-start border-b-[#09090B] flex flex-row items-center">
                        <input type="checkbox" checked={deletePriceAlertList?.includes(item?.id) ?? false} class="{!editMode ? 'hidden' : ''} bg-[#2E3238] h-[18px] w-[18px] rounded-sm ring-offset-0 mr-3" />
                        {item?.symbol}
                      </td>
        
                      <td on:click={() => handleFilter(item?.id)} class="text-white text-sm sm:text-[1rem] whitespace-nowrap border-b-[#09090B]">
                        {item?.name?.length > charNumber ? item?.name?.slice(0,charNumber) + "..." : item?.name}
                      </td>
                    
                    <td class="text-white font-medium text-sm sm:text-[1rem] whitespace-nowrap text-end border-b-[#09090B]">
                        {new Intl.NumberFormat("en", {
                            minimumFractionDigits: 2,
                            maximumFractionDigits: 2
                        }).format(item?.volume)}
                        
                    </td>
                    <td class="text-white font-medium text-sm sm:text-[1rem] whitespace-nowrap text-end border-b-[#09090B]">
                        ${item?.priceWhenCreated}
                    </td>

        
                    <td class="text-white font-medium text-sm sm:text-[1rem] whitespace-nowrap text-end border-b-[#09090B]">
                        ${item?.targetPrice}
                    </td>

                    <td class="text-white font-medium text-sm sm:text-[1rem] whitespace-nowrap text-end border-b-[#09090B]">
                      ${item.price?.toFixed(2)}
                    </td>

                    <td class="text-white font-medium text-sm sm:text-[1rem] whitespace-nowrap text-end border-b-[#09090B]">
                      {#if item?.changesPercentage >=0}
                        <span class="text-[#37C97D]">+{item?.changesPercentage?.toFixed(2)}%</span>
                      {:else}
                        <span class="text-[#FF2F1F]">{item?.changesPercentage?.toFixed(2)}% </span> 
                      {/if}
                    </td>
        

                    </tr>
                    
                 
        
        
                    {/each}
                  </tbody>
                </table>
        
        
                
        
        
              </div>
            <!--End Table-->


            {/if}
        {:else}
        <div class="flex justify-center items-center h-80">
          <div class="relative">
          <label class="bg-[#09090B] rounded-xl h-14 w-14 flex justify-center items-center absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">
              <span class="loading loading-spinner loading-md text-gray-400"></span>
          </label>
          </div>
      </div>
    
        {/if}
        
       </main>


  <aside class="hidden lg:block relative fixed w-1/4 ml-4">        
  
    {#if data?.user?.tier !== 'Pro' || data?.user?.freeTrial}
    <div on:click={() => goto('/pricing')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
        <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
            <div class="w-full flex justify-between items-center p-3 mt-3">
            <h2 class="text-start text-xl font-semibold text-white ml-3">
            Pro Subscription 🔥
            </h2>
            <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
            </div>
            <span class="text-white p-3 ml-3 mr-3">
                Upgrade now for unlimited access to all data and tools.
            </span>
        </div>
    </div>
    {/if}
  
    <div on:click={() => goto('/watchlist/stocks')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
        <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
            <div class="w-full flex justify-between items-center p-3 mt-3">
            <h2 class="text-start text-xl font-semibold text-white ml-3">
            Watchlist ⭐
            </h2>
            <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
            </div>
            <span class="text-white p-3 ml-3 mr-3">
                Build your watchlist to keep track of their performance.
            </span>
        </div>
    </div>
  
     <div on:click={() => goto('/stock-screener')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
        <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
            <div class="w-full flex justify-between items-center p-3 mt-3">
            <h2 class="text-start text-xl font-semibold text-white ml-3">
            Stock Screener 🔎
            </h2>
            <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
            </div>
            <span class="text-white p-3 ml-3 mr-3">
                Build your Stock Screener to find profitable stocks.
            </span>
        </div>
    </div>
  
  </aside>
  
  </div>
  </div>
  
  
  </div>
  
  
  
  </section>
  