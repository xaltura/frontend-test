<script lang='ts'>
import { goto } from '$app/navigation';
import { screenWidth, numberOfUnreadNotification } from '$lib/store';
import logo from '$lib/images/top_winner_logo.png';
import { abbreviateNumber } from '$lib/utils';
import MiniPlot from '$lib/components/MiniPlot.svelte';
import { onMount } from 'svelte';
import ArrowLogo from "lucide-svelte/icons/move-up-right";

export let data;
let isLoaded = false;
const rawData = data?.getMiniPlotsIndex;
let timePeriod = '1D'
let priceDataSP500;
let priceDataNasdaq;
let priceDataDowJones;
let priceDataRussel2000;

let changeSP500, changeNasdaq, changeDowJones, changeRussel2000;
let previousCloseSP500, previousCloseNasdaq, previousCloseDowJones, previousCloseRussel2000;



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


let outputList = data?.getDailyGainerLoserActive;
let gainerLoserActive = outputList?.gainers[timePeriod]

let buttonText = 'Top Winners';


const tabs = [
      {
        title: "Gainers",
      },
      {
        title: "Losers",
      },
      {
        title: "Active",
      },
    ];
  
let activeIdx = 0;

function changeSection(index) {
    activeIdx = index;

    if (index === 0)
    {
      gainerLoserActive = outputList?.gainers[timePeriod];
      buttonText = 'Top Winners'
    }
    else if (index === 1)
    {
      gainerLoserActive = outputList?.losers[timePeriod];
      buttonText = 'Top Losers'
    }
    else if (index === 2)
    {
      gainerLoserActive = outputList?.active[timePeriod];
      buttonText = 'Most Active'
    }
} 



function selectTimeInterval(event) {

   
    timePeriod = event.target.value === 'oneDay' ? '1D' : event.target.value === 'oneWeek' ? '1W' : event.target.value === 'oneMonth' ? '1M' : event.target.value === 'threeMonths' ? '3M' : '6M';


    if (buttonText === 'Top Winners')
    {
      gainerLoserActive = outputList?.gainers[timePeriod];
    }
    else if (buttonText === 'Top Losers')
    {
      gainerLoserActive = outputList?.losers[timePeriod];
    }
    else if (buttonText === 'Most Active')
    {
      gainerLoserActive = outputList?.active[timePeriod];
    }
  }

onMount( () => {
  isLoaded = true;
})


let sortOrders = {
  symbol: 'none',
  name: 'none',
  change: 'none',
  price: 'none',
  marketCap: 'none',
  volume: 'none',
};

// Generalized sorting function
function sortData(key) {
  // Reset all other keys to 'none' except the current key
  for (const k in sortOrders) {
    if (k !== key) {
      sortOrders[k] = 'none';
    }
  }

  // Cycle through 'none', 'asc', 'desc' for the clicked key
  const orderCycle = ['none', 'asc', 'desc'];
  let originalData = [];

  if (buttonText === 'Top Winners')
    {
      originalData = data?.getDailyGainerLoserActive?.gainers[timePeriod]
    }
    else if (buttonText === 'Top Losers')
    {
      originalData = data?.getDailyGainerLoserActive?.losers[timePeriod]
    }
    else if (buttonText === 'Most Active')
    {
      originalData = data?.getDailyGainerLoserActive?.active[timePeriod]
    }

  const currentOrderIndex = orderCycle.indexOf(sortOrders[key]);
  sortOrders[key] = orderCycle[(currentOrderIndex + 1) % orderCycle.length];

  const sortOrder = sortOrders[key];

  // Reset to original data when 'none' and stop further sorting
  if (sortOrder === 'none') {
    gainerLoserActive = [...originalData]; // Reset to original data (spread to avoid mutation)
    return;
  }

  // Define comparison functions for each key
  const compareFunctions = {
    symbol: (a, b) => {
      const symbolA = a.symbol.toUpperCase();
      const symbolB = b.symbol.toUpperCase();
      return sortOrder === 'asc' ? symbolA.localeCompare(symbolB) : symbolB.localeCompare(symbolA);
    },
    name: (a, b) => {
      const nameA = a.name.toUpperCase();
      const nameB = b.name.toUpperCase();
      return sortOrder === 'asc' ? nameA.localeCompare(nameB) : nameB.localeCompare(nameA);
    },
    change: (a, b) => {
      const numA = parseFloat(a?.changesPercentage);
      const numB = parseFloat(b?.changesPercentage);
      return sortOrder === 'asc' ? numA - numB : numB - numA;
    },
    price: (a, b) => {
      const numA = parseFloat(a?.price);
      const numB = parseFloat(b?.price);
      return sortOrder === 'asc' ? numA - numB : numB - numA;
    },
    marketCap: (a, b) => {
      const numA = parseFloat(a.marketCap);
      const numB = parseFloat(b.marketCap);
      return sortOrder === 'asc' ? numA - numB : numB - numA;
    },
    volume: (a, b) => {
      const numA = parseFloat(a.volume);
      const numB = parseFloat(b.volume);
      return sortOrder === 'asc' ? numA - numB : numB - numA;
    },
  };

  // Sort using the appropriate comparison function
  gainerLoserActive = [...originalData].sort(compareFunctions[key]);
}


