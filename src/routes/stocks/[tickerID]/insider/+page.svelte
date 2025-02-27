<script lang="ts">
import { displayCompanyName, numberOfUnreadNotification, stockTicker } from '$lib/store';
import { formatString, abbreviateNumber } from '$lib/utils';
import UpgradeToPro from "$lib/components/UpgradeToPro.svelte";
import { onMount } from 'svelte';



  export let data;
  let isLoaded = false;
 

  let statistics = {};


  function calculateInsiderTradingStatistics(data) {
  const now = new Date();
  const year = now.getFullYear();
  const quarter = Math.floor(now.getMonth() / 3) + 1;

  // Helper function to check if the transaction date is within the current quarter
  const isInCurrentQuarter = (transactionDate) => {
    const date = new Date(transactionDate);
    return date.getFullYear() === year && Math.floor(date.getMonth() / 3) + 1 === quarter;
  };

  // Initialize statistics object
  let statistics = {
    soldShares: 0,
    buyShares: 0,
    buySharesPercentage: 0,
    soldSharesPercentage: 0,
    quarter: quarter,
    year: year,
  };

  // Summing up bought and sold shares efficiently in a single loop
  data.forEach(({ transactionType, securitiesTransacted, price, transactionDate }) => {
    if (price > 0 && isInCurrentQuarter(transactionDate)) {
      if (transactionType === "Sold") {
        statistics.soldShares += securitiesTransacted;
      } else if (transactionType === "Bought") {
        statistics.buyShares += securitiesTransacted;
      }
    }
  });

  const totalShares = statistics.buyShares + statistics.soldShares;

  if (totalShares > 0) {
    statistics.buySharesPercentage = Math.floor((statistics.buyShares / totalShares) * 100);
    statistics.soldSharesPercentage = 100 - statistics.buySharesPercentage;
  }

  return statistics;
}

  let rawData = data?.getInsiderTrading?.sort(
    (a, b) => new Date(b?.transactionDate) - new Date(a?.transactionDate)
  );

  let insiderTradingList = rawData?.slice(0,50)


  function backToTop() {
    window.scrollTo({
        top: 0,
    });
}

function extractOfficeInfo(inputString) {
  const indexOfficer = inputString?.toLowerCase()?.indexOf("officer:");
    const indexOther = inputString?.toLowerCase()?.indexOf("other:");
    if (indexOfficer !== -1) {
        return inputString?.substring(indexOfficer + "officer:"?.length)?.trim();
    } 
    else if (indexOther !== -1) {
        return inputString?.substring(indexOther + "other:"?.length)?.trim();
    } else if (inputString?.toLowerCase()?.includes('director')) {
        return 'Director';
    } 
    else if (inputString?.toLowerCase()?.includes('percent owner')) {
      return inputString?.replace('percent owner', '% Owner');
    } 
    else {
        return "n/a";
    }
}

async function handleScroll() {
    const scrollThreshold = document.body.offsetHeight * 0.8; // 80% of the website height
    const isBottom = window.innerHeight + window.scrollY >= scrollThreshold;
    if (isBottom && insiderTradingList?.length !== rawData?.length) {
        const nextIndex = insiderTradingList?.length;
        const filteredNewResults = rawData?.slice(nextIndex, nextIndex + 50);
        insiderTradingList = [...insiderTradingList, ...filteredNewResults];
    }
  }


onMount(() => {

  statistics = calculateInsiderTradingStatistics(rawData);

  isLoaded = true;
   window.addEventListener('scroll', handleScroll);
      return () => {
          window.removeEventListener('scroll', handleScroll);
      };
})

const transactionStyles = {
    'Bought': { text: 'Bought', class: 'text-[#37C97D]', border: 'border-[#37C97D]' },
    'Grant': { text: 'Grant', class: 'text-[#F8901E]', border: 'border-[#F8901E]'},
    'Sold': { text: 'Sold', class: 'text-[#FF2F1F]', border: 'border-[#FF2F1F]'},
    'Exercise': { text: 'Exercise', class: 'text-[#F8901E]', border: 'border-[#v]'},
    'n/a': { text: 'n/a', class: 'text-gray-300'}
  };

