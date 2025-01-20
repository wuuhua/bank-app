<template>
  <h1 class="text-5xl -ml-1 font-light">
    台灣銀行代碼查詢
  </h1>
  <div class="flex gap-2 mt-2">
    <div class="flex flex-col relative">
      <label for="bank-name" class="font-bold text-gray-700">
        銀行名稱
      </label>
      <input id="bank-name" type="text" placeholder="請輸入關鍵字或銀行代碼" class="w-full p-2 border border-zinc-400 rounded font-normal text-zinc-400 text-ellipsis focus:outline-none focus:ring-1 focus:ring-blue-500">
      <i class="fa-solid fa-angle-down absolute right-3 top-9 text-zinc-300"></i>
      <p class="text-sm font-light text-zinc-400 p-1">使用下拉選單或直接輸入關鍵字查詢</p>
      <div class="absolute top-0 right-0 mt-2 mr-2 bg-white border border-zinc-400 rounded shadow-lg max-h-16 overflow-y-auto z-10">
        <div></div>
      </div>
    </div>
    <div class="flex flex-col relative">
      <label for="branch-name" class="font-bold text-gray-700">
        分行名稱
      </label>
      <input id="branch-name" type="text" placeholder="請選擇分行名稱" class="w-full p-2 border border-zinc-400 rounded font-normal text-zinc-500 text-ellipsis focus:outline-none focus:ring-1 focus:ring-blue-500 disabled:bg-zinc-500">
      <i class="fa-solid fa-angle-down absolute right-3 top-9 text-zinc-300"></i>
      <div class="absolute top-0 right-0 mt-2 mr-2 bg-white border border-zinc-400 rounded shadow-lg max-h-16 overflow-y-auto z-10">
        <div></div>
      </div>
    </div>
  </div>
</template>
