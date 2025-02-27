<script lang="ts">
  import {numberOfUnreadNotification, displayCompanyName, stockTicker} from '$lib/store';
  import { onMount } from 'svelte';
  
  import { Chart } from 'svelte-echarts'
  import { init, use } from 'echarts/core'
  import { LineChart, BarChart } from 'echarts/charts'
  import { GridComponent, TooltipComponent } from 'echarts/components'
  import { CanvasRenderer } from 'echarts/renderers'
  use([LineChart, BarChart, TooltipComponent, GridComponent, CanvasRenderer])
  
  
  export let data;
  let isLoaded = false;
  let dateDistance;
  let rawData = data?.getStockDividend;
  let optionsDividend;
      
      
    
  let exDividendDate = rawData?.history?.at(0)?.date;
  let dividendYield = rawData?.dividendYield;
  let annualDividend = rawData?.annualDividend;
  let payoutFrequency = rawData?.payoutFrequency;
  let payoutRatio = rawData?.payoutRatio;
  let dividendGrowth = rawData?.dividendGrowth;
      
  
    
  
  async function plotDividend() {
  
    // Combine the data into an array of objects to keep them linked
    const combinedData = rawData?.history?.map(item => ({
        date: item?.paymentDate,
        dividend: item?.adjDividend?.toFixed(2)
    }));

    // Sort the combined data array based on the date
    combinedData.sort((a, b) => new Date(a?.date) - new Date(b?.date));

    // Separate the sorted data back into individual arrays
    const dates = combinedData.map(item => item.date);
    const dividendList = combinedData.map(item => item.dividend);

  
    const options = {
       tooltip: {
          trigger: 'axis',
          hideDelay: 100, // Set the delay in milliseconds
      },
      animation: false,
          grid: {
          left: '3%',
          right: '3%',
          bottom: '10%',
          top: '10%',
          containLabel: true
          },
        xAxis: {
          data: dates,
          type: 'category',
          axisLabel: {
            color: '#fff',
          },
          splitLine: {
              show: false, // Disable x-axis grid lines
          },
        },
        yAxis: [
          {
          type: 'value',
          splitLine: {
                show: false, // Disable x-axis grid lines
          },
          
          axisLabel: {
            show: false // Hide y-axis labels
          }
    },
        ],
        series: [
          {
            name: 'Dividend per Share',
            data: dividendList,
            type: 'bar',
            smooth: true,
          },
        ],
      };
      
      
          return options;
  }
  
  
  
  
  
  onMount(async() => {
    optionsDividend = await plotDividend()
    isLoaded = true;
  })
  
  
  </script>
  
   
  <svelte:head>
  
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>
      {$numberOfUnreadNotification > 0 ? `(${$numberOfUnreadNotification})` : ''} {$displayCompanyName} ({$stockTicker}) Dividend History, Dates & Yield · stocknear
    </title>
  
    <meta name="description" content={`Get the latest dividend data for ${$displayCompanyName} (${$stockTicker}) stock price quote with breaking news, financials, statistics, charts and more.`}>
    <!-- Other meta tags -->
    <meta property="og:title" content={`${$displayCompanyName} (${$stockTicker}) Dividend History, Dates & Yield · stocknear`}/>
    <meta property="og:description" content={`Get the latest dividend data for ${$displayCompanyName} (${$stockTicker}), including dividend history, yield, key dates, growth and other metrics.`} />
    <meta property="og:type" content="website"/>
    <!-- Add more Open Graph meta tags as needed -->
  
    <!-- Twitter specific meta tags -->
    <meta name="twitter:card" content="summary_large_image"/>
    <meta name="twitter:title" content={`${$displayCompanyName} (${$stockTicker}) Dividend History, Dates & Yield · stocknear`}/>
    <meta name="twitter:description" content={`Get the latest dividend data for ${$displayCompanyName} (${$stockTicker}) stock price quote with breaking news, financials, statistics, charts and more.`} />
    <!-- Add more Twitter meta tags as needed -->
  
  </svelte:head>
    
  
            
      <section class="w-full bg-[#09090B] overflow-hidden text-white h-full mb-40 sm:mb-0">
          <div class="w-full flex h-full overflow-hidden">
              <div class="w-full relative flex justify-center items-center overflow-hidden">
                    <div class="sm:p-7 w-full m-auto mt-2 sm:mt-0">
                          <div class="w-full mb-6">
                              <h1 class="text-2xl sm:text-3xl text-gray-200 font-bold mb-4">
                                  Dividends
                              </h1>
      
                  
                              
                            
                          <div class="w-full text-white text-start p-3 sm:p-5 mb-10 rounded-lg sm:flex sm:flex-row sm:items-center border border-slate-800 text-sm sm:text-[1rem]">
                            <svg class="w-6 h-6 flex-shrink-0 inline-block sm:mr-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256"><path fill="#a474f6" d="M128 24a104 104 0 1 0 104 104A104.11 104.11 0 0 0 128 24m-4 48a12 12 0 1 1-12 12a12 12 0 0 1 12-12m12 112a16 16 0 0 1-16-16v-40a8 8 0 0 1 0-16a16 16 0 0 1 16 16v40a8 8 0 0 1 0 16"/></svg>
  
                            
                                {#if rawData?.history?.length !== 0}
                                {#if !dateDistance}
                                    {$displayCompanyName} has an annual dividend of
                                      ${annualDividend}
                                    per share, with a forward yield of
                                      {dividendYield}%.
                                    The dividend is paid every
                                    {payoutFrequency === 4 ? '3 months' : payoutFrequency === 2 ? '6 months' : payoutFrequency === 1 ? '12 months' : 'n/a'} 
                                    and the last ex-dividend date was
                                      {new Date(exDividendDate)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' })}
                                {:else}
                                  {$displayCompanyName} issued its most recent dividend on 
                                  {new Date(rawData?.history?.at(0)?.date)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' })}. 
                                  Since then, the company has not distributed any further dividends for over 12 months.
                                {/if}
  
                             
                              {:else}
                              No dividend history available for {$displayCompanyName}.
                              {/if}
                            
                          </div>
      
                          </div>
      
                        {#if rawData?.history?.length !== 0}
                        
                        <div class="mb-4 grid grid-cols-2 grid-rows-1 divide-gray-500 rounded-lg border border-gray-600 bg-[#272727] shadow md:grid-cols-3 md:grid-rows-1 divide-x">
                        <div class="p-4 bp:p-5 sm:p-6">
                          <label class="mr-1 cursor-pointer flex flex-row items-center text-white text-[1rem]">
                            Dividend Yield
                          </label>   
                            <div class="mt-1 break-words font-semibold leading-8 text-light text-xl">
                              {dividendYield !== '0.00' ? dividendYield : '0'}%
                            </div> 
                        </div>
                        <div class="p-4 bp:p-5 sm:p-6 border-l border-b border-contrast ">
                          <label class="mr-1 cursor-pointer flex flex-row items-center text-white text-[1rem]">
                            Annual Dividend
                          </label>  
            
                            <div class="mt-1 break-words font-semibold leading-8 text-light text-xl">
                             {annualDividend !== '0.00' ? annualDividend : '0'}
                            </div> 
                        </div>
                        <div class="p-4 bp:p-5 sm:p-6 border-r border-contrast ">
                          <label class="mr-1 cursor-pointer flex flex-row items-center text-white text-[1rem]">
                            Ex-Dividend Date
                          </label>    
                         
                            <div class="mt-1 break-words font-semibold leading-8 text-light text-xl">
                             {new Date(exDividendDate)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' })}
                            </div> 
                        </div>

                        <div class="p-4 bp:p-5 sm:p-6 border-t border-r border-contrast ">
                          <label class="mr-1 cursor-pointer flex flex-row items-center text-white text-[1rem]">
                            Payout Frequency
                          </label>    
                         
                            <div class="mt-1 break-words font-semibold leading-8 text-light text-xl">
                             {payoutFrequency === 4 ? 'Quartely' : payoutFrequency === 2 ? 'Half-Yearly' : payoutFrequency === 1 ? 'Annually' : 'n/a'}
                            </div> 
                        </div>
                        <div class="p-4 bp:p-5 sm:p-6 border-t border-r border-contrast ">
                          <label class="mr-1 cursor-pointer flex flex-row items-center text-white text-[1rem]">
                            Payout Ratio 
                          </label>    
                         
                            <div class="mt-1 break-words font-semibold leading-8 text-light text-xl">
                             {payoutRatio !== '0.00' ? payoutRatio : '0'}%
                            </div> 
                        </div>
                        <div class="p-4 bp:p-5 sm:p-6 border-t border-r border-contrast ">
                          <label class="mr-1 cursor-pointer flex flex-row items-center text-white text-[1rem]">
                            Dividend Growth
                          </label>    
                         
                            <div class="mt-1 break-words font-semibold leading-8 text-light text-xl">
                             {dividendGrowth !== 'NaN' ? dividendGrowth+'%' : '-'}
                            </div> 
                        </div>
  
                           
                      </div>



                            <div class="flex flex-col sm:flex-row items-start sm:items-center w-full mt-14 mb-8">
          
                              <h3 class="text-xl text-white font-semibold ">
                                Dividends History
                              </h3>
          
        
          
                            </div>
        
  
                    {#if isLoaded} 
                          {#if rawData?.history?.length !== 0 && optionsDividend}
        
                          <div class="app w-full">
                            <Chart {init} options={optionsDividend} class="chart" />
                          </div>
        
                          
                              <div class="overflow-x-scroll no-scrollbar flex justify-start items-center w-full m-auto shadow-md rounded-none sm:rounded-lg mb-4">
                                <table class="table table-sm table-compact flex justify-start items-center w-full m-auto">
                                  <thead>
                                    <tr class="bg-[#09090B] border-b-slate-600 shadow-md">
                                      <th class="text-start bg-[#09090B] border-b border-[#09090B] text-white text-sm font-semibold">
                                        Ex-Divid. Date
                                      </th>
                                      <th class="text-end bg-[#09090B] border-b border-[#09090B] text-white text-sm font-semibold">
                                        Cash Amount
                                      </th>
                                      <th class="text-end bg-[#09090B] border-b border-[#09090B] text-white text-sm font-semibold">
                                        Record Date
                                      </th>
                                      <th class="text-end bg-[#09090B] border-b border-[#09090B] text-white text-sm font-semibold">
                                        Pay Date
                                      </th>
                                    </tr>
                                  </thead>
                                  <tbody class="shadow-md">
                                    {#each rawData?.history as item}
                                    <tr class="text-gray-200 odd:bg-[#27272A]">
                                      <td class="text-start text-sm sm:text-[1rem] whitespace-nowrap text-white font-medium border-b border-[#09090B]">
                                        {new Date(item?.date)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' })}
                                      </td>
                                      <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                        {item?.adjDividend?.toFixed(3)}
                                      </td>
                                      <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                        {item?.recordDate?.length !== 0 ? new Date(item?.recordDate)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' }) : 'n/a'}
                                      </td>
                                      <td class="text-end text-sm sm:text-[1rem] whitespace-nowrap text-white border-b border-[#09090B]">
                                        {item?.paymentDate?.length !== 0 ? new Date(item?.paymentDate)?.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', daySuffix: '2-digit' }) : 'n/a'}
                                      </td>
                                    </tr>
                                  {/each}
                                  </tbody>
                                </table>
                              </div>
                              <span class="text-gray-200 text-sm italic">
                                * Dividend amounts are adjusted for stock splits when applicable.
                              </span>
        
                          {:else}
                          <h1 class="text-xl m-auto flex justify-center text-gray-200 mb-4 mt-10">
                            No history found
                          </h1>
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