<script lang='ts'>
  import { goto } from '$app/navigation';
  import { numberOfUnreadNotification, screenWidth } from '$lib/store';
	import { abbreviateNumber } from '$lib/utils';
  import { onMount } from 'svelte';
  import UpgradeToPro from '$lib/components/UpgradeToPro.svelte';
  import ArrowLogo from "lucide-svelte/icons/move-up-right";

  
    export let data;
    let cloudFrontUrl = import.meta.env.VITE_IMAGE_URL;

    let isLoaded = false;
    let rawData = []
    let stockList =  [];
  

  function handleMode(i) {
    activeIdx = i;
    if(activeIdx === 0) {
      rawData = data?.getSentimentTracker?.twitter;
    } else if (activeIdx === 1) {
      rawData = data?.getSentimentTracker?.stocktwits;
    }
    stockList = rawData?.slice(0,50)
  }
  
  const tabs = [
      {
        title: "Twitter",
      },
      {
        title: "Stocktwits",
      },
    ];
  
let activeIdx = 0;
  
async function handleScroll() {
    const scrollThreshold = document.body.offsetHeight * 0.8; // 80% of the website height
    const isBottom = window.innerHeight + window.scrollY >= scrollThreshold;
    if (isBottom && stockList?.length !== rawData?.length) {
        const nextIndex = stockList?.length;
        const filteredNewResults = rawData?.slice(nextIndex, nextIndex + 25);
        stockList = [...stockList, ...filteredNewResults];
    }
  }

  
    
  onMount(() => {
    rawData = data?.getSentimentTracker?.twitter ?? [];
    stockList = rawData?.slice(0,50) ?? []

    isLoaded = true;


      if(data?.user?.tier === 'Pro') {
        window.addEventListener('scroll', handleScroll);
        return () => {
            window.removeEventListener('scroll', handleScroll);
        };
      }

  })
  
  
  $: charNumber = $screenWidth < 640 ? 15 : 40;

  
        
  </script>
  
  <svelte:head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width" />
  <title>
      {$numberOfUnreadNotification > 0 ? `(${$numberOfUnreadNotification})` : ''} Sentiment Tracker · stocknear
  </title>
  <meta name="description" content={`Stay ahead of the market with our real-time filtering of the most discussed and bullish stocks on social media. Discover trending stocks and make informed investment decisions today!`} />
  
  <!-- Other meta tags -->
  <meta property="og:title" content={`Sentiment Tracker · stocknear`}/>
  <meta property="og:description" content={`Stay ahead of the market with our real-time filtering of the most discussed and bullish stocks on social media. Discover trending stocks and make informed investment decisions today!`} />
  <meta property="og:type" content="website"/>
  <!-- Add more Open Graph meta tags as needed -->
  
  <!-- Twitter specific meta tags -->
  <meta name="twitter:card" content="summary_large_image"/>
  <meta name="twitter:title" content={`Sentiment Tracker · stocknear`}/>
  <meta name="twitter:description" content={`Stay ahead of the market with our real-time filtering of the most discussed and bullish stocks on social media. Discover trending stocks and make informed investment decisions today!`} />
  <!-- Add more Twitter meta tags as needed -->
  
  </svelte:head>
  
      
  
  <section class="w-full max-w-3xl sm:max-w-screen-2xl overflow-hidden min-h-screen pt-5 pb-40">
        
    <div class="text-sm sm:text-[1rem] breadcrumbs ml-4">
      <ul>
        <li><a href="/" class="text-gray-300">Home</a></li>
        <li class="text-gray-300">Sentiment Tracker</li>
      </ul>
    </div>
            
    <div class="w-full overflow-hidden m-auto mt-5">
        
      <div class="sm:p-0 flex justify-center w-full m-auto overflow-hidden ">
          <div class="relative flex justify-center items-start overflow-hidden w-full">
              
            
            <main class="w-full lg:w-3/4 lg:pr-5">
               
                <div class="w-full m-auto sm:bg-[#27272A] sm:rounded-xl h-auto pl-10 pr-10 pt-5 sm:pb-10 sm:pt-10 mt-3 mb-8">
                  <div class="grid grid-cols-1 sm:grid-cols-2 gap-10">
                
                    <!-- Start Column -->
                    <div>
                      <div class="flex flex-row justify-center items-center">
                        <h1 class="text-3xl sm:text-4xl text-white text-center font-bold mb-5">
                          Sentiment Tracker
                        </h1>
                      </div>
            
                      <span class="text-white text-md font-medium text-center flex justify-center items-center ">
                        Find out the top stocks discussed in social media.
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
                      
                      
                      <div class="z-1 absolute -top-3">
                          <img class="w-36 " src={cloudFrontUrl+'/assets/sentiment_tracker_logo.png'} alt="logo" loading="lazy">
                        </div>
                    </div>
                    <!-- End Column -->
                  </div>
            
                 
            
            
                </div>

                {#if isLoaded}

                <div class="mb-8 w-full text-center sm:text-start sm:flex sm:flex-row sm:items-center m-auto text-gray-100 border border-gray-800 sm:rounded-lg h-auto p-5 ">
                  <svg class="w-5 h-5 inline-block sm:mr-2 flex-shrink-0" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256"><path fill="#a474f6" d="M128 24a104 104 0 1 0 104 104A104.11 104.11 0 0 0 128 24m-4 48a12 12 0 1 1-12 12a12 12 0 0 1 12-12m12 112a16 16 0 0 1-16-16v-40a8 8 0 0 1 0-16a16 16 0 0 1 16 16v40a8 8 0 0 1 0 16"/></svg>
                    We update our data in realtime to provide you with the latest trends and the most bullish stocks being discussed on Twitter and StockTwits
                </div>

                
                  <div class="bg-[#313131] w-fit relative m-auto sm:m-0 sm:mr-auto flex sm:flex-wrap items-center justify-center rounded-lg p-1">
                    {#each tabs as item, i}
                    <button
                        on:click={() => handleMode(i)}
                        class="group relative z-[1] rounded-full px-6 py-1 {activeIdx === i
                        ? 'z-0'
                        : ''} "
                    >
                        {#if activeIdx === i}
                            <div
                            class="absolute inset-0 rounded-lg bg-purple-600"
                            ></div>
                        {/if}
                        <span class="relative text-[1rem] sm:text-[1rem] block font-semibold duration-200 text-white">
                          {#if i === 0}
                            <svg class="inline-block w-4 h-4 -mt-1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"><path fill="#fff" d="M5.026 15c6.038 0 9.341-5.003 9.341-9.334q.002-.211-.006-.422A6.7 6.7 0 0 0 16 3.542a6.7 6.7 0 0 1-1.889.518a3.3 3.3 0 0 0 1.447-1.817a6.5 6.5 0 0 1-2.087.793A3.286 3.286 0 0 0 7.875 6.03a9.32 9.32 0 0 1-6.767-3.429a3.29 3.29 0 0 0 1.018 4.382A3.3 3.3 0 0 1 .64 6.575v.045a3.29 3.29 0 0 0 2.632 3.218a3.2 3.2 0 0 1-.865.115a3 3 0 0 1-.614-.057a3.28 3.28 0 0 0 3.067 2.277A6.6 6.6 0 0 1 .78 13.58a6 6 0 0 1-.78-.045A9.34 9.34 0 0 0 5.026 15"/></svg>
                          {:else}
                            <svg class="inline-block w-6 h-6 -mt-1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="none" stroke="#fff" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 3L8 7.5l8 4.5m-8 0l8 4.5L8 21"/></svg>
                          {/if}  
                          {item.title}
                        </span>
                    </button>
                    {/each}
                    </div>


                <div class="w-screen sm:w-full m-auto mt-20 sm:mt-10">
                  
                  
                  
                    <div class="w-screen sm:w-full m-auto rounded-none sm:rounded-lg mb-4 overflow-x-scroll sm:overflow-hidden">
                      <table class="table table-sm table-compact rounded-none sm:rounded-md w-full bg-[#09090B] border-bg-[#09090B] m-auto">
                        <thead>
                          <tr class="bg-[#09090B] border-b border-[#27272A]">
                            <th class="text-end bg-[#09090B] text-white text-[1rem] font-semibold">
                              #
                            </th>
                            <th class="text-start bg-[#09090B] text-white text-[1rem] font-semibold">
                              Symbol
                            </th>
  
                            <th class=" text-start bg-[#09090B] text-white text-[1rem] font-semibold">
                              Name
                            </th>
  
                            <th class="text-end bg-[#09090B] text-white text-[1rem] font-semibold">
                              Market Cap
                            </th>

                            <th class="text-end bg-[#09090B] text-white text-[1rem] font-semibold">
                              Price
                            </th>
                            <th class="text-end bg-[#09090B] text-white text-[1rem] font-semibold">
                              % Change
                            </th>
                            <th class="text-end bg-[#09090B] text-white text-[1rem] font-semibold">
                              Sentiment
                            </th>
                          </tr>
                        </thead>
                        <tbody>
                          {#each stockList as item, index}
  
                          <tr class="sm:hover:bg-[#245073] border-b border-[#27272A] sm:hover:bg-opacity-[0.2] odd:bg-[#27272A] {index+1 === stockList?.length && data?.user?.tier !== 'Pro' ? 'opacity-[0.1]' : ''}">
                            <td class="text-white text-sm sm:text-[1rem] font-medium text-white text-end">
                              {item?.rank}
                            </td>
  
                            <td class="text-sm sm:text-[1rem] text-start">
                              <a href={"/stocks/"+item?.symbol} class="sm:hover:text-white text-blue-400">
                                {item?.symbol}
                              </a>
                            </td>
  
                            <td class="whitespace-nowrap text-white text-sm sm:text-[1rem] text-white text-start">
                                {item?.name?.length > charNumber ? item?.name?.slice(0,charNumber) + "..." : item?.name}
                            </td>
                            
                             <td class="text-end text-sm sm:text-[1rem] font-medium text-white whitespace-nowrap">
                              {abbreviateNumber(item?.marketCap)}
                            </td>
  
                            <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap font-medium text-white">
                                {item?.price}
                            </td>
      
                            <td class="text-sm sm:text-[1rem] whitespace-nowrap {item?.changesPercentage >= 0 ? 'text-[#37C97D]' : 'text-[#FF2F1F]'} text-end">
                                {item?.changesPercentage > 0 ? '+' : ''}{item?.changesPercentage}%
                              </td>

                            <td class="text-end text-sm sm:text-[1rem] font-medium whitespace-nowrap {item?.sentiment >= 55 ? 'text-[#37C97D]' : item?.sentiment >= 50 ? 'text-[#E57C34]' : 'text-[#FF2F1F]'}">
                              
                              <div class="flex flex-row items-center justify-end">
                                      <div class="">{item?.sentiment >= 80 ? 'Very Bullish' : item?.sentiment >= 55 ? 'Bullish' : item?.sentiment > 50 ? 'Mixed' : 'Bearish'}</div>
                                      <div class="ml-2 px-1.5 py-1.5 border text-center rounded-lg text-xs font-semibold">
                                        {item?.sentiment}
                                      </div>
                                  </div>
                            </td>
                          </tr>
                        {/each}
                        </tbody>
                      </table>
                  </div>
                    <UpgradeToPro data={data} title="Get the latest stock trends from social media to never miss out the next hype"/>
  
                </div>

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

              <div on:click={() => goto('/cramer-tracker')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
                  <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
                      <div class="w-full flex justify-between items-center p-3 mt-3">
                      <h2 class="text-start text-xl font-semibold text-white ml-3">
                        Cramer Tracker 📉
                      </h2>
                      <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
                      </div>
                      <span class="text-white p-3 ml-3 mr-3">
                          Follow Jim Cramer latest stock picks
                      </span>
                  </div>
              </div>

              <div on:click={() => goto('/reddit-tracker')} class="w-full bg-[#141417] duration-100 ease-out sm:hover:text-white text-gray-400 sm:hover:border-gray-700 border border-gray-800 rounded-lg h-fit pb-4 mt-4 cursor-pointer">
                  <div class="w-auto lg:w-full p-1 flex flex-col m-auto px-2 sm:px-0">
                      <div class="w-full flex justify-between items-center p-3 mt-3">
                      <h2 class="text-start text-xl font-semibold text-white ml-3">
                        Reddit Tracker 🚀
                          
                      </h2>
                      <ArrowLogo class="w-8 h-8 mr-3 flex-shrink-0"/>
                      </div>
                      <span class="text-white p-3 ml-3 mr-3">
                          Get the latest trends of r/Wallstreetbets
                      </span>
                  </div>
              </div>

            </aside>


          </div>
      </div>
  
    
    </div>
        
        
    
  </section>
  
  