</script>



<svelte:head>

  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width" />
  <title>
    {$numberOfUnreadNotification > 0 ? `(${$numberOfUnreadNotification})` : ''} {$displayCompanyName} ({$stockTicker}) US Congress & Senate Trading · stocknear
  </title>
  <meta name="description" content={`Get the latest US congress & senate trading of ${$displayCompanyName} (${$stockTicker}) from democrates and republicans.`} />
  
  <!-- Other meta tags -->
  <meta property="og:title" content={`${$displayCompanyName} (${$stockTicker}) US Congress & Senate Trading · stocknear`}/>
  <meta property="og:description" content={`Get the latest US congress & senate trading of ${$displayCompanyName} (${$stockTicker}) from democrates and republicans.`} />
  <meta property="og:type" content="website"/>
  <!-- Add more Open Graph meta tags as needed -->

  <!-- Twitter specific meta tags -->
  <meta name="twitter:card" content="summary_large_image"/>
  <meta name="twitter:title" content={`${$displayCompanyName} (${$stockTicker}) US Congress & Senate Trading · stocknear`}/>
  <meta name="twitter:description" content={`Get the latest US congress & senate trading of ${$displayCompanyName} (${$stockTicker}) from democrates and republicans.`} />
  <!-- Add more Twitter meta tags as needed -->

</svelte:head>
  
     