$: charNumber = $screenWidth < 640 ? 20 : 30;

</script>



<svelte:head>

<meta charset="utf-8" />
<meta name="viewport" content="width=device-width" />
<title>
  {$numberOfUnreadNotification > 0 ? `(${$numberOfUnreadNotification})` : ''} Today's Top Stock Gainers, Losers and Most Active · stocknear
</title>
<meta name="description" content={`A list of the stocks with the highest percentage gain, highest percentage loss and most active today. See stock price, volume, market cap and more.`} />

<!-- Other meta tags -->
<meta property="og:title" content={`Today's Top Stock Gainers, Losers and Most Active · stocknear`}/>
<meta property="og:description" content={`A list of the stocks with the highest percentage gain, highest percentage loss and most active today. See stock price, volume, market cap and more.`} />
<meta property="og:type" content="website"/>
<!-- Add more Open Graph meta tags as needed -->

<!-- Twitter specific meta tags -->
<meta name="twitter:card" content="summary_large_image"/>
<meta name="twitter:title" content={`Today's Top Stock Gainers, Losers and Most Active · stocknear`}/>
<meta name="twitter:description" content={`A list of the stocks with the highest percentage gain, highest percentage loss and most active today. See stock price, volume, market cap and more.`} />
<!-- Add more Twitter meta tags as needed -->

</svelte:head>
  
