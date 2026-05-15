<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>敏敏Jasmine的工作紀錄</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; background-color: #f3f4f6; }
        .tab-active { border-bottom: 4px solid #3b82f6; color: #3b82f6; font-weight: bold; }
        .tab-inactive { border-bottom: 4px solid transparent; color: #6b7280; }
        .tab-inactive:hover { color: #3b82f6; }
        
        /* 狀態顏色設定 */
        .status-1 { background-color: #dcfce7; border-left: 4px solid #22c55e; } 
        .status-2 { background-color: #ffedd5; border-left: 4px solid #f97316; } 
        .status-3 { background-color: #fee2e2; border-left: 4px solid #ef4444; } 
        .status-done { background-color: #f9fafb; opacity: 0.6; color: #9ca3af; border-left: 4px solid #d1d5db;}
        
        /* 行事曆格子特效 */
        .calendar-cell { min-height: 90px; transition: all 0.2s; }
        .calendar-cell:hover { box-shadow: inset 0 0 0 2px #bfdbfe; }
        
        /* 隱藏滾動條但保留功能 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="p-4 md:p-6 lg:p-8">

    <div class="max-w-7xl mx-auto space-y-6">
        
        <!-- 頁首 -->
        <header class="flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
                <h1 class="text-2xl font-bold text-gray-800"><i class="fa-solid fa-seedling text-green-600 mr-2"></i>敏敏Jasmine的工作紀錄</h1>
                <p class="text-sm text-gray-500 mt-1">工作排程 / 當月行事曆總表 / 分校待辦狀態</p>
            </div>
            <div class="text-right text-sm text-gray-500 bg-white px-4 py-2 rounded-lg shadow-sm border border-gray-100">
                操作日期：<input type="date" id="input-date" class="font-bold text-blue-600 text-base bg-transparent outline-none cursor-pointer">
            </div>
        </header>

        <!-- 第一排：輸入與待辦 -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            
            <!-- 新增工作紀錄 -->
            <section class="bg-white rounded-xl shadow-sm p-5 border border-gray-100 lg:col-span-1">
                <h2 class="text-lg font-bold text-gray-700 mb-4 border-b pb-2"><i class="fa-solid fa-pen-to-square mr-2"></i>記錄與計畫 (輸入)</h2>
                <form id="work-form" class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-600 mb-1">工作內容</label>
                        <input type="text" id="input-task" placeholder="例如：招募計畫撰寫..." required class="w-full border border-gray-300 rounded-lg p-2 focus:ring-2 focus:ring-blue-500 outline-none transition">
                    </div>
                    <div class="grid grid-cols-2 gap-3">
                        <div>
                            <label class="block text-sm font-medium text-gray-600 mb-1">分類</label>
                            <select id="input-category" class="w-full border border-gray-300 rounded-lg p-2 focus:ring-2 focus:ring-blue-500 outline-none bg-white">
                                <option value="HRD">HRD</option>
                                <option value="HRM">HRM</option>
                                <option value="人事">人事</option>
                                <option value="訪談關心">訪談關心</option>
                                <option value="團隊管理與會議">團隊管理與會議</option>
                                <option value="獎項與榮譽">獎項與榮譽</option>
                                <option value="財務">財務</option>
                                <option value="其他">其他</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-600 mb-1">耗費時間 <span class="text-xs text-orange-500 font-bold">(0為待辦)</span></label>
                            <input type="number" id="input-hours" step="0.5" min="0" placeholder="0" value="0" required class="w-full border border-gray-300 rounded-lg p-2 focus:ring-2 focus:ring-blue-500 outline-none transition">
                        </div>
                    </div>
                    <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-medium py-2.5 px-4 rounded-lg transition duration-200 shadow-sm">
                        <i class="fa-solid fa-plus mr-1"></i> 加入紀錄 / 待辦
                    </button>
                </form>
            </section>

            <!-- 待辦清單 (Todo List) -->
            <section class="bg-white rounded-xl shadow-sm p-5 border border-gray-100 lg:col-span-2 flex flex-col" style="max-height: 280px;">
                <h2 class="text-lg font-bold text-gray-700 mb-4 border-b pb-2 flex justify-between items-center">
                    <span><i class="fa-solid fa-list-check mr-2"></i>未完成待辦清單 (待輸入時數)</span>
                    <span class="text-xs font-normal text-gray-500 bg-orange-50 text-orange-600 px-2 py-1 rounded border border-orange-100">完成後將移入行事曆</span>
                </h2>
                
                <div id="todo-list" class="overflow-y-auto no-scrollbar flex-grow space-y-2 pr-1 grid grid-cols-1 md:grid-cols-2 gap-2 content-start">
                    <!-- 待辦動態生成 -->
                </div>
            </section>
        </div>

        <!-- 第二排：當月行事曆總表 -->
        <section class="bg-white rounded-xl shadow-sm p-5 border border-gray-100">
            <h2 class="text-lg font-bold text-gray-700 mb-4 border-b pb-2 flex flex-col md:flex-row md:items-center justify-between gap-3">
                <div class="flex items-center space-x-3">
                    <i class="fa-regular fa-calendar-days text-gray-500 text-xl hidden md:inline"></i>
                    <!-- 月份/年份切換器 -->
                    <div class="flex items-center bg-gray-100 rounded-lg p-1 shadow-sm border border-gray-200">
                        <button onclick="changeMonth(-1)" class="w-8 h-8 flex justify-center items-center rounded hover:bg-white hover:shadow-sm transition text-gray-600" title="上個月"><i class="fa-solid fa-chevron-left"></i></button>
                        <div id="calendar-title" class="px-4 font-bold text-gray-800 cursor-pointer hover:text-blue-600 transition text-lg" onclick="changeYearPrompt()" title="點擊輸入年份">2026年 5月</div>
                        <button onclick="changeMonth(1)" class="w-8 h-8 flex justify-center items-center rounded hover:bg-white hover:shadow-sm transition text-gray-600" title="下個月"><i class="fa-solid fa-chevron-right"></i></button>
                    </div>
                    <span class="text-sm font-bold text-gray-500 hidden md:inline">行事曆總表</span>
                </div>
                <span class="text-sm font-normal text-blue-600 bg-blue-50 px-2 py-1 rounded-md border border-blue-100 self-start md:self-auto"><i class="fa-solid fa-hand-pointer mr-1"></i>點擊日期可編輯工作或標記休假</span>
            </h2>
            
            <div class="border rounded-lg overflow-hidden">
                <!-- 星期表頭 -->
                <div class="grid grid-cols-7 bg-gray-50 border-b text-center text-sm font-bold text-gray-600">
                    <div class="py-2 text-red-500">日</div>
                    <div class="py-2">一</div>
                    <div class="py-2">二</div>
                    <div class="py-2">三</div>
                    <div class="py-2">四</div>
                    <div class="py-2">五</div>
                    <div class="py-2 text-green-600">六</div>
                </div>
                <!-- 日曆網格 -->
                <div id="calendar-grid" class="grid grid-cols-7 bg-gray-200 gap-px">
                    <!-- 動態生成 -->
                </div>
            </div>
        </section>

        <!-- 第三排：分校管理 -->
        <section class="bg-white rounded-xl shadow-sm p-5 border border-gray-100">
            <div class="flex flex-wrap justify-between items-center border-b pb-2 mb-4 gap-2">
                <h2 class="text-lg font-bold text-gray-700"><i class="fa-solid fa-building mr-2"></i>分校待辦與協助事項審核</h2>
                <div class="flex gap-2">
                    <button onclick="openAddTaskModal('track')" class="text-sm bg-blue-50 text-blue-600 hover:bg-blue-100 py-1.5 px-3 rounded-md font-medium transition border border-blue-100">
                        <i class="fa-solid fa-plus mr-1"></i> 新增追蹤(紅綠燈)
                    </button>
                    <button onclick="openAddTaskModal('help')" class="text-sm bg-purple-50 text-purple-600 hover:bg-purple-100 py-1.5 px-3 rounded-md font-medium transition border border-purple-100">
                        <i class="fa-solid fa-hand-holding-heart mr-1"></i> 新增協助(壓期限)
                    </button>
                </div>
            </div>
            
            <!-- 分校頁籤 -->
            <div class="flex space-x-4 mb-6 border-b border-gray-200 overflow-x-auto no-scrollbar" id="branch-tabs">
                <button onclick="switchTab('方舟')" class="pb-2 px-1 whitespace-nowrap tab-active" data-branch="方舟">方舟</button>
                <button onclick="switchTab('旭日')" class="pb-2 px-1 whitespace-nowrap tab-inactive" data-branch="旭日">旭日</button>
                <button onclick="switchTab('參天')" class="pb-2 px-1 whitespace-nowrap tab-inactive" data-branch="參天">參天</button>
                <button onclick="switchTab('心夢想')" class="pb-2 px-1 whitespace-nowrap tab-inactive" data-branch="心夢想">心夢想</button>
                <button onclick="switchTab('升大')" class="pb-2 px-1 whitespace-nowrap tab-inactive" data-branch="升大">升大</button>
            </div>

            <div class="grid grid-cols-1 xl:grid-cols-2 gap-8">
                
                <!-- 區塊 1: 我追蹤分校的代辦 -->
                <div>
                    <h3 class="text-md font-bold text-blue-800 mb-3 flex items-center bg-blue-50 px-3 py-2 rounded-t-lg border-b-2 border-blue-200">
                        <i class="fa-solid fa-radar mr-2"></i> 1. 我追蹤分校的待辦事項
                    </h3>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse min-w-[450px]">
                            <thead>
                                <tr class="text-gray-500 text-sm border-b">
                                    <th class="py-2 px-2 font-medium w-1/3">事項名稱</th>
                                    <th class="py-2 px-2 font-medium text-center w-1/3">提醒紀錄 (1綠, 2橘, 3紅)</th>
                                    <th class="py-2 px-2 font-medium text-center w-1/6">狀態</th>
                                </tr>
                            </thead>
                            <tbody id="track-list-body" class="text-sm"></tbody>
                        </table>
                    </div>
                </div>

                <!-- 區塊 2: 分校請我協助的事 -->
                <div>
                    <h3 class="text-md font-bold text-purple-800 mb-3 flex items-center bg-purple-50 px-3 py-2 rounded-t-lg border-b-2 border-purple-200">
                        <i class="fa-solid fa-hands-helping mr-2"></i> 2. 分校請我協助的事項
                    </h3>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse min-w-[450px]">
                            <thead>
                                <tr class="text-gray-500 text-sm border-b">
                                    <th class="py-2 px-2 font-medium w-1/3">事項名稱</th>
                                    <th class="py-2 px-2 font-medium text-center w-1/3">預計完成期限</th>
                                    <th class="py-2 px-2 font-medium text-center w-1/6">狀態</th>
                                </tr>
                            </thead>
                            <tbody id="help-list-body" class="text-sm"></tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- 處長追蹤清單 (等級2以上) -->
            <div class="mt-8 bg-red-50 rounded-lg p-4 border border-red-100">
                <h3 class="text-md font-bold text-red-700 mb-3 flex items-center">
                    <i class="fa-solid fa-triangle-exclamation mr-2"></i> 處長回報追蹤清單 (追蹤事項提醒次數達 2 次以上)
                </h3>
                <div class="overflow-x-auto bg-white rounded shadow-sm border border-red-100">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-red-100 text-red-800">
                            <tr>
                                <th class="py-2 px-3 font-medium w-1/6">提醒日期</th>
                                <th class="py-2 px-3 font-medium w-1/6">分校</th>
                                <th class="py-2 px-3 font-medium w-1/2">待辦事項</th>
                                <th class="py-2 px-3 font-medium w-1/6 text-center">當前等級</th>
                            </tr>
                        </thead>
                        <tbody id="escalation-list-body"></tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- 第四排：工作佔比圓餅圖 -->
        <section class="bg-white rounded-xl shadow-sm p-5 border border-gray-100">
            <h2 class="text-lg font-bold text-gray-700 mb-4 border-b pb-2"><i class="fa-solid fa-chart-pie mr-2"></i>工作分類佔比分析</h2>
            
            <!-- 分類圖例區塊 -->
            <div id="category-legend" class="flex flex-wrap justify-center gap-x-5 gap-y-2 mb-6 bg-gray-50 p-3 rounded-lg border border-gray-100"></div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <!-- 當月 -->
                <div class="flex flex-col items-center">
                    <h3 id="title-month" class="text-sm font-bold text-gray-600 mb-2">當月佔比</h3>
                    <div class="relative w-full aspect-square max-h-[200px]"><canvas id="pieMonth"></canvas></div>
                </div>
                <!-- 當季 -->
                <div class="flex flex-col items-center">
                    <h3 id="title-quarter" class="text-sm font-bold text-gray-600 mb-2">當季佔比</h3>
                    <div class="relative w-full aspect-square max-h-[200px]"><canvas id="pieQuarter"></canvas></div>
                </div>
                <!-- 當年 -->
                <div class="flex flex-col items-center">
                    <h3 id="title-year" class="text-sm font-bold text-gray-600 mb-2">當年佔比</h3>
                    <div class="relative w-full aspect-square max-h-[200px]"><canvas id="pieYear"></canvas></div>
                </div>
            </div>

            <!-- 歷史數據佔比總整理表格 (新增) -->
            <div class="mt-8 pt-6 border-t border-gray-100">
                <h3 class="text-md font-bold text-gray-700 mb-4 flex items-center">
                    <i class="fa-solid fa-table-list text-blue-500 mr-2"></i> 年度 / 季度 / 月份 工作佔比總整理
                    <span class="ml-3 text-xs font-normal text-gray-500 bg-gray-50 px-2 py-1 rounded">用於設立未來績效指標參考</span>
                </h3>
                <div class="overflow-x-auto border border-gray-200 rounded-lg shadow-sm">
                    <table class="w-full text-left text-sm whitespace-nowrap">
                        <thead id="summary-table-head" class="bg-gray-100 text-gray-600 font-bold border-b">
                            <!-- JS 動態生成表頭 -->
                        </thead>
                        <tbody id="summary-table-body" class="divide-y divide-gray-100">
                            <!-- JS 動態生成內容 -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- 第五排：後台紀錄總表 (新增) -->
        <section class="bg-white rounded-xl shadow-sm p-5 border border-gray-100 mt-6">
            <h2 class="text-lg font-bold text-gray-700 mb-4 border-b pb-2 flex justify-between items-center">
                <span><i class="fa-solid fa-database mr-2 text-gray-600"></i>後台資料庫：所有工作紀錄總表</span>
                <span class="text-xs font-normal text-gray-400">系統自動依日期排序</span>
            </h2>
            <div class="overflow-x-auto max-h-[400px] border border-gray-200 rounded-lg shadow-inner">
                <table class="w-full text-left text-sm whitespace-nowrap">
                    <thead class="bg-gray-100 text-gray-600 font-bold sticky top-0 shadow-sm z-10">
                        <tr>
                            <th class="py-3 px-4 border-b border-gray-200">日期</th>
                            <th class="py-3 px-4 border-b border-gray-200">工作內容</th>
                            <th class="py-3 px-4 border-b border-gray-200">分類</th>
                            <th class="py-3 px-4 border-b border-gray-200">耗費時間 (小時)</th>
                        </tr>
                    </thead>
                    <tbody id="backend-list-body" class="divide-y divide-gray-100 bg-white">
                        <!-- JS 動態生成 -->
                    </tbody>
                </table>
            </div>
        </section>
    </div>

    <!-- ===== 彈出視窗 (Modals) ===== -->
    
    <!-- 日曆工作詳情編輯 (點擊行事曆格子觸發) -->
    <div id="modal-day-detail" class="hidden fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl shadow-lg w-full max-w-2xl overflow-hidden flex flex-col max-h-[90vh]">
            <div class="bg-blue-600 px-4 py-3 flex justify-between items-center">
                <h3 id="day-detail-title" class="text-white font-bold text-lg"><i class="fa-regular fa-calendar-check mr-2"></i>2026-05-15 工作詳情與編輯</h3>
                <button onclick="closeDayDetailModal()" class="text-white hover:text-gray-200 text-xl"><i class="fa-solid fa-xmark"></i></button>
            </div>
            
            <div class="p-5 overflow-y-auto flex-grow space-y-4 bg-gray-50/50">
                <!-- 休假標記區 -->
                <div class="flex justify-between items-center bg-white p-4 rounded-lg shadow-sm border border-gray-100">
                    <div>
                        <span class="text-gray-700 font-bold block mb-1">當日狀態設定</span>
                        <span class="text-xs text-gray-500">如為特休或排休，請點擊右側按鈕標記</span>
                    </div>
                    <button id="btn-toggle-vacation" onclick="toggleVacation()" class="px-4 py-2 rounded-lg text-sm font-bold transition shadow-sm border">
                        <!-- JS 動態產生 -->
                    </button>
                </div>

                <!-- 該日工作紀錄列表 -->
                <div>
                    <h4 class="text-gray-700 font-bold mb-3 px-1 border-b pb-2">工作紀錄清單</h4>
                    <div id="day-detail-logs" class="space-y-3">
                        <!-- JS 動態產生編輯框 -->
                    </div>
                </div>
            </div>

            <div class="p-4 border-t bg-white flex justify-end gap-3">
                <button onclick="closeDayDetailModal()" class="bg-gray-100 hover:bg-gray-200 text-gray-700 font-medium py-2 px-5 rounded-lg transition">取消關閉</button>
                <button onclick="saveDayDetails()" class="bg-blue-600 hover:bg-blue-700 text-white font-medium py-2 px-6 rounded-lg shadow-sm transition">儲存所有變更</button>
            </div>
        </div>
    </div>


    <!-- 新增分校事項 Modal -->
    <div id="modal-add-task" class="hidden fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl shadow-lg w-full max-w-md overflow-hidden">
            <div id="modal-header-bg" class="bg-blue-600 px-4 py-3 flex justify-between items-center">
                <h3 id="modal-add-title" class="text-white font-bold">新增分校事項</h3>
                <button onclick="closeAddTaskModal()" class="text-white hover:text-gray-200"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-5 space-y-4">
                <input type="hidden" id="modal-task-type">
                <div>
                    <label class="block text-sm font-medium text-gray-600 mb-1">分校</label>
                    <select id="modal-branch" class="w-full border border-gray-300 rounded-lg p-2 outline-none bg-gray-50">
                        <option value="方舟">方舟</option>
                        <option value="旭日">旭日</option>
                        <option value="參天">參天</option>
                        <option value="心夢想">心夢想</option>
                        <option value="升大">升大</option>
                    </select>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-600 mb-1">工作名稱</label>
                    <input type="text" id="modal-task-name" placeholder="請輸入待辦或協助事項" class="w-full border border-gray-300 rounded-lg p-2 outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                
                <!-- 只有協助事項才顯示期限 -->
                <div id="modal-deadline-group" class="hidden">
                    <label class="block text-sm font-medium text-gray-600 mb-1">預計完成期限</label>
                    <input type="date" id="modal-deadline" class="w-full border border-gray-300 rounded-lg p-2 outline-none focus:ring-2 focus:ring-purple-500">
                </div>

                <div>
                    <label class="block text-sm font-medium text-gray-600 mb-1">分類標籤</label>
                    <select id="modal-category" class="w-full border border-gray-300 rounded-lg p-2 outline-none">
                        <option value="HRD">HRD</option>
                        <option value="HRM">HRM</option>
                        <option value="人事">人事</option>
                        <option value="訪談關心">訪談關心</option>
                        <option value="團隊管理與會議">團隊管理與會議</option>
                        <option value="獎項與榮譽">獎項與榮譽</option>
                        <option value="財務">財務</option>
                        <option value="其他">其他</option>
                    </select>
                </div>
                <button onclick="confirmAddTask()" id="modal-submit-btn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-medium py-2.5 rounded-lg mt-2 transition shadow-sm">確認新增</button>
            </div>
        </div>
    </div>

    <!-- 設定提醒日期 Modal -->
    <div id="modal-reminder" class="hidden fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl shadow-lg w-full max-w-sm overflow-hidden">
            <div class="bg-orange-500 px-4 py-3 flex justify-between items-center">
                <h3 class="text-white font-bold">記錄提醒日期</h3>
                <button onclick="closeReminderModal()" class="text-white hover:text-gray-200"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-5 space-y-4">
                <p class="text-sm text-gray-600" id="reminder-modal-desc"></p>
                <input type="hidden" id="reminder-task-id">
                <input type="hidden" id="reminder-level">
                
                <div>
                    <label class="block text-sm font-medium text-gray-600 mb-1">選擇提醒日期</label>
                    <input type="date" id="reminder-date-input" class="w-full border border-gray-300 rounded-lg p-2 outline-none focus:ring-2 focus:ring-orange-500">
                </div>
                
                <div class="flex gap-2 mt-4">
                    <button onclick="clearReminderDate()" class="w-1/3 bg-red-50 hover:bg-red-100 text-red-600 font-medium py-2 rounded-lg text-sm border border-red-200 transition">清除紀錄</button>
                    <button onclick="closeReminderModal()" class="w-1/3 bg-gray-100 hover:bg-gray-200 text-gray-700 font-medium py-2 rounded-lg text-sm transition">取消</button>
                    <button onclick="confirmReminderDate()" class="w-1/3 bg-orange-500 hover:bg-orange-600 text-white font-medium py-2 rounded-lg text-sm transition shadow-sm">儲存</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 完成待辦輸入時數 Modal -->
    <div id="modal-complete" class="hidden fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl shadow-lg w-full max-w-sm overflow-hidden">
            <div class="bg-green-600 px-4 py-3 flex justify-between items-center">
                <h3 class="text-white font-bold">完成待辦事項</h3>
                <button onclick="closeCompleteModal()" class="text-white hover:text-gray-200"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-5 space-y-4">
                <p class="text-sm text-gray-600">將 <strong id="complete-task-name" class="text-gray-800"></strong> 移入行事曆，請輸入實際耗費時數：</p>
                <input type="hidden" id="complete-task-id">
                
                <div>
                    <input type="number" id="complete-hours-input" step="0.5" min="0.5" value="1" class="w-full border border-gray-300 rounded-lg p-3 text-lg font-bold text-center outline-none focus:ring-2 focus:ring-green-500">
                </div>
                
                <button onclick="confirmCompleteTodo()" class="w-full bg-green-600 hover:bg-green-700 text-white font-medium py-2.5 rounded-lg transition shadow-sm">確認完成並移入行事曆</button>
            </div>
        </div>
    </div>

    <!-- 新增追蹤紀錄(文字備註) Modal -->
    <div id="modal-branch-log" class="hidden fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl shadow-lg w-full max-w-md overflow-hidden">
            <div class="bg-slate-700 px-4 py-3 flex justify-between items-center">
                <h3 class="text-white font-bold text-sm"><i class="fa-solid fa-pen mr-2"></i>新增文字追蹤紀錄</h3>
                <button onclick="closeLogModal()" class="text-white hover:text-gray-200"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-5 space-y-4">
                <p class="text-sm text-gray-600">為 <strong id="log-task-name" class="text-gray-800"></strong> 加上進度備註：</p>
                <input type="hidden" id="log-task-id">
                <textarea id="log-text-input" rows="3" class="w-full border border-gray-300 rounded-lg p-3 outline-none focus:ring-2 focus:ring-slate-500 text-sm" placeholder="例如：提醒主任有人投履歷，但他未接電話，預計下午3點再打..."></textarea>
                <div class="flex gap-2">
                    <button onclick="closeLogModal()" class="w-1/3 bg-gray-100 hover:bg-gray-200 text-gray-700 font-medium py-2 rounded-lg text-sm transition">取消</button>
                    <button onclick="confirmAddLog()" class="w-2/3 bg-slate-700 hover:bg-slate-800 text-white font-medium py-2 rounded-lg transition shadow-sm text-sm">儲存紀錄</button>
                </div>
            </div>
        </div>
    </div>


    <!-- Script 邏輯區 -->
    <script>
        // 初始化日期設定 (為了 Demo，配合截圖設定在 2026-05)
        const demoDate = '2026-05-15';
        const dateInput = document.getElementById('input-date');
        dateInput.value = demoDate; 

        // 分類顏色設定 (共用)
        const categoryColors = {
            'HRD': '#3b82f6', 'HRM': '#8b5cf6', '人事': '#10b981', '訪談關心': '#f59e0b',
            '團隊管理與會議': '#ef4444', '獎項與榮譽': '#06b6d4', '財務': '#ec4899', '其他': '#9ca3af'
        };

        // --- 資料模型 ---
        
        // 紀錄休假日期陣列 (儲存字串 YYYY-MM-DD)
        let vacationDates = ['2026-05-08']; // 範例：預設 5/8 請假

        // hours: 0 代表仍在待辦中，尚未完成
        let workLogs = [
            { id: 1, date: '2026-05-15', task: '早會與團隊對焦', category: '團隊管理與會議', hours: 1 },
            { id: 2, date: '2026-05-15', task: '人資處共識訪談', category: '訪談關心', hours: 1.5 },
            { id: 3, date: '2026-05-14', task: 'Super 8 實體教育訓練', category: 'HRD', hours: 2.5 }, 
            { id: 4, date: '2026-05-15', task: '準備下週新人培訓PPT', category: 'HRD', hours: 0 }, // 0=待辦
            { id: 5, date: '2026-05-16', task: '核對本月部門費用', category: '財務', hours: 0 }    // 0=待辦
        ];

        let currentBranch = '方舟';
        
        // type: 'track' = 追蹤事項(紅綠燈), 'help' = 協助事項(押期限)
        // 新增 logs: [] 用於儲存文字紀錄
        let branchTasks = [
            { id: 1, branch: '方舟', type: 'track', name: '確認新進夥伴合約與勞健保', category: '人事', reminderDates: {1:'', 2:'', 3:''}, logs: [], done: false },
            { id: 2, branch: '旭日', type: 'track', name: '店長月度面談', category: '訪談關心', reminderDates: {1:'2026-05-14', 2:'', 3:''}, logs: [{date: '05/14 10:30', text: '已傳Line提醒主任約時間'}], done: false },
            { id: 3, branch: '方舟', type: 'track', name: '辦理離職手續追蹤', category: 'HRM', reminderDates: {1:'', 2:'2026-05-13', 3:''}, logs: [], done: false },
            { id: 4, branch: '參天', type: 'help', name: '提供招募海報圖檔', category: '人事', deadline: '2026-05-18', logs: [], done: false },
            { id: 5, branch: '方舟', type: 'help', name: '協助申請補助金資料', category: 'HRM', deadline: '2026-05-20', logs: [], done: false }
        ];

        let charts = { month: null, quarter: null, year: null };

        // --- 渲染分類圖例 ---
        function renderLegend() {
            const legendContainer = document.getElementById('category-legend');
            if (!legendContainer) return;
            legendContainer.innerHTML = '';
            for (const [category, color] of Object.entries(categoryColors)) {
                legendContainer.innerHTML += `
                    <div class="flex items-center text-sm text-gray-700 font-medium">
                        <span class="w-3 h-3 rounded-full mr-1.5 shadow-sm" style="background-color: ${color}"></span>
                        ${category}
                    </div>
                `;
            }
        }

        // --- 核心邏輯：更新所有畫面 ---
        function updateDashboard() {
            renderTodos();
            renderCalendarGrid();
            updateCharts();
            renderSummaryTable();
            renderBackendList();
        }

        // --- 1. 渲染待辦清單 ---
        function renderTodos() {
            const allTodoLogs = workLogs.filter(log => log.hours === 0);
            allTodoLogs.sort((a, b) => new Date(a.date) - new Date(b.date));
            
            const todoContainer = document.getElementById('todo-list');
            todoContainer.innerHTML = '';

            if (allTodoLogs.length === 0) {
                todoContainer.innerHTML = `<p class="text-sm text-gray-400 italic col-span-2 text-center py-6">太棒了！目前清空了所有待辦</p>`;
                return;
            }

            allTodoLogs.forEach(log => {
                const color = categoryColors[log.category] || '#9ca3af';
                todoContainer.innerHTML += `
                    <div class="flex items-start p-3 bg-white border border-orange-200 rounded-lg shadow-sm hover:border-orange-400 transition">
                        <button onclick="openCompleteModal(${log.id})" class="mt-0.5 w-5 h-5 rounded border-2 border-orange-400 mr-3 flex-shrink-0 hover:bg-orange-100 transition cursor-pointer flex items-center justify-center text-transparent hover:text-orange-600" title="點擊輸入時數並完成">
                            <i class="fa-solid fa-check text-xs"></i>
                        </button>
                        <div class="flex-grow min-w-0">
                            <p class="text-sm text-gray-800 font-medium truncate">${log.task}</p>
                            <div class="mt-1 flex items-center gap-2">
                                <span class="text-[10px] text-gray-500 bg-gray-100 px-1.5 py-0.5 rounded"><i class="fa-regular fa-calendar mr-1"></i>${log.date.substring(5)}</span>
                                <span class="text-[10px] px-1.5 py-0.5 rounded-full text-white" style="background-color: ${color}">${log.category}</span>
                            </div>
                        </div>
                    </div>
                `;
            });
        }

        // --- 2. 渲染當月行事曆總表 ---
        function renderCalendarGrid() {
            const dateStr = dateInput.value;
            const targetDate = new Date(dateStr);
            const year = targetDate.getFullYear();
            const month = targetDate.getMonth(); // 0-11
            
            document.getElementById('calendar-title').innerText = `${year}年 ${month + 1}月`;

            const firstDayIndex = new Date(year, month, 1).getDay(); // 0=Sun, 1=Mon...
            const daysInMonth = new Date(year, month + 1, 0).getDate();
            
            const grid = document.getElementById('calendar-grid');
            grid.innerHTML = '';

            // 填補月初空白格
            for (let i = 0; i < firstDayIndex; i++) {
                grid.innerHTML += `<div class="bg-gray-50/50 calendar-cell"></div>`;
            }

            // 填入當月日期與工作
            for (let d = 1; d <= daysInMonth; d++) {
                const currentCellDateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
                const isSelected = currentCellDateStr === dateStr;
                const isToday = currentCellDateStr === new Date().toISOString().split('T')[0];
                const isVacation = vacationDates.includes(currentCellDateStr);

                // 抓取該日期的「已完成」(hours > 0)
                const dayLogs = workLogs.filter(log => log.date === currentCellDateStr && log.hours > 0);
                
                let logsHtml = '';
                let totalHours = 0;
                dayLogs.forEach(log => {
                    const color = categoryColors[log.category] || '#9ca3af';
                    logsHtml += `<div class="text-[10px] text-white px-1.5 mt-1 rounded truncate leading-tight py-0.5 shadow-sm" style="background-color: ${color}" title="${log.task} (${log.hours}h)">${log.task}</div>`;
                    totalHours += log.hours;
                });

                const highlightClass = isSelected ? 'bg-blue-50 ring-2 ring-blue-400 ring-inset relative z-10' : (isToday ? 'bg-yellow-50/50 hover:bg-yellow-100' : 'bg-white hover:bg-blue-50/50');
                const textHighlight = isSelected ? 'text-blue-700 font-bold' : 'text-gray-600 font-medium';
                const hourBadge = totalHours > 0 ? `<span class="text-[10px] text-gray-400 ml-1">(${totalHours}h)</span>` : '';
                
                // 休假標籤
                const vacationBadge = isVacation ? `<div class="text-center font-bold text-green-700 my-1 text-[11px] bg-green-100 rounded py-0.5 shadow-sm border border-green-200">🌴 休假中</div>` : '';

                grid.innerHTML += `
                    <div class="calendar-cell p-1.5 flex flex-col ${highlightClass} cursor-pointer transition-colors duration-200" onclick="openDayDetailModal('${currentCellDateStr}')">
                        <div class="text-right text-xs ${textHighlight} mb-1">${d} ${hourBadge}</div>
                        ${vacationBadge}
                        <div class="flex-grow space-y-px overflow-y-auto no-scrollbar max-h-[80px]">
                            ${logsHtml}
                        </div>
                    </div>
                `;
            }

            // 填補月末空白格使其為完整的網格
            const totalCells = firstDayIndex + daysInMonth;
            const remainingCells = (7 - (totalCells % 7)) % 7;
            for (let i = 0; i < remainingCells; i++) {
                grid.innerHTML += `<div class="bg-gray-50/50 calendar-cell"></div>`;
            }
        }

        // --- 3. 更新圖表 ---
        function updateCharts() {
            const targetDateObj = new Date(dateInput.value);
            const currentYear = targetDateObj.getFullYear();
            const currentMonth = targetDateObj.getMonth();
            const currentQuarter = Math.floor(currentMonth / 3);

            document.getElementById('title-month').innerText = `${currentMonth + 1}月佔比`;
            document.getElementById('title-quarter').innerText = `Q${currentQuarter + 1} 佔比`;
            document.getElementById('title-year').innerText = `${currentYear}年佔比`;

            let totals = {
                month: { 'HRD': 0, 'HRM': 0, '人事': 0, '訪談關心': 0, '團隊管理與會議': 0, '獎項與榮譽': 0, '財務': 0, '其他': 0 },
                quarter: { 'HRD': 0, 'HRM': 0, '人事': 0, '訪談關心': 0, '團隊管理與會議': 0, '獎項與榮譽': 0, '財務': 0, '其他': 0 },
                year: { 'HRD': 0, 'HRM': 0, '人事': 0, '訪談關心': 0, '團隊管理與會議': 0, '獎項與榮譽': 0, '財務': 0, '其他': 0 }
            };

            workLogs.forEach(log => {
                const logDate = new Date(log.date);
                const hrs = parseFloat(log.hours) || 0;
                const cat = log.category;
                
                if (hrs <= 0) return; // 圖表只統計已完成工時

                if (logDate.getFullYear() === currentYear) {
                    if(totals.year[cat] !== undefined) totals.year[cat] += hrs;
                    if (Math.floor(logDate.getMonth() / 3) === currentQuarter) {
                        if(totals.quarter[cat] !== undefined) totals.quarter[cat] += hrs;
                    }
                    if (logDate.getMonth() === currentMonth) {
                        if(totals.month[cat] !== undefined) totals.month[cat] += hrs;
                    }
                }
            });

            drawPieChart('month', 'pieMonth', totals.month);
            drawPieChart('quarter', 'pieQuarter', totals.quarter);
            drawPieChart('year', 'pieYear', totals.year);
        }

        // --- 4. 渲染工作佔比總整理表格 ---
        function renderSummaryTable() {
            const targetDateObj = new Date(dateInput.value);
            const currentYear = targetDateObj.getFullYear();
            
            const cats = Object.keys(categoryColors);
            const initCats = () => cats.reduce((acc, cat) => ({...acc, [cat]: 0}), {});

            // 初始化存放 12 個月、4 個季度與全年的資料結構
            let monthData = Array.from({length: 12}, () => ({ total: 0, cats: initCats() }));
            let quarterData = Array.from({length: 4}, () => ({ total: 0, cats: initCats() }));
            let yearData = { total: 0, cats: initCats() };

            workLogs.forEach(log => {
                const date = new Date(log.date);
                if (date.getFullYear() !== currentYear || log.hours <= 0) return;

                const m = date.getMonth(); // 0-11
                const q = Math.floor(m / 3); // 0-3
                const c = log.category;
                const h = parseFloat(log.hours);

                // 累加時數
                monthData[m].total += h;
                if(monthData[m].cats[c] !== undefined) monthData[m].cats[c] += h;

                quarterData[q].total += h;
                if(quarterData[q].cats[c] !== undefined) quarterData[q].cats[c] += h;

                yearData.total += h;
                if(yearData.cats[c] !== undefined) yearData.cats[c] += h;
            });

            // 生成動態表頭
            const thead = document.getElementById('summary-table-head');
            let theadHtml = `<tr><th class="py-3 px-4 border-r w-24">期間 (${currentYear})</th><th class="py-3 px-4 border-r text-center w-24">總工時</th>`;
            cats.forEach(c => {
                const color = categoryColors[c];
                theadHtml += `<th class="py-3 px-2 text-center min-w-[80px]"><span class="inline-block w-2.5 h-2.5 rounded-full mr-1.5 shadow-sm" style="background-color: ${color}"></span>${c}</th>`;
            });
            theadHtml += `</tr>`;
            thead.innerHTML = theadHtml;

            // 生成表格內容
            const tbody = document.getElementById('summary-table-body');
            tbody.innerHTML = '';

            const renderRow = (title, data, customClass = '') => {
                let html = `<tr class="hover:bg-blue-50/50 transition ${customClass}">
                    <td class="py-2.5 px-4 border-r font-medium text-gray-700">${title}</td>
                    <td class="py-2.5 px-4 border-r text-center font-bold text-gray-600">${data.total}h</td>`;
                
                cats.forEach(c => {
                    const val = data.cats[c];
                    const pct = data.total > 0 ? Math.round((val / data.total) * 100) : 0;
                    const txt = pct > 0 ? `<span class="font-bold text-gray-800">${pct}%</span> <span class="text-[10px] text-gray-400">(${val}h)</span>` : '<span class="text-gray-300">-</span>';
                    html += `<td class="py-2.5 px-2 text-center text-sm">${txt}</td>`;
                });
                html += `</tr>`;
                return html;
            };

            // 月份
            monthData.forEach((data, i) => {
                if (data.total > 0) tbody.innerHTML += renderRow(`${i+1} 月`, data);
            });
            if (yearData.total > 0) tbody.innerHTML += `<tr><td colspan="${cats.length + 2}" class="bg-gray-50 h-2 border-y"></td></tr>`;

            // 季度
            quarterData.forEach((data, i) => {
                if (data.total > 0) tbody.innerHTML += renderRow(`Q${i+1} 季度`, data, 'bg-blue-50/20');
            });
            if (yearData.total > 0) tbody.innerHTML += `<tr><td colspan="${cats.length + 2}" class="bg-gray-50 h-2 border-y"></td></tr>`;

            // 全年
            if (yearData.total > 0) {
                tbody.innerHTML += renderRow(`全年總計`, yearData, 'bg-blue-100/50 font-bold');
            } else {
                tbody.innerHTML = `<tr><td colspan="${cats.length + 2}" class="text-center py-10 text-gray-400 italic">該年度目前尚無已完成的工作紀錄</td></tr>`;
            }
        }

        function drawPieChart(chartKey, canvasId, dataObj) {
            const ctx = document.getElementById(canvasId).getContext('2d');
            const labels = Object.keys(dataObj);
            const data = Object.values(dataObj);
            const colors = labels.map(label => categoryColors[label] || '#9ca3af');

            if (charts[chartKey]) charts[chartKey].destroy();

            charts[chartKey] = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{ data: data, backgroundColor: colors, borderWidth: 2, borderColor: '#ffffff' }]
                },
                options: {
                    responsive: true, maintainAspectRatio: false, cutout: '65%',
                    plugins: {
                        legend: { display: false }, 
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.label || '';
                                    let value = context.parsed || 0;
                                    let dataset = context.chart.data.datasets[context.datasetIndex];
                                    let total = dataset.data.reduce((acc, curr) => acc + curr, 0);
                                    let percentage = total > 0 ? Math.round((value / total) * 100) : 0;
                                    return `${label}: ${value} 小時 (${percentage}%)`;
                                }
                            }
                        }
                    }
                }
            });
        }

        // --- 5. 渲染後台紀錄總表 ---
        function renderBackendList() {
            const tbody = document.getElementById('backend-list-body');
            tbody.innerHTML = '';
            
            // 複製並依照日期由新到舊排序
            const sortedLogs = [...workLogs].sort((a, b) => new Date(b.date) - new Date(a.date));

            if (sortedLogs.length === 0) {
                tbody.innerHTML = `<tr><td colspan="4" class="text-center py-6 text-gray-500 italic">目前系統中尚無任何紀錄</td></tr>`;
                return;
            }

            sortedLogs.forEach(log => {
                const color = categoryColors[log.category] || '#9ca3af';
                // 時數為 0 顯示為待辦狀態
                const hoursDisplay = log.hours === 0 ? '<span class="text-orange-500 font-bold bg-orange-50 px-2 py-1 rounded text-xs border border-orange-100">待辦中 (0)</span>' : `<span class="font-bold text-gray-600">${log.hours} <span class="font-normal text-xs text-gray-400">h</span></span>`;
                
                tbody.innerHTML += `
                    <tr class="hover:bg-gray-50 transition border-b border-gray-100 last:border-0">
                        <td class="py-3 px-4 text-gray-700 font-medium">${log.date}</td>
                        <td class="py-3 px-4 text-gray-800">${log.task}</td>
                        <td class="py-3 px-4">
                            <span class="text-[11px] px-2 py-1 rounded text-white shadow-sm" style="background-color: ${color}">${log.category}</span>
                        </td>
                        <td class="py-3 px-4">${hoursDisplay}</td>
                    </tr>
                `;
            });
        }


        // --- 6. 渲染分校管理區塊 ---
        function renderBranchTasks() {
            const trackBody = document.getElementById('track-list-body');
            const helpBody = document.getElementById('help-list-body');
            const escBody = document.getElementById('escalation-list-body');
            
            trackBody.innerHTML = '';
            helpBody.innerHTML = '';
            escBody.innerHTML = '';

            const tasks = branchTasks.filter(t => t.branch === currentBranch);
            
            const trackTasks = tasks.filter(t => t.type === 'track');
            const helpTasks = tasks.filter(t => t.type === 'help');

            // 產生文字紀錄的 HTML (共用函數)
            const buildLogsHtml = (task) => {
                const logsStr = (task.logs && task.logs.length > 0) 
                    ? `<div class="mt-2 space-y-1 max-h-[100px] overflow-y-auto no-scrollbar border-l-2 border-slate-300 pl-2">
                        ${task.logs.map(l => `<div class="text-[11px] text-gray-600 leading-tight"><span class="text-slate-500 font-bold mr-1">${l.date}</span>${l.text}</div>`).join('')}
                       </div>` 
                    : '';
                const addLogBtn = `<button onclick="openLogModal(${task.id})" class="text-[10px] text-gray-400 hover:text-slate-600 mt-1.5 flex items-center transition bg-gray-50 hover:bg-slate-100 px-2 py-1 rounded border border-gray-100 w-fit"><i class="fa-solid fa-pen-to-square mr-1"></i>新增追蹤紀錄</button>`;
                return logsStr + addLogBtn;
            };

            // --- 渲染追蹤事項 (Type: track) ---
            if (trackTasks.length === 0) {
                trackBody.innerHTML = `<tr><td colspan="3" class="text-center py-4 text-gray-400 italic">無追蹤事項</td></tr>`;
            } else {
                trackTasks.forEach(task => {
                    let rowClass = 'border-b hover:bg-blue-50/30 transition';
                    
                    let maxLevel = 0;
                    if (task.reminderDates[3]) maxLevel = 3;
                    else if (task.reminderDates[2]) maxLevel = 2;
                    else if (task.reminderDates[1]) maxLevel = 1;

                    if (task.done) rowClass += ' status-done';
                    else if (maxLevel === 1) rowClass += ' status-1';
                    else if (maxLevel === 2) rowClass += ' status-2';
                    else if (maxLevel === 3) rowClass += ' status-3';

                    const btnBase = "w-7 h-7 rounded-full font-bold transition shadow-sm text-xs focus:outline-none flex-shrink-0";
                    
                    const buildBtn = (lvl, colorClass, hoverClass) => {
                        const dateVal = task.reminderDates[lvl];
                        const btnClass = (dateVal && !task.done) ? colorClass : `bg-gray-200 text-gray-500 ${hoverClass}`;
                        const dateHtml = dateVal ? `<span class="text-[9.5px] text-gray-600 mt-1 font-medium bg-white/80 px-1 rounded-sm">${dateVal.substring(5)}</span>` : `<span class="text-[9.5px] text-transparent mt-1">-</span>`;
                        return `
                            <div class="flex flex-col items-center w-10">
                                <button onclick="openReminderModal(${task.id}, ${lvl})" class="${btnBase} ${btnClass}" ${task.done ? 'disabled' : ''}>${lvl}</button>
                                ${dateHtml}
                            </div>
                        `;
                    };

                    trackBody.innerHTML += `
                        <tr class="${rowClass}">
                            <td class="py-3 px-2 align-top">
                                <div class="font-medium text-gray-800">${task.name}</div>
                                <div class="text-[10px] mt-1 text-white px-1.5 py-0.5 rounded-full inline-block" style="background-color: ${categoryColors[task.category]||'#9ca3af'}">${task.category}</div>
                                ${buildLogsHtml(task)}
                            </td>
                            <td class="py-2 px-1 text-center align-top pt-3">
                                <div class="flex justify-center items-start gap-1">
                                    ${buildBtn(1, 'bg-green-500 text-white', 'hover:bg-green-200')}
                                    ${buildBtn(2, 'bg-orange-500 text-white', 'hover:bg-orange-200')}
                                    ${buildBtn(3, 'bg-red-500 text-white', 'hover:bg-red-200')}
                                </div>
                            </td>
                            <td class="py-3 px-2 text-center align-top">
                                <button onclick="toggleBranchTaskDone(${task.id})" class="px-3 py-1.5 rounded-md text-xs font-bold transition ${task.done ? 'bg-gray-200 text-gray-500' : 'bg-blue-50 text-blue-600 border border-blue-200 hover:bg-blue-100'}">
                                    ${task.done ? '<i class="fa-solid fa-check mr-1"></i>已完成' : '標記完成'}
                                </button>
                            </td>
                        </tr>
                    `;
                });
            }

            // --- 渲染協助事項 (Type: help) ---
            if (helpTasks.length === 0) {
                helpBody.innerHTML = `<tr><td colspan="3" class="text-center py-4 text-gray-400 italic">無協助事項</td></tr>`;
            } else {
                helpTasks.forEach(task => {
                    let rowClass = task.done ? 'border-b status-done' : 'border-b hover:bg-purple-50/30 transition';
                    const deadlineAlert = (!task.done && new Date(task.deadline) < new Date(dateInput.value)) ? 'text-red-600 font-bold' : 'text-gray-600';

                    helpBody.innerHTML += `
                        <tr class="${rowClass}">
                            <td class="py-3 px-2 align-top">
                                <div class="font-medium text-gray-800">${task.name}</div>
                                <div class="text-[10px] mt-1 text-white px-1.5 py-0.5 rounded-full inline-block" style="background-color: ${categoryColors[task.category]||'#9ca3af'}">${task.category}</div>
                                ${buildLogsHtml(task)}
                            </td>
                            <td class="py-3 px-2 text-center text-sm align-top ${deadlineAlert}">
                                <i class="fa-regular fa-calendar mr-1"></i>${task.deadline || '未設定'}
                            </td>
                            <td class="py-3 px-2 text-center align-top">
                                <button onclick="toggleBranchTaskDone(${task.id})" class="px-3 py-1.5 rounded-md text-xs font-bold transition ${task.done ? 'bg-gray-200 text-gray-500' : 'bg-purple-50 text-purple-600 border border-purple-200 hover:bg-purple-100'}">
                                    ${task.done ? '<i class="fa-solid fa-check mr-1"></i>已協助' : '未完成'}
                                </button>
                            </td>
                        </tr>
                    `;
                });
            }

            // --- 渲染處長回報清單 (跨分校, level >= 2) ---
            const escalationTasks = branchTasks.filter(t => t.type === 'track' && !t.done && (t.reminderDates[2] || t.reminderDates[3]));
            if (escalationTasks.length === 0) {
                escBody.innerHTML = `<tr><td colspan="4" class="text-center py-4 text-red-400 text-sm italic">目前無需要回報處長的事項</td></tr>`;
            } else {
                escalationTasks.sort((a, b) => new Date(b.reminderDates[3]||b.reminderDates[2]) - new Date(a.reminderDates[3]||a.reminderDates[2])).forEach(task => {
                    const levelBadge = task.reminderDates[3] ? '<span class="bg-red-600 text-white px-2 py-0.5 rounded text-xs shadow-sm">級別 3 (緊急)</span>' : '<span class="bg-orange-500 text-white px-2 py-0.5 rounded text-xs shadow-sm">級別 2 (注意)</span>';
                    const escalateDate = task.reminderDates[3] || task.reminderDates[2];
                    
                    // 這裡加上 buildLogsHtml(task) 讓處長也能看到追蹤進度
                    escBody.innerHTML += `
                        <tr class="border-b border-red-100 hover:bg-red-50">
                            <td class="py-3 px-3 text-red-700 font-medium align-top">${escalateDate || '-'}</td>
                            <td class="py-3 px-3 align-top"><span class="bg-white border border-gray-200 text-gray-600 px-2 py-0.5 rounded text-xs shadow-sm">${task.branch}</span></td>
                            <td class="py-3 px-3 font-medium text-gray-800 align-top">
                                ${task.name}
                                ${buildLogsHtml(task)}
                            </td>
                            <td class="py-3 px-3 text-center align-top">${levelBadge}</td>
                        </tr>
                    `;
                });
            }
        }

        // ==========================================
        //  模組化功能：日曆詳細資訊與編輯 (Day Detail)
        // ==========================================
        
        let currentEditingDate = null;
        let editingLogs = []; // 暫存修改的資料陣列

        function openDayDetailModal(dateStr) {
            currentEditingDate = dateStr;
            document.getElementById('day-detail-title').innerHTML = `<i class="fa-regular fa-calendar-check mr-2"></i>${dateStr} 工作詳情與編輯`;
            
            // 處理休假按鈕狀態
            const isVacation = vacationDates.includes(dateStr);
            const vacBtn = document.getElementById('btn-toggle-vacation');
            if (isVacation) {
                vacBtn.className = "px-4 py-2 rounded-lg text-sm font-bold transition shadow-sm border bg-gray-100 text-gray-700 hover:bg-gray-200 border-gray-300";
                vacBtn.innerHTML = "<i class='fa-solid fa-briefcase mr-1'></i> 取消休假 (恢復上班)";
            } else {
                vacBtn.className = "px-4 py-2 rounded-lg text-sm font-bold transition shadow-sm border bg-green-50 text-green-700 hover:bg-green-100 border-green-200";
                vacBtn.innerHTML = "<i class='fa-solid fa-umbrella-beach mr-1'></i> 標記為休假";
            }

            // 抓出這一天有時數的工作，並做 Deep Copy 以防未儲存就蓋掉原始資料
            editingLogs = workLogs.filter(log => log.date === dateStr && log.hours > 0).map(log => ({...log}));
            
            renderEditingLogs();
            document.getElementById('modal-day-detail').classList.remove('hidden');
        }

        function closeDayDetailModal() {
            document.getElementById('modal-day-detail').classList.add('hidden');
        }

        function renderEditingLogs() {
            const container = document.getElementById('day-detail-logs');
            container.innerHTML = '';

            if (editingLogs.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-8 bg-white border border-gray-200 rounded-lg">
                        <i class="fa-regular fa-folder-open text-gray-300 text-4xl mb-2"></i>
                        <p class="text-sm text-gray-500">這天目前沒有工作紀錄</p>
                    </div>`;
                return;
            }

            editingLogs.forEach((log, index) => {
                const color = categoryColors[log.category] || '#9ca3af';
                container.innerHTML += `
                    <div class="bg-white border border-gray-200 p-4 rounded-lg flex flex-col md:flex-row gap-4 items-center shadow-sm relative">
                        <div class="absolute top-0 left-0 w-1.5 h-full rounded-l-lg" style="background-color: ${color}"></div>
                        
                        <div class="flex-grow w-full pl-2">
                            <label class="block text-xs font-bold text-gray-500 mb-1">工作名稱</label>
                            <input type="text" value="${log.task}" onchange="updateEditingLog(${index}, 'task', this.value)" class="w-full border border-gray-300 rounded-md p-2 text-sm focus:ring-2 focus:ring-blue-500 outline-none">
                        </div>
                        
                        <div class="w-full md:w-36">
                            <label class="block text-xs font-bold text-gray-500 mb-1">所屬日期 (可移動)</label>
                            <input type="date" value="${log.date}" onchange="updateEditingLog(${index}, 'date', this.value)" class="w-full border border-gray-300 rounded-md p-2 text-sm focus:ring-2 focus:ring-blue-500 outline-none">
                        </div>
                        
                        <div class="w-full md:w-24">
                            <label class="block text-xs font-bold text-gray-500 mb-1">耗費時數</label>
                            <input type="number" step="0.5" min="0" value="${log.hours}" onchange="updateEditingLog(${index}, 'hours', this.value)" class="w-full border border-gray-300 rounded-md p-2 text-sm focus:ring-2 focus:ring-blue-500 outline-none">
                        </div>
                        
                        <div class="w-full md:w-auto flex justify-end mt-2 md:mt-0 pt-4 md:pt-5">
                            <button onclick="revertToTodoInsideModal(${index})" class="text-xs font-bold text-orange-600 hover:text-orange-800 bg-orange-50 hover:bg-orange-100 border border-orange-200 px-3 py-2 rounded-md transition whitespace-nowrap" title="時數將歸零並移回待辦清單">
                                <i class="fa-solid fa-rotate-left mr-1"></i>退回待辦
                            </button>
                        </div>
                    </div>
                `;
            });
        }

        // 修改編輯暫存資料
        function updateEditingLog(index, field, value) {
            if (field === 'hours') {
                editingLogs[index][field] = parseFloat(value) || 0;
            } else {
                editingLogs[index][field] = value;
            }
        }

        // 在編輯視窗內將工作退回待辦
        function revertToTodoInsideModal(index) {
            if (confirm(`確定要將「${editingLogs[index].task}」退回未完成待辦嗎？`)) {
                editingLogs[index].hours = 0; // 時數設為0就會被系統判定為待辦
                alert('已標記為待辦，請點擊右下角「儲存所有變更」生效。');
                renderEditingLogs(); // 重新渲染輸入框數值
            }
        }

        // 切換休假狀態
        function toggleVacation() {
            const idx = vacationDates.indexOf(currentEditingDate);
            if (idx > -1) {
                vacationDates.splice(idx, 1); // 移除休假
            } else {
                vacationDates.push(currentEditingDate); // 加入休假
            }
            openDayDetailModal(currentEditingDate); // 刷新 Modal 內的按鈕
            updateDashboard(); // 同步刷新後方行事曆 UI
        }

        // 儲存修改內容覆蓋至原始資料庫
        function saveDayDetails() {
            editingLogs.forEach(editedLog => {
                const originalLogIndex = workLogs.findIndex(l => l.id === editedLog.id);
                if (originalLogIndex > -1) {
                    workLogs[originalLogIndex] = {...editedLog}; // 更新回原本陣列
                }
            });
            closeDayDetailModal();
            updateDashboard(); // 重新整理所有畫面
        }



        // --- 互動事件綁定 ---

        // 主表單提交 (新增工作紀錄/待辦)
        document.getElementById('work-form').addEventListener('submit', function(e) {
            e.preventDefault();
            const date = dateInput.value;
            const task = document.getElementById('input-task').value;
            const category = document.getElementById('input-category').value;
            const hours = parseFloat(document.getElementById('input-hours').value) || 0;

            workLogs.push({ id: Date.now(), date, task, category, hours });

            document.getElementById('input-task').value = '';
            document.getElementById('input-hours').value = '0'; // 預設歸零為待辦
            updateDashboard();
        });

        // 頂部日期切換時重繪行事曆與圖表
        dateInput.addEventListener('change', updateDashboard);

        // --- 行事曆月份與年份切換 ---
        function changeMonth(offset) {
            const currentDateStr = dateInput.value;
            if (!currentDateStr) return;
            
            let [year, month, day] = currentDateStr.split('-').map(Number);
            month += offset;
            
            if (month > 12) { month = 1; year++; }
            if (month < 1) { month = 12; year--; }
            
            // 防呆處理：如切換月份導致日期超出該月天數（例如 5/31 切到 6月，會卡在 6/30）
            const maxDays = new Date(year, month, 0).getDate();
            if (day > maxDays) day = maxDays;
            
            dateInput.value = `${year}-${String(month).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            updateDashboard();
        }

        function changeYearPrompt() {
            const currentDateStr = dateInput.value;
            if (!currentDateStr) return;
            
            let [year, month, day] = currentDateStr.split('-').map(Number);
            const newYearStr = prompt("請輸入欲前往的年份 (例如: 2026)：", year);
            
            if (newYearStr && !isNaN(newYearStr) && newYearStr.length === 4) {
                year = parseInt(newYearStr);
                
                const maxDays = new Date(year, month, 0).getDate();
                if (day > maxDays) day = maxDays;
                
                dateInput.value = `${year}-${String(month).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
                updateDashboard();
            }
        }

        // 分校頁籤切換
        function switchTab(branchName) {
            currentBranch = branchName;
            document.querySelectorAll('#branch-tabs button').forEach(btn => {
                btn.className = btn.dataset.branch === branchName ? 'pb-2 px-1 whitespace-nowrap tab-active' : 'pb-2 px-1 whitespace-nowrap tab-inactive';
            });
            renderBranchTasks();
        }

        // --- 待辦轉換為已完成 (Modal) ---
        function openCompleteModal(logId) {
            const log = workLogs.find(l => l.id === logId);
            if (!log) return;
            document.getElementById('complete-task-id').value = logId;
            document.getElementById('complete-task-name').innerText = log.task;
            document.getElementById('complete-hours-input').value = 1; // 預設 1 小時
            document.getElementById('modal-complete').classList.remove('hidden');
        }
        
        function closeCompleteModal() {
            document.getElementById('modal-complete').classList.add('hidden');
        }

        function confirmCompleteTodo() {
            const logId = parseInt(document.getElementById('complete-task-id').value);
            const hrs = parseFloat(document.getElementById('complete-hours-input').value);
            const log = workLogs.find(l => l.id === logId);
            
            if (log && hrs > 0) {
                log.hours = hrs;
                log.date = dateInput.value; // 將日期設為當前選擇的操作日期
                closeCompleteModal();
                updateDashboard(); // 會自動從 Todo 消失並移入 Calendar
            } else {
                alert('請輸入大於 0 的時數');
            }
        }

        // --- 分校待辦狀態切換 ---
        function toggleBranchTaskDone(taskId) {
            const task = branchTasks.find(t => t.id === taskId);
            if (task) {
                task.done = !task.done;
                renderBranchTasks();
            }
        }

        // --- 新增分校事項 (Modal) ---
        function openAddTaskModal(type) {
            document.getElementById('modal-branch').value = currentBranch;
            document.getElementById('modal-task-name').value = '';
            document.getElementById('modal-task-type').value = type;
            
            const isTrack = type === 'track';
            document.getElementById('modal-add-title').innerText = isTrack ? '新增追蹤事項 (紅綠燈)' : '新增協助事項 (押期限)';
            document.getElementById('modal-header-bg').className = isTrack ? 'bg-blue-600 px-4 py-3 flex justify-between items-center' : 'bg-purple-600 px-4 py-3 flex justify-between items-center';
            document.getElementById('modal-submit-btn').className = isTrack ? 'w-full bg-blue-600 hover:bg-blue-700 text-white font-medium py-2.5 rounded-lg mt-2 transition shadow-sm' : 'w-full bg-purple-600 hover:bg-purple-700 text-white font-medium py-2.5 rounded-lg mt-2 transition shadow-sm';
            
            const dlGroup = document.getElementById('modal-deadline-group');
            if(isTrack) {
                dlGroup.classList.add('hidden');
            } else {
                dlGroup.classList.remove('hidden');
                document.getElementById('modal-deadline').value = dateInput.value;
            }

            document.getElementById('modal-add-task').classList.remove('hidden');
        }

        function closeAddTaskModal() {
            document.getElementById('modal-add-task').classList.add('hidden');
        }

        function confirmAddTask() {
            const branch = document.getElementById('modal-branch').value;
            const name = document.getElementById('modal-task-name').value.trim();
            const category = document.getElementById('modal-category').value;
            const type = document.getElementById('modal-task-type').value;
            const deadline = document.getElementById('modal-deadline').value;
            
            if(!name) { alert('請輸入工作名稱'); return; }

            branchTasks.push({
                id: Date.now(), branch, name, category, type, done: false,
                reminderDates: {1:'', 2:'', 3:''},
                logs: [], // 初始化紀錄陣列
                escalationNote: '', // 新增處長回報說明欄位
                deadline: type === 'help' ? deadline : null
            });
            
            closeAddTaskModal();
            if(branch === currentBranch) renderBranchTasks(); 
        }

        // --- 處長追蹤清單說明更新 ---
        function updateEscalationNote(taskId, note) {
            const task = branchTasks.find(t => t.id === taskId);
            if (task) {
                task.escalationNote = note;
            }
        }

        // --- 追蹤紀錄 (文字備註) Modal ---
        function openLogModal(taskId) {
            const task = branchTasks.find(t => t.id === taskId);
            if (!task) return;
            document.getElementById('log-task-id').value = taskId;
            document.getElementById('log-task-name').innerText = task.name;
            document.getElementById('log-text-input').value = '';
            document.getElementById('modal-branch-log').classList.remove('hidden');
            setTimeout(() => document.getElementById('log-text-input').focus(), 100);
        }

        function closeLogModal() {
            document.getElementById('modal-branch-log').classList.add('hidden');
        }

        function confirmAddLog() {
            const taskId = parseInt(document.getElementById('log-task-id').value);
            const text = document.getElementById('log-text-input').value.trim();
            if (!text) {
                closeLogModal();
                return;
            }

            const task = branchTasks.find(t => t.id === taskId);
            if (task) {
                if (!task.logs) task.logs = [];
                // 產生目前的時間標籤 (月/日 時:分)
                const now = new Date();
                const timeStr = `${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;
                
                task.logs.push({ date: timeStr, text: text });
            }
            
            closeLogModal();
            renderBranchTasks();
        }

        // --- 提醒日期設定 (Modal) ---
        function openReminderModal(taskId, level) {
            const task = branchTasks.find(t => t.id === taskId);
            if(!task) return;

            document.getElementById('reminder-task-id').value = taskId;
            document.getElementById('reminder-level').value = level;
            document.getElementById('reminder-date-input').value = task.reminderDates[level] || dateInput.value;
            document.getElementById('reminder-modal-desc').innerHTML = `將 <strong>${task.name}</strong> 設為提醒等級 <span class="font-bold text-orange-600">${level}</span>，請記錄提醒日期：`;
            
            document.getElementById('modal-reminder').classList.remove('hidden');
        }

        function closeReminderModal() { document.getElementById('modal-reminder').classList.add('hidden'); }
        
        function clearReminderDate() {
            const taskId = parseInt(document.getElementById('reminder-task-id').value);
            const level = parseInt(document.getElementById('reminder-level').value);
            const task = branchTasks.find(t => t.id === taskId);
            if(task) task.reminderDates[level] = '';
            closeReminderModal();
            renderBranchTasks();
        }

        function confirmReminderDate() {
            const taskId = parseInt(document.getElementById('reminder-task-id').value);
            const level = parseInt(document.getElementById('reminder-level').value);
            const task = branchTasks.find(t => t.id === taskId);
            if(task) task.reminderDates[level] = document.getElementById('reminder-date-input').value;
            closeReminderModal();
            renderBranchTasks();
        }

        // 初始執行
        window.onload = function() {
            renderLegend();
            updateDashboard();
            renderBranchTasks();
        };

    </script>
</body>
</html>