<section class="bg-[#09090B] overflow-hidden text-white h-full mb-40 sm:mb-0 w-full">
    <div class="flex justify-center m-auto h-full overflow-hidden w-full">
        <div class="relative flex justify-center items-center overflow-hidden w-full">
              <div class="xl:p-7 w-full m-auto mt-2">
                    <div class="mb-6">
                      <h1 class="text-2xl sm:text-3xl text-gray-200 font-bold mb-4">
                          Insider Trading
                      </h1>
  

                        <div class="w-fit text-white p-3 sm:p-5 mb-5 rounded-lg sm:flex sm:flex-row sm:items-center border border-slate-800 text-sm sm:text-[1rem]">
                          <svg class="w-6 h-6 flex-shrink-0 inline-block sm:mr-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256"><path fill="#a474f6" d="M128 24a104 104 0 1 0 104 104A104.11 104.11 0 0 0 128 24m-4 48a12 12 0 1 1-12 12a12 12 0 0 1 12-12m12 112a16 16 0 0 1-16-16v-40a8 8 0 0 1 0-16a16 16 0 0 1 16 16v40a8 8 0 0 1 0 16"/></svg>
                        
                          {#if insiderTradingList?.length !== 0}
                          Get detailed insights of Insiders who bought or sold {$displayCompanyName} and the amounts involved!
                            {:else}
                            No trading history available for {$displayCompanyName}. Likely no insider trading has happened yet.
                          {/if}
                        </div>

                    </div>
      
                    {#if isLoaded}

                      {#if insiderTradingList?.length !== 0}

               

                    
                    {#if Object?.keys(statistics)?.length !== 0 }
                    <h3 class="text-white text-lg font-semibold pt-5">
                      Q{statistics?.quarter} {statistics?.year} Insider Statistics
                    </h3>
                     <!--Start Widget-->
                     <div class="w-full mt-5 mb-10 m-auto flex justify-center items-center p-1">
                      <div class="w-full grid grid-cols-2 lg:grid-cols-3 gap-y-3 lg:gap-y-3 gap-x-3 ">
      
                        <!--Start Put/Call-->  
                        <div class="flex flex-row items-center flex-wrap w-full px-3 sm:px-4 bg-[#262626] shadow-lg rounded-md h-20">
                          <div class="flex flex-col items-start">
                              <span class="font-medium text-gray-200 text-sm sm:text-[1rem]">Buy/Sell</span>
                              <span class="text-start text-sm sm:text-[1rem] font-medium text-white">
                                {(statistics?.buyShares/statistics?.soldShares)?.toFixed(2) }
                              </span>
                          </div>
                          <!-- Circular Progress -->
                            <div class="relative size-12 sm:size-14 ml-auto">
                              <svg class="size-full w-12 h-12 sm:w-14 sm:h-14" viewBox="0 0 36 36" xmlns="http://www.w3.org/2000/svg">
                                <!-- Background Circle -->
                                <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-[#3E3E3E]" stroke-width="3"></circle>
                                <!-- Progress Circle inside a group with rotation -->
                                <g class="origin-center -rotate-90 transform">
                                  <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-blue-500" stroke-width="3" stroke-dasharray="100" stroke-dashoffset={statistics?.buyShares/statistics?.soldShares>= 1 ? 0 : 100-(statistics?.buyShares/statistics?.soldShares*100)?.toFixed(2)}></circle>
                                </g>
                              </svg>
                              <!-- Percentage Text -->
                              <div class="absolute top-1/2 start-1/2 transform -translate-y-1/2 -translate-x-1/2">
                                <span class="text-center text-white text-sm sm:text-[1rem]">{(statistics?.buyShares/statistics?.soldShares)?.toFixed(2)}</span>
                              </div>
                            </div>
                          <!-- End Circular Progress -->
                
                      </div>
                      <!--End Put/Call-->
                      <!--Start Call Flow-->  
                      <div class="flex flex-row items-center flex-wrap w-full px-3 sm:px-4 bg-[#262626] shadow-lg rounded-md h-20">
                        <div class="flex flex-col items-start">
                            <span class="font-medium text-gray-200 text-sm sm:text-[1rem]">Bought Shares</span>
                            <span class="text-start text-sm sm:text-[1rem] font-medium text-white">
                              {new Intl.NumberFormat("en", {
                                minimumFractionDigits: 0,
                                maximumFractionDigits: 0
                            }).format(statistics?.buyShares)}
                            </span>
                        </div>
                        <!-- Circular Progress -->
                        <div class="relative size-12 sm:size-14 ml-auto">
                          <svg class="size-full w-12 h-12 sm:w-14 sm:h-14" viewBox="0 0 36 36" xmlns="http://www.w3.org/2000/svg">
                            <!-- Background Circle -->
                            <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-[#3E3E3E]" stroke-width="3"></circle>
                            <!-- Progress Circle inside a group with rotation -->
                            <g class="origin-center -rotate-90 transform">
                              <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-[#00FC50]" stroke-width="3" stroke-dasharray="100" stroke-dashoffset={100-statistics?.buySharesPercentage}></circle>
                            </g>
                          </svg>
                          <!-- Percentage Text -->
                          <div class="absolute top-1/2 start-1/2 transform -translate-y-1/2 -translate-x-1/2">
                            <span class="text-center text-white text-sm sm:text-[1rem]">{statistics?.buySharesPercentage}%</span>
                          </div>
                        </div>
                        <!-- End Circular Progress -->
                      </div>
                      <!--End Call Flow-->
                      <!--Start Put Flow-->  
                      <div class="flex flex-row items-center flex-wrap w-full px-3 sm:px-4 bg-[#262626] shadow-lg rounded-md h-20">
                        <div class="flex flex-col items-start">
                            <span class="font-medium text-gray-200 text-sm sm:text-[1rem]">Sold Shares</span>
                            <span class="text-start text-sm sm:text-[1rem] font-medium text-white">
                              {new Intl.NumberFormat("en", {
                                minimumFractionDigits: 0,
                                maximumFractionDigits: 0
                            }).format(statistics?.soldShares)}
                            </span>
                        </div>
                        <!-- Circular Progress -->
                        <div class="relative size-12 sm:size-14 ml-auto">
                          <svg class="size-full w-12 h-12 sm:w-14 sm:h-14" viewBox="0 0 36 36" xmlns="http://www.w3.org/2000/svg">
                            <!-- Background Circle -->
                            <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-[#3E3E3E]" stroke-width="3"></circle>
                            <!-- Progress Circle inside a group with rotation -->
                            <g class="origin-center -rotate-90 transform">
                              <circle cx="18" cy="18" r="16" fill="none" class="stroke-current text-[#EE5365]" stroke-width="3" stroke-dasharray="100" stroke-dashoffset={100-statistics?.soldSharesPercentage}></circle>
                            </g>
                          </svg>
                          <!-- Percentage Text -->
                          <div class="absolute top-1/2 start-1/2 transform -translate-y-1/2 -translate-x-1/2">
                            <span class="text-center text-white text-sm sm:text-[1rem]">{statistics?.soldSharesPercentage}%</span>
                          </div>
                        </div>
                        <!-- End Circular Progress -->
                        
                      </div>
                      <!--End Put Flow-->
                
                
                        </div>
                      </div>
                    <!--End Widget-->
                    {/if}


                      <div class="flex justify-start items-center w-full m-auto rounded-none sm:rounded-lg mb-4 overflow-x-scroll no-scrollbar">
                          <table class="table table-sm table-pin-rows table-compact rounded-none sm:rounded-md w-full bg-[#09090B] border-bg-[#09090B] m-auto">
                            <thead>
                              <tr class="bg-[#09090B] shadow-md">
                                <th class="text-start bg-[#09090B] text-white text-[1rem] font-semibold">
                                  Name
                                </th>
                                <th class="text-end bg-[#09090B] text-white text-[1rem] font-semibold">
                                  Date
                                </th>
                                <th class="text-end bg-[#09090B]  text-white text-[1rem] font-semibold">
                                  Shares
                                </th>
                                <th class="text-end bg-[#09090B]  text-white text-[1rem] font-semibold">
                                  Price
                                </th>
                                <th class="text-white font-semibold text-end text-[1rem]">Value</th>
                              </tr>
                            </thead>
                            <tbody>
                              {#each (data?.user?.tier === 'Pro' ? insiderTradingList : insiderTradingList?.slice(0,3)) as item, index}
                              {#if item?.price > 0}
                              <tr class="text-white odd:bg-[#27272A] {index+1 === insiderTradingList?.slice(0,3)?.length && data?.user?.tier !== 'Pro' ? 'opacity-[0.1]' : ''}">
      
                                <td class="text-white text-sm sm:text-[1rem] border-b border-[#09090B] whitespace-nowrap">
                                  <div class="flex flex-col">
                                    <span class="">{formatString(item?.reportingName)?.replace('/de/','')}</span>
                                    <span class="text-sm text-white/80">{extractOfficeInfo(item?.typeOfOwner)}</span>
                                  </div>
                                </td>
                               
                                  <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                      {new Date(item?.transactionDate)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' })}
                                  </td>

                                  <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                      {abbreviateNumber(item?.securitiesTransacted)}
                                  </td>
                                  <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                    {item?.price?.toFixed(2)}
                                  </td>
                                  <td class="font-medium text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                    
                                   <div class="flex flex-row items-center justify-end">
                                    {#if transactionStyles[item?.transactionType]}
                                      <div class="{transactionStyles[item?.transactionType]?.class}">{abbreviateNumber(item?.securitiesTransacted * item?.price)}</div>
                                      <div class="{transactionStyles[item?.transactionType]?.class} {transactionStyles[item?.transactionType]?.border} ml-2 px-1.5 py-1.5 border text-center rounded-lg text-xs font-semibold">
                                        {transactionStyles[item?.transactionType].text}
                                      </div>
                                    {:else}
                                      <span class="text-gray-300">n/a</span>
                                    {/if}
                                  </div>
                                  </td>
                              </tr>
                              {/if}
                            {/each}
                            </tbody>
                          </table>
                      </div>

                

                      {#if rawData?.length === insiderTradingList?.length}
                      <label on:click={backToTop} class="w-32 py-1.5 mt-10 hover:bg-white hover:bg-opacity-[0.05] cursor-pointer m-auto flex justify-center items-center border border-slate-800 rounded-full">
                        Back to top
                      </label>
                      {/if}

                      <UpgradeToPro data={data} title="Access {$displayCompanyName}'s insider transactions to track executive selling and purchasing activity"/>


          
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
      

    
                </div>
            </div>
         </div>
    </section>


    <style>
      .app {
          height: 400px;
          width: 100%;
      }
      
      @media (max-width: 560px) {
          .app {
              width: 100%;
              height: 300px;
          }
      }
  
      .chart {
          width: 100%;
      }
  </style>