<section class="w-full max-w-3xl sm:max-w-screen-2xl overflow-hidden min-h-screen pt-5 pb-40 lg:px-3">
          
  <div class="text-sm sm:text-[1rem] breadcrumbs ml-4">
    <ul>
      <li><a href="/" class="text-gray-300">Home</a></li>
      <li class="text-gray-300">Market Movers</li>
    </ul>
  </div>
          
  <div class="w-full overflow-hidden m-auto mt-5">
    
    <div class="sm:p-0 flex justify-center w-full m-auto overflow-hidden ">
        <div class="relative flex justify-center items-start overflow-hidden w-full">


            <main class="w-full lg:w-3/4 lg:pr-5">


              <div class="w-full  m-auto sm:bg-[#27272A] h-auto pl-10 pr-10 pt-5 sm:pb-10 sm:pt-10 mt-3 mb-8">
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-10">
  
      <!-- Start Column -->
      <div>
        <div class="flex flex-row justify-center items-center">
          <h1 class="text-3xl sm:text-4xl text-white text-center font-bold mb-5">
            Market Movers
          </h1>
        </div>

        <span class="hidden sm:block text-white text-md font-semibold text-center flex justify-center items-center ">
          Explore top-performing, underperforming & most active traded stocks across different time frames.
        </span>


      </div>
      <!-- End Column -->
  
      <!-- Start Column -->
      <div class="hidden sm:block relative m-auto mb-5 mt-5 md:mb-0 md:mt-0">
        <svg class="w-32 -my-4" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
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
        
        
        <div class="z-1 absolute -top-3 right-1">
          <img class="w-28" src={logo} alt="logo" loading="lazy">
        </div>
      </div>
      <!-- End Column -->
    </div>

  </div>

  {#if isLoaded}

  <div class="sm:hidden text-white text-xs sm:text-sm pb-5 sm:pb-2 pl-3 sm:pl-0">
    Stock Indexes - {getCurrentDateFormatted()}
  </div>

  <div class="w-full sm:hidden -mt-4 sm:mt-0 mb-8 m-auto flex justify-start sm:justify-center items-center p-3 sm:p-0">
    <div class="w-full grid grid-cols-2 md:grid-cols-3 gap-y-3 gap-x-3 ">
    <MiniPlot title="S&P500" priceData = {priceDataSP500} changesPercentage={changeSP500} previousClose={previousCloseSP500}/>
    <MiniPlot title="Nasdaq" priceData = {priceDataNasdaq} changesPercentage={changeNasdaq} previousClose={previousCloseNasdaq}/>
    <MiniPlot title="Dow" priceData = {priceDataDowJones} changesPercentage={changeDowJones} previousClose={previousCloseDowJones}/>
    <MiniPlot title="Russel" priceData = {priceDataRussel2000} changesPercentage={changeRussel2000} previousClose={previousCloseRussel2000}/>
    </div>
  </div>


    <div class="w-full m-auto mb-10 sm:mb-5 bg-[#09090B] pl-3 pr-3 sm:pl-0 sm:pr-0">
     
      

        <div class="bg-[#313131] w-fit relative m-auto sm:m-0 sm:mr-auto flex sm:flex-wrap items-center justify-center rounded-lg p-1 -mt-3">
          {#each tabs as item, i}
          <label
              on:click={() => (changeSection(i))}
              class="cursor-pointer group relative z-[1] rounded-full px-6 py-1 {activeIdx === i
              ? 'z-0'
              : ''} "
          >
              {#if activeIdx === i}
                  <div
                  class="absolute inset-0 rounded-lg bg-purple-600"
                  ></div>
              {/if}
              <span class="relative text-[1rem] sm:text-lg block font-semibold duration-200 text-white">
                  {item.title}
              </span>
            </label>
          {/each}
          </div>


      </div>

    
    <!--Start Top Winners/Losers-->
    <div class="flex flex-col justify-center items-center overflow-hidden">
      

        
      <div class="w-full flex justify-start sm:justify-end items-center mb-10 ml-5 sm:ml-0">
        <div class="relative flex flex-col items-start">
          <span class="sm:hidden text-white text-sm mb-3">
            Time Period:
          </span>
        <select class="w-32 text-white select select-bordered select-sm p-0 pl-5 overflow-y-auto bg-[#2A303C]" on:change={selectTimeInterval}>
            <option disabled>Choose a time period</option>
            <option value="oneDay" selected>1 Day</option>
            <option value="oneWeek">1 Week</option>
            <option value="oneMonth">1 Month</option>
            <option value="threeMonths">3 Months</option>
            <option value="sixMonths">6 Months</option>                                    
        </select>
        </div>
    </div>
    


    <div class="w-full overflow-x-scroll no-scrollbar">
      <table class="table table-sm table-compact rounded-none sm:rounded-md w-full bg-[#09090B] border-bg-[#09090B]">
        <thead>
          <tr class="border-b border-[#27272A]">
            <th on:click={() => sortData('symbol')} class="cursor-pointer select-none text-white font-semibold text-[1rem] whitespace-nowrap">
              Symbol
              <svg class="flex-shrink-0 w-4 h-4 inline-block {sortOrders['symbol'] === 'asc' ? 'rotate-180' : sortOrders['symbol'] === 'desc' ? '' : 'hidden'} " viewBox="0 0 20 20" fill="currentColor" style="max-width:50px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
            <th on:click={() => sortData('name')} class="cursor-pointer select-none text-white font-semibold text-[1rem] whitespace-nowrap">
              Name
              <svg class="flex-shrink-0 w-4 h-4 inline-block {sortOrders['name'] === 'asc' ? 'rotate-180' : sortOrders['name'] === 'desc' ? '' : 'hidden'} " viewBox="0 0 20 20" fill="currentColor" style="max-width:50px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
            <th on:click={() => sortData('change')} class="cursor-pointer select-none text-end text-white font-semibold text-[1rem] whitespace-nowrap">
              % Change
              <svg class="flex-shrink-0 w-4 h-4 inline-block {sortOrders['change'] === 'asc' ? 'rotate-180' : sortOrders['change'] === 'desc' ? '' : 'hidden'} " viewBox="0 0 20 20" fill="currentColor" style="max-width:50px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
            <th on:click={() => sortData('price')} class="cursor-pointer select-none text-end text-white font-semibold text-[1rem] whitespace-nowrap">
              Price
              <svg class="flex-shrink-0 w-4 h-4 inline-block {sortOrders['price'] === 'asc' ? 'rotate-180' : sortOrders['price'] === 'desc' ? '' : 'hidden'} " viewBox="0 0 20 20" fill="currentColor" style="max-width:50px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
           <th on:click={() => sortData('marketCap')} class="cursor-pointer select-none text-end text-white font-semibold text-[1rem] whitespace-nowrap">
              Market Cap
              <svg class="flex-shrink-0 w-4 h-4 inline-block {sortOrders['marketCap'] === 'asc' ? 'rotate-180' : sortOrders['marketCap'] === 'desc' ? '' : 'hidden'} " viewBox="0 0 20 20" fill="currentColor" style="max-width:50px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
            <th on:click={() => sortData('volume')} class="cursor-pointer select-none text-end text-white font-semibold text-[1rem] whitespace-nowrap">
              Volume
              <svg class="flex-shrink-0 w-4 h-4 inline-block {sortOrders['volume'] === 'asc' ? 'rotate-180' : sortOrders['volume'] === 'desc' ? '' : 'hidden'} " viewBox="0 0 20 20" fill="currentColor" style="max-width:50px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
          </tr>
        </thead>
        <tbody>
          {#each gainerLoserActive as item}
          <tr class="border-b border-[#27272A] sm:hover:bg-[#245073] sm:hover:bg-opacity-[0.2] odd:bg-[#27272A]">
    
          <td class="border-b-[#09090B] text-sm sm:text-[1rem] whitespace-nowrap">
             <a href={"/stocks/"+item?.symbol} class="sm:hover:text-white text-blue-400">
              {item?.symbol}
            </a>
          </td>
          <td class="border-b-[#09090B] text-sm sm:text-[1rem] whitespace-nowrap text-white">
            {item?.name?.length > charNumber ? item?.name?.slice(0,charNumber) + "..." : item?.name}
          </td>

          <td class="text-white text-end text-sm sm:text-[1rem] font-medium border-b-[#09090B]">
              {#if item?.changesPercentage >=0}
                <span class="text-[#37C97D]">+{item?.changesPercentage >= 1000 ? abbreviateNumber(item?.changesPercentage) : item?.changesPercentage?.toFixed(2)}%</span>
              {:else}
                <span class="text-[#FF2F1F]">{item?.changesPercentage <= -1000 ? abbreviateNumber(item?.changesPercentage) : item?.changesPercentage?.toFixed(2)}% </span> 
              {/if}
          </td>

          <td class="text-white text-sm sm:text-[1rem] text-end border-b-[#09090B]">
            {item?.price?.toFixed(2)}
          </td>

          <td class="text-white text-sm sm:text-[1rem] border-b-[#09090B] text-end">
            {item?.marketCap !== null ? abbreviateNumber(item?.marketCap) : '-'}
          </td>

          <td class="text-white text-sm sm:text-[1rem] border-b-[#09090B] text-end">
            {item?.volume !== null ? abbreviateNumber(item?.volume) : '-'}
          </td>





          </tr>
          {/each}
        </tbody>
      </table>

    </div>



  </div>

  {:else}
  <div class="flex justify-center items-center h-80">
    <div class="relative">
    <label class="bg-[#09090B] rounded-lg h-14 w-14 flex justify-center items-center absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">
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
          Pro Subscription
          </h2>
          <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
          </div>
          <span class="text-white p-3 ml-3 mr-3">
              Upgrade now for unlimited access to all data and tools.
          </span>
      </div>
  </div>
  {/if}

  <div on:click={() => goto('/analysts')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
      <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
          <div class="w-full flex justify-between items-center p-3 mt-3">
          <h2 class="text-start text-xl font-semibold text-white ml-3">
          Top Analyst 📊
          </h2>
          <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
          </div>
          <span class="text-white p-3 ml-3 mr-3">
              Get the latest top Wall Street analyst ratings.
          </span>
      </div>
  </div>

  <div on:click={() => goto('/politicians')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
      <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
          <div class="w-full flex justify-between items-center p-3 mt-3">
          <h2 class="text-start text-xl font-semibold text-white ml-3">
          Congress Trading 🇺🇸
          </h2>
          <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
          </div>
          <span class="text-white p-3 ml-3 mr-3">
              Get the latest top Congress trading insights.
          </span>
      </div>
  </div>

</aside>

</div>
</div>


</div>



</section>

