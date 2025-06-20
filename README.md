<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>市場情資與競品分析平台</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://apis.google.com/js/api.js"></script>
    <script src="https://accounts.google.com/gsi/client"></script>
    <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: {
                            50: '#f0f9ff',
                            100: '#e0f2fe',
                            200: '#bae6fd',
                            300: '#7dd3fc',
                            400: '#38bdf8',
                            500: '#0ea5e9',
                            600: '#0284c7',
                            700: '#0369a1',
                            800: '#075985',
                            900: '#0c4a6e',
                        }
                    },
                    fontFamily: {
                        sans: ['"Noto Sans TC"', 'sans-serif']
                    }
                }
            }
        }
    </script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Noto Sans TC', sans-serif; }
        .tab.active { border-color: #0ea5e9; background-color: #f0f9ff; color: #0369a1; }
        .panel { display: none; }
        .panel.active { display: block; animation: fadeIn 0.5s ease; }
        .locked { opacity: 0.5; pointer-events: none; }
        .spinner { border: 4px solid rgba(0, 0, 0, 0.1); width: 36px; height: 36px; border-radius: 50%; border-left-color: #0ea5e9; animation: spin 1s ease infinite; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes spin { to { transform: rotate(360deg); } }
        .competitor-card:hover { transform: translateY(-5px); }
        .tooltip { position: relative; display: inline-block; }
        .tooltip .tooltiptext { visibility: hidden; width: 200px; background-color: #555; color: #fff; text-align: center; border-radius: 6px; padding: 5px; position: absolute; z-index: 1; bottom: 125%; left: 50%; margin-left: -100px; opacity: 0; transition: opacity 0.3s; }
        .tooltip:hover .tooltiptext { visibility: visible; opacity: 1; }
        .keyword-tag { transition: all 0.2s; }
        .keyword-tag:hover { transform: translateY(-2px); }
        .industry-badge { font-size: 0.7rem; padding: 0.15rem 0.5rem; border-radius: 9999px; }
        .chart-container { height: 300px; }
        .gradient-card { background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%); }
        .shadow-hover:hover { box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05); }
        .table-header { position: sticky; top: 0; background-color: #f9fafb; z-index: 10; }
        .table-row:nth-child(even) { background-color: #f9fafb; }
        .table-row:hover { background-color: #f0f9ff; }
    </style>
</head>
<body class="bg-gray-50">

    <header class="bg-white shadow-md">
        <div class="container mx-auto px-6 py-4 flex justify-between items-center">
            <div class="flex items-center">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-primary-600 mr-3" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M5.3 8.6a1 1 0 011.4-1.4l4.3 4.3 4.3-4.3a1 1 0 011.4 1.4l-5 5a1 1 0 01-1.4 0l-5-5z" clip-rule="evenodd" />
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm0-2a6 6 0 100-12 6 6 0 000 12z" clip-rule="evenodd" />
                </svg>
                <h1 class="text-2xl font-bold text-gray-800">市場情資與競品分析平台</h1>
            </div>
            <div id="auth-container">
                <button id="authorize_button" class="bg-primary-600 text-white font-semibold px-4 py-2 rounded-lg hover:bg-primary-700 transition shadow-md">使用 Google 帳戶登入</button>
                <div id="user-profile" class="hidden items-center gap-3">
                    <img id="user-avatar" class="w-10 h-10 rounded-full border-2 border-primary-300" src="" alt="User Avatar">
                    <span id="user-name" class="font-semibold text-gray-700"></span>
                    <button id="signout_button" class="text-sm text-gray-500 hover:text-gray-800 border border-gray-300 rounded-md px-3 py-1 hover:bg-gray-100 transition">登出</button>
                </div>
            </div>
        </div>
    </header>
    
    <div id="alert-container" class="sticky top-0 z-50 p-4"></div>

    <main class="container mx-auto px-6 py-8">
        <div id="app-container" class="locked">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                
                <div class="lg:col-span-1 space-y-6">
                    <section id="project-section" class="bg-white p-6 rounded-xl shadow-lg border border-gray-100">
                        <h2 class="text-xl font-bold mb-4 text-gray-800 flex items-center">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-primary-500" viewBox="0 0 20 20" fill="currentColor">
                                <path d="M2 6a2 2 0 012-2h5l2 2h5a2 2 0 012 2v6a2 2 0 01-2 2H4a2 2 0 01-2-2V6z" />
                            </svg>
                            專案管理
                        </h2>
                        <div class="space-y-4">
                            <div>
                                <label for="project-name" class="block text-sm font-medium text-gray-700">專案名稱</label>
                                <input type="text" id="project-name" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="請輸入專案名稱">
                            </div>
                            <div>
                                <label for="client-name" class="block text-sm font-medium text-gray-700">客戶名稱</label>
                                <input type="text" id="client-name" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="請輸入客戶名稱">
                            </div>
                            <div>
                                <label for="drive-folder" class="block text-sm font-medium text-gray-700">Google Drive 資料夾名稱</label>
                                <input type="text" id="drive-folder" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="例如：Project_A_Analysis">
                            </div>
                            <div class="flex gap-2">
                                <button id="save-project" class="flex-1 bg-primary-600 text-white font-semibold py-2 rounded-md hover:bg-primary-700 transition shadow-sm">儲存專案設定</button>
                                <button id="load-project" class="flex-1 bg-gray-600 text-white font-semibold py-2 rounded-md hover:bg-gray-700 transition shadow-sm">載入專案</button>
                            </div>
                        </div>
                        
                        <div class="mt-6">
                            <h3 class="font-semibold text-gray-700 mb-3 flex items-center">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1 text-primary-500" viewBox="0 0 20 20" fill="currentColor">
                                    <path d="M9 2a1 1 0 000 2h2a1 1 0 100-2H9z" />
                                    <path fill-rule="evenodd" d="M4 5a2 2 0 012-2 3 3 0 003 3h2a3 3 0 003-3 2 2 0 012 2v11a2 2 0 01-2 2H6a2 2 0 01-2-2V5zm3 4a1 1 0 000 2h.01a1 1 0 100-2H7zm3 0a1 1 0 000 2h3a1 1 0 100-2h-3zm-3 4a1 1 0 100 2h.01a1 1 0 100-2H7zm3 0a1 1 0 100 2h3a1 1 0 100-2h-3z" clip-rule="evenodd" />
                                </svg>
                                歷史紀錄
                            </h3>
                            <div id="history-list" class="space-y-3 max-h-60 overflow-y-auto pr-1">
                                <div class="text-gray-500 text-center py-4">尚無歷史紀錄</div>
                            </div>
                        </div>
                    </section>

                    <section id="settings-section" class="bg-white p-6 rounded-xl shadow-lg border border-gray-100">
                        <h2 class="text-xl font-bold mb-4 text-gray-800 flex items-center">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-primary-500" viewBox="0 0 20 20" fill="currentColor">
                                <path fill-rule="evenodd" d="M11.49 3.17c-.38-1.56-2.6-1.56-2.98 0a1.532 1.532 0 01-2.286.948c-1.372-.836-2.942.734-2.106 2.106.54.886.061 2.042-.947 2.287-1.561.379-1.561 2.6 0 2.978a1.532 1.532 0 01.947 2.287c-.836 1.372.734 2.942 2.106 2.106a1.532 1.532 0 012.287.947c.379 1.561 2.6 1.561 2.978 0a1.533 1.533 0 012.287-.947c1.372.836 2.942-.734 2.106-2.106a1.533 1.533 0 01.947-2.287c1.561-.379 1.561-2.6 0-2.978a1.532 1.532 0 01-.947-2.287c.836-1.372-.734-2.942-2.106-2.106a1.532 1.532 0 01-2.287-.947zM10 13a3 3 0 100-6 3 3 0 000 6z" clip-rule="evenodd" />
                            </svg>
                            API & 應用程式設定
                        </h2>
                        <div class="space-y-4">
                            <div>
                                <label for="google-api-key" class="block text-sm font-medium text-gray-700">Google API 金鑰</label>
                                <input type="password" id="google-api-key" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="請輸入您的 Google API 金鑰">
                            </div>
                            <div>
                                <label for="search-engine-id" class="block text-sm font-medium text-gray-700">可程式化搜尋引擎 ID (CX)</label>
                                <input type="text" id="search-engine-id" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="請輸入您的 CX ID">
                            </div>
                            <div>
                                <label for="ai-provider" class="block text-sm font-medium text-gray-700">AI 模型提供商</label>
                                <select id="ai-provider" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                    <option value="gemini">Google Gemini</option>
                                    <option value="monica" disabled>Monica (即將支援)</option>
                                </select>
                            </div>
                            <div>
                                <label for="ai-api-key" class="block text-sm font-medium text-gray-700">AI API 金鑰</label>
                                <input type="password" id="ai-api-key" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="請輸入您的 AI API 金鑰">
                            </div>
                            <button id="save-settings" class="w-full bg-primary-600 text-white font-semibold py-2 rounded-md hover:bg-primary-700 transition shadow-sm">儲存設定</button>
                        </div>
                    </section>

                    <section id="industry-section" class="bg-white p-6 rounded-xl shadow-lg border border-gray-100">
                        <h2 class="text-xl font-bold mb-4 text-gray-800 flex items-center">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-primary-500" viewBox="0 0 20 20" fill="currentColor">
                                <path fill-rule="evenodd" d="M4 4a2 2 0 012-2h4.586A2 2 0 0112 2.586L15.414 6A2 2 0 0116 7.414V16a2 2 0 01-2 2H6a2 2 0 01-2-2V4zm2 6a1 1 0 011-1h6a1 1 0 110 2H7a1 1 0 01-1-1zm1 3a1 1 0 100 2h6a1 1 0 100-2H7z" clip-rule="evenodd" />
                            </svg>
                            產業分類管理
                        </h2>
                        <div class="space-y-4">
                            <div>
                                <label for="industry-name" class="block text-sm font-medium text-gray-700">產業名稱</label>
                                <input type="text" id="industry-name" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="輸入產業名稱">
                            </div>
                            <div>
                                <label for="industry-code" class="block text-sm font-medium text-gray-700">產業代碼</label>
                                <input type="text" id="industry-code" class="mt-1 w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="例如：TECH, FIN, EDU">
                            </div>
                            <div>
                                <label for="industry-color" class="block text-sm font-medium text-gray-700">顏色標記</label>
                                <input type="color" id="industry-color" class="mt-1 w-full p-1 h-10 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" value="#0ea5e9">
                            </div>
                            <button id="add-industry" class="w-full bg-primary-600 text-white font-semibold py-2 rounded-md hover:bg-primary-700 transition shadow-sm">新增產業分類</button>
                        </div>
                        
                        <div class="mt-6">
                            <h3 class="font-semibold text-gray-700 mb-3 flex items-center">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1 text-primary-500" viewBox="0 0 20 20" fill="currentColor">
                                    <path fill-rule="evenodd" d="M3 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clip-rule="evenodd" />
                                </svg>
                                已建立的產業分類
                            </h3>
                            <div id="industries-list" class="space-y-3 max-h-60 overflow-y-auto pr-1">
                                <!-- 產業分類列表將在此動態生成 -->
                            </div>
                        </div>
                    </section>
                </div>
                
                <div class="lg:col-span-2 space-y-6">
                    <section class="bg-white p-6 rounded-xl shadow-lg border border-gray-100">
                        <div class="flex mb-6 border-b">
                            <button class="tab active px-4 py-2 border-b-2 font-medium text-gray-700 flex items-center" data-tab="keywords-analysis">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                    <path fill-rule="evenodd" d="M3 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clip-rule="evenodd" />
                                </svg>
                                行業關鍵字分析
                            </button>
                            <button class="tab px-4 py-2 border-b-2 font-medium text-gray-700 flex items-center" data-tab="competitor-websites">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                    <path fill-rule="evenodd" d="M6.672 1.911a1 1 0 10-1.932.518l.259.966a1 1 0 001.932-.518l-.26-.966zM2.429 4.74a1 1 0 10-.517 1.932l.966.259a1 1 0 00.517-1.932l-.966-.26zm8.814-.569a1 1 0 00-1.415-1.414l-.707.707a1 1 0 101.415 1.415l.707-.708zm-7.071 7.072l.707-.707A1 1 0 003.465 9.12l-.708.707a1 1 0 001.415 1.415zm3.2-5.171a1 1 0 00-1.3 1.3l4 10a1 1 0 001.823.075l1.38-2.759 3.018 3.02a1 1 0 001.414-1.415l-3.019-3.02 2.76-1.379a1 1 0 00-.076-1.822l-10-4z" clip-rule="evenodd" />
                                </svg>
                                競爭對手網站
                            </button>
                            <button class="tab px-4 py-2 border-b-2 font-medium text-gray-700 flex items-center" data-tab="linkedin-analysis">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                    <path d="M13 6a3 3 0 11-6 0 3 3 0 016 0zM18 8a2 2 0 11-4 0 2 2 0 014 0zM14 15a4 4 0 00-8 0v3h8v-3zM6 8a2 2 0 11-4 0 2 2 0 014 0zM16 18v-3a5.972 5.972 0 00-.75-2.906A3.005 3.005 0 0119 15v3h-3zM4.75 12.094A5.973 5.973 0 004 15v3H1v-3a3 3 0 013.75-2.906z" />
                                </svg>
                                LinkedIn 公司分析
                            </button>
                        </div>
                        
                        <div id="keywords-analysis" class="panel active">
                            <div class="flex justify-between items-center mb-4">
                                <h3 class="text-lg font-semibold text-gray-800">關鍵字管理與分析</h3>
                                <div class="flex gap-2">
                                    <button id="save-analysis" class="bg-primary-600 text-white px-4 py-2 rounded-md hover:bg-primary-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path d="M7.707 10.293a1 1 0 10-1.414 1.414l3 3a1 1 0 001.414 0l3-3a1 1 0 00-1.414-1.414L11 11.586V6h5a2 2 0 012 2v7a2 2 0 01-2 2H4a2 2 0 01-2-2V8a2 2 0 012-2h5v5.586l-1.293-1.293zM9 4a1 1 0 012 0v2H9V4z" />
                                        </svg>
                                        儲存本次分析
                                    </button>
                                    <button id="export-excel" class="bg-green-600 text-white px-4 py-2 rounded-md hover:bg-green-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM6.293 6.707a1 1 0 010-1.414l3-3a1 1 0 011.414 0l3 3a1 1 0 01-1.414 1.414L11 5.414V13a1 1 0 11-2 0V5.414L7.707 6.707a1 1 0 01-1.414 0z" clip-rule="evenodd" />
                                        </svg>
                                        匯出為 Excel
                                    </button>
                                </div>
                            </div>
                            
                            <div class="mb-6">
                                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                                    <div>
                                        <label for="keyword-input" class="block text-sm font-medium text-gray-700 mb-1">關鍵字</label>
                                        <input type="text" id="keyword-input" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="輸入關鍵字">
                                    </div>
                                    <div>
                                        <label for="keyword-language" class="block text-sm font-medium text-gray-700 mb-1">語言</label>
                                        <select id="keyword-language" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                            <option value="zh-TW">繁體中文</option>
                                            <option value="zh-CN">簡體中文</option>
                                            <option value="en">英文</option>
                                            <option value="ja">日文</option>
                                            <option value="ko">韓文</option>
                                        </select>
                                    </div>
                                    <div>
                                        <label for="keyword-type" class="block text-sm font-medium text-gray-700 mb-1">關鍵字類型</label>
                                        <select id="keyword-type" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                            <option value="brand">品牌字</option>
                                            <option value="product">產品字</option>
                                            <option value="question">問題字</option>
                                            <option value="general">一般字</option>
                                            <option value="location">地點字</option>
                                        </select>
                                    </div>
                                </div>
                                
                                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                                    <div>
                                        <label for="search-volume" class="block text-sm font-medium text-gray-700 mb-1">預估搜尋量級</label>
                                        <select id="search-volume" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                            <option value="high">高</option>
                                            <option value="medium">中</option>
                                            <option value="low">低</option>
                                        </select>
                                    </div>
                                    <div>
                                        <label for="keyword-industry" class="block text-sm font-medium text-gray-700 mb-1">相關產業</label>
                                        <select id="keyword-industry" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                            <option value="">請選擇產業</option>
                                            <!-- 產業選項將在此動態生成 -->
                                        </select>
                                    </div>
                                    <div class="flex items-end">
                                        <button id="add-keyword" class="w-full bg-primary-600 text-white font-semibold py-2 rounded-md hover:bg-primary-700 transition shadow-sm flex items-center justify-center">
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                                <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
                                            </svg>
                                            新增關鍵字
                                        </button>
                                    </div>
                                </div>
                                
                                <div class="bg-gray-50 p-4 rounded-lg">
                                    <div class="flex justify-between items-center mb-3">
                                        <h4 class="font-medium text-gray-700">關鍵字列表</h4>
                                        <div class="flex gap-2">
                                            <select id="keyword-filter" class="text-sm p-1 border border-gray-300 rounded-md">
                                                <option value="all">全部類型</option>
                                                <option value="brand">品牌字</option>
                                                <option value="product">產品字</option>
                                                <option value="question">問題字</option>
                                                <option value="general">一般字</option>
                                                <option value="location">地點字</option>
                                            </select>
                                            <button id="bulk-import" class="text-sm bg-gray-600 text-white px-3 py-1 rounded-md hover:bg-gray-700 transition">批量匯入</button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            
                            <div class="overflow-x-auto bg-white rounded-lg border border-gray-200 shadow-sm">
                                <table class="min-w-full divide-y divide-gray-200">
                                    <thead class="bg-gray-50">
                                        <tr class="table-header">
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="keyword">
                                                關鍵字
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="language">
                                                語言
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="volume">
                                                搜尋量級
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="type">
                                                關鍵字類型
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="industry">
                                                相關產業
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">操作</th>
                                        </tr>
                                    </thead>
                                    <tbody id="keywords-table" class="bg-white divide-y divide-gray-200">
                                        <!-- 關鍵字表格內容將在此動態生成 -->
                                    </tbody>
                                </table>
                            </div>
                        </div>
                        
                        <div id="competitor-websites" class="panel">
                            <div class="flex justify-between items-center mb-4">
                                <h3 class="text-lg font-semibold text-gray-800">競爭對手網站管理</h3>
                                <div class="flex gap-2">
                                    <button id="save-competitors" class="bg-primary-600 text-white px-4 py-2 rounded-md hover:bg-primary-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path d="M7.707 10.293a1 1 0 10-1.414 1.414l3 3a1 1 0 001.414 0l3-3a1 1 0 00-1.414-1.414L11 11.586V6h5a2 2 0 012 2v7a2 2 0 01-2 2H4a2 2 0 01-2-2V8a2 2 0 012-2h5v5.586l-1.293-1.293zM9 4a1 1 0 012 0v2H9V4z" />
                                        </svg>
                                        儲存本次分析
                                    </button>
                                    <button id="export-competitors-excel" class="bg-green-600 text-white px-4 py-2 rounded-md hover:bg-green-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM6.293 6.707a1 1 0 010-1.414l3-3a1 1 0 011.414 0l3 3a1 1 0 01-1.414 1.414L11 5.414V13a1 1 0 11-2 0V5.414L7.707 6.707a1 1 0 01-1.414 0z" clip-rule="evenodd" />
                                        </svg>
                                        匯出為 Excel
                                    </button>
                                </div>
                            </div>
                            
                            <div class="mb-6">
                                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                                    <div>
                                        <label for="competitor-name" class="block text-sm font-medium text-gray-700 mb-1">競爭對手品牌名</label>
                                        <input type="text" id="competitor-name" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="輸入品牌名稱">
                                    </div>
                                    <div>
                                        <label for="competitor-url" class="block text-sm font-medium text-gray-700 mb-1">主要網站連結</label>
                                        <input type="url" id="competitor-url" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="https://example.com">
                                    </div>
                                    <div>
                                        <label for="competitor-industry" class="block text-sm font-medium text-gray-700 mb-1">產業類別</label>
                                        <select id="competitor-industry" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                            <option value="">請選擇產業</option>
                                            <!-- 產業選項將在此動態生成 -->
                                        </select>
                                    </div>
                                </div>
                                
                                <div class="mb-4">
                                    <label for="competitor-summary" class="block text-sm font-medium text-gray-700 mb-1">網站摘要</label>
                                    <textarea id="competitor-summary" rows="3" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="簡短描述競爭對手網站的特點和內容"></textarea>
                                </div>
                                
                                <div class="flex justify-end">
                                    <button id="add-competitor" class="bg-primary-600 text-white font-semibold px-4 py-2 rounded-md hover:bg-primary-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
                                        </svg>
                                        新增競爭對手
                                    </button>
                                </div>
                            </div>
                            
                            <div class="space-y-4" id="competitors-list">
                                <!-- 競爭對手列表將在此動態生成 -->
                            </div>
                        </div>
                        
                        <div id="linkedin-analysis" class="panel">
                            <div class="flex justify-between items-center mb-4">
                                <h3 class="text-lg font-semibold text-gray-800">LinkedIn 公司分析</h3>
                                <div class="flex gap-2">
                                    <button id="save-linkedin" class="bg-primary-600 text-white px-4 py-2 rounded-md hover:bg-primary-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path d="M7.707 10.293a1 1 0 10-1.414 1.414l3 3a1 1 0 001.414 0l3-3a1 1 0 00-1.414-1.414L11 11.586V6h5a2 2 0 012 2v7a2 2 0 01-2 2H4a2 2 0 01-2-2V8a2 2 0 012-2h5v5.586l-1.293-1.293zM9 4a1 1 0 012 0v2H9V4z" />
                                        </svg>
                                        儲存本次分析
                                    </button>
                                    <button id="export-linkedin-excel" class="bg-green-600 text-white px-4 py-2 rounded-md hover:bg-green-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM6.293 6.707a1 1 0 010-1.414l3-3a1 1 0 011.414 0l3 3a1 1 0 01-1.414 1.414L11 5.414V13a1 1 0 11-2 0V5.414L7.707 6.707a1 1 0 01-1.414 0z" clip-rule="evenodd" />
                                        </svg>
                                        匯出為 Excel
                                    </button>
                                </div>
                            </div>
                            
                            <div class="mb-6">
                                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                                    <div>
                                        <label for="company-name" class="block text-sm font-medium text-gray-700 mb-1">公司名稱</label>
                                        <input type="text" id="company-name" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="輸入公司名稱">
                                    </div>
                                    <div>
                                        <label for="linkedin-url" class="block text-sm font-medium text-gray-700 mb-1">LinkedIn 公司頁面連結</label>
                                        <input type="url" id="linkedin-url" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="https://www.linkedin.com/company/...">
                                    </div>
                                    <div>
                                        <label for="company-industry" class="block text-sm font-medium text-gray-700 mb-1">產業類別</label>
                                        <select id="company-industry" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500">
                                            <option value="">請選擇產業</option>
                                            <!-- 產業選項將在此動態生成 -->
                                        </select>
                                    </div>
                                </div>
                                
                                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                                    <div>
                                        <label for="employee-count" class="block text-sm font-medium text-gray-700 mb-1">員工人數</label>
                                        <input type="number" id="employee-count" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="輸入員工人數">
                                    </div>
                                    <div>
                                        <label for="stock-code" class="block text-sm font-medium text-gray-700 mb-1">股票上市代碼</label>
                                        <input type="text" id="stock-code" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="例如：AAPL">
                                    </div>
                                    <div>
                                        <label for="company-location" class="block text-sm font-medium text-gray-700 mb-1">公司所在州別/地區</label>
                                        <input type="text" id="company-location" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="例如：California">
                                    </div>
                                </div>
                                
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                                    <div>
                                        <label for="email-suffix" class="block text-sm font-medium text-gray-700 mb-1">公司信箱後綴</label>
                                        <input type="text" id="email-suffix" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="例如：@company.com">
                                    </div>
                                    <div>
                                        <label for="company-type" class="block text-sm font-medium text-gray-700 mb-1">公司類型 (AI 分類)</label>
                                        <input type="text" id="company-type" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500" placeholder="例如：科技新創、跨國企業">
                                    </div>
                                </div>
                                
                                <div class="flex justify-end">
                                    <button id="add-company" class="bg-primary-600 text-white font-semibold px-4 py-2 rounded-md hover:bg-primary-700 transition shadow-sm flex items-center">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
                                        </svg>
                                        新增公司
                                    </button>
                                </div>
                            </div>
                            
                            <div class="overflow-x-auto bg-white rounded-lg border border-gray-200 shadow-sm">
                                <table class="min-w-full divide-y divide-gray-200">
                                    <thead class="bg-gray-50">
                                        <tr class="table-header">
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="company">
                                                公司名稱
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">LinkedIn 連結</th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="type">
                                                公司類型
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer" data-sort="employees">
                                                員工人數
                                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 inline-block ml-1" viewBox="0 0 20 20" fill="currentColor">
                                                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                                                </svg>
                                            </th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">股票代碼</th>
                                            <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">地區</th>
                                            <th class="py-3 px-4 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">操作</th>
                                        </tr>
                                    </thead>
                                    <tbody id="companies-table" class="bg-white divide-y divide-gray-200">
                                        <!-- 公司表格內容將在此動態生成 -->
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </section>
                </div>
            </div>
        </div>
        
        <div id="loading-overlay" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
            <div class="bg-white p-6 rounded-lg shadow-xl flex flex-col items-center">
                <div class="spinner mb-4"></div>
                <p id="loading-message" class="text-gray-700">處理中，請稍候...</p>
            </div>
        </div>
        
        <!-- 批量匯入關鍵字對話框 -->
        <div id="import-dialog" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
            <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-lg">
                <h3 class="text-lg font-semibold mb-4 text-gray-800">批量匯入關鍵字</h3>
                <p class="text-sm text-gray-600 mb-4">請輸入或貼上關鍵字，每行一個。格式：關鍵字,語言,搜尋量級,關鍵字類型,產業</p>
                <textarea id="import-keywords" rows="10" class="w-full p-2 border border-gray-300 rounded-md focus:ring-primary-500 focus:border-primary-500 mb-4" placeholder="例如：
人工智慧,zh-TW,high,technology,TECH
數位轉型,zh-TW,medium,general,TECH
雲端運算,zh-TW,high,product,TECH"></textarea>
                <div class="flex justify-end gap-2">
                    <button id="cancel-import" class="px-4 py-2 border border-gray-300 rounded-md hover:bg-gray-100 transition">取消</button>
                    <button id="confirm-import" class="bg-primary-600 text-white px-4 py-2 rounded-md hover:bg-primary-700 transition shadow-sm">匯入</button>
                </div>
            </div>
        </div>
    </main>

    <footer class="bg-white mt-12 py-6 border-t">
        <div class="container mx-auto px-6">
            <div class="flex flex-col md:flex-row justify-between items-center">
                <p class="text-gray-600">© 2023 市場情資與競品分析平台</p>
                <div class="flex gap-4 mt-4 md:mt-0">
                    <a href="#" class="text-gray-600 hover:text-primary-600 transition">使用說明</a>
                    <a href="#" class="text-gray-600 hover:text-primary-600 transition">隱私政策</a>
                    <a href="#" class="text-gray-600 hover:text-primary-600 transition">聯絡我們</a>
                </div>
            </div>
        </div>
    </footer>

    <script>
        // 全域變數
        let isAuthenticated = false;
        let currentProject = {
            name: '',
            client: '',
            folder: ''
        };
        let industries = [];
        let keywords = [];
        let competitors = [];
        let companies = [];
        let settings = {
            googleApiKey: '',
            searchEngineId: '',
            aiProvider: 'gemini',
            aiApiKey: ''
        };
        let history = [];

        // DOM 載入完成後執行
        document.addEventListener('DOMContentLoaded', function() {
            // 初始化 Google API
            initGoogleAPI();
            
            // 載入本地儲存的設定
            loadSettings();
            
            // 載入產業分類
            loadIndustries();
            
            // 設定事件監聽器
            setupEventListeners();
            
            // 載入示範資料
            loadDemoData();
        });

        // 初始化 Google API
        function initGoogleAPI() {
            gapi.load('client:auth2', () => {
                gapi.client.init({
                    apiKey: settings.googleApiKey,
                    clientId: '733619441496-5gigdpkv42k84no3q5vvh93ths1e8u29.apps.googleusercontent.com', // 請替換為您的 Google OAuth 客戶端 ID
                    discoveryDocs: [
                        'https://www.googleapis.com/discovery/v1/apis/drive/v3/rest',
                        'https://sheets.googleapis.com/$discovery/rest?version=v4'
                    ],
                    scope: 'https://www.googleapis.com/auth/drive https://www.googleapis.com/auth/spreadsheets https://www.googleapis.com/auth/userinfo.profile'
                }).then(() => {
                    // 監聽認證狀態變化
                    gapi.auth2.getAuthInstance().isSignedIn.listen(updateSigninStatus);
                    
                    // 設定初始認證狀態
                    updateSigninStatus(gapi.auth2.getAuthInstance().isSignedIn.get());
                    
                    // 設定登入按鈕事件
                    document.getElementById('authorize_button').onclick = handleAuthClick;
                    document.getElementById('signout_button').onclick = handleSignoutClick;
                }).catch(error => {
                    showAlert('Google API 初始化失敗: ' + error.message, 'error');
                });
            });
        }

        // 更新登入狀態
        function updateSigninStatus(isSignedIn) {
            isAuthenticated = isSignedIn;
            
            if (isSignedIn) {
                document.getElementById('authorize_button').classList.add('hidden');
                const userProfile = document.getElementById('user-profile');
                userProfile.classList.remove('hidden');
                userProfile.classList.add('flex');
                
                const user = gapi.auth2.getAuthInstance().currentUser.get();
                const profile = user.getBasicProfile();
                
                document.getElementById('user-avatar').src = profile.getImageUrl();
                document.getElementById('user-name').textContent = profile.getName();
                
                // 解鎖應用程式
                document.getElementById('app-container').classList.remove('locked');
                
                // 載入專案歷史
                loadProjectHistory();
            } else {
                document.getElementById('authorize_button').classList.remove('hidden');
                document.getElementById('user-profile').classList.add('hidden');
                document.getElementById('user-profile').classList.remove('flex');
                
                // 鎖定應用程式
                document.getElementById('app-container').classList.add('locked');
            }
        }

        // 處理登入點擊
        function handleAuthClick() {
            gapi.auth2.getAuthInstance().signIn();
        }

        // 處理登出點擊
        function handleSignoutClick() {
            gapi.auth2.getAuthInstance().signOut();
        }

        // 載入設定
        function loadSettings() {
            const savedSettings = localStorage.getItem('marketIntelSettings');
            if (savedSettings) {
                settings = JSON.parse(savedSettings);
                document.getElementById('google-api-key').value = settings.googleApiKey;
                document.getElementById('search-engine-id').value = settings.searchEngineId;
                document.getElementById('ai-provider').value = settings.aiProvider;
                document.getElementById('ai-api-key').value = settings.aiApiKey;
            }
        }

        // 儲存設定
        function saveSettings() {
            settings.googleApiKey = document.getElementById('google-api-key').value;
            settings.searchEngineId = document.getElementById('search-engine-id').value;
            settings.aiProvider = document.getElementById('ai-provider').value;
            settings.aiApiKey = document.getElementById('ai-api-key').value;
            
            localStorage.setItem('marketIntelSettings', JSON.stringify(settings));
            showAlert('設定已儲存', 'success');
        }

        // 載入產業分類
        function loadIndustries() {
            const savedIndustries = localStorage.getItem('marketIntelIndustries');
            if (savedIndustries) {
                industries = JSON.parse(savedIndustries);
                renderIndustriesList();
                updateIndustrySelects();
            } else {
                // 預設產業分類
                industries = [
                    { id: '1', name: '科技業', code: 'TECH', color: '#0ea5e9' },
                    { id: '2', name: '金融業', code: 'FIN', color: '#10b981' },
                    { id: '3', name: '零售業', code: 'RETAIL', color: '#f59e0b' },
                    { id: '4', name: '製造業', code: 'MFG', color: '#6366f1' },
                    { id: '5', name: '醫療保健', code: 'HEALTH', color: '#ec4899' }
                ];
                localStorage.setItem('marketIntelIndustries', JSON.stringify(industries));
                renderIndustriesList();
                updateIndustrySelects();
            }
        }

        // 新增產業分類
        function addIndustry() {
            const name = document.getElementById('industry-name').value;
            const code = document.getElementById('industry-code').value;
            const color = document.getElementById('industry-color').value;
            
            if (!name || !code) {
                showAlert('請填寫產業名稱和代碼', 'error');
                return;
            }
            
            // 檢查代碼是否已存在
            if (industries.some(ind => ind.code === code)) {
                showAlert('此產業代碼已存在', 'error');
                return;
            }
            
            const industry = {
                id: Date.now().toString(),
                name,
                code,
                color
            };
            
            industries.push(industry);
            localStorage.setItem('marketIntelIndustries', JSON.stringify(industries));
            
            // 清空輸入欄位
            document.getElementById('industry-name').value = '';
            document.getElementById('industry-code').value = '';
            
            // 更新產業列表和選擇器
            renderIndustriesList();
            updateIndustrySelects();
            
            showAlert('產業分類已新增', 'success');
        }

        // 刪除產業分類
        function deleteIndustry(id) {
            industries = industries.filter(ind => ind.id !== id);
            localStorage.setItem('marketIntelIndustries', JSON.stringify(industries));
            
            renderIndustriesList();
            updateIndustrySelects();
            
            showAlert('產業分類已刪除', 'success');
        }

        // 渲染產業分類列表
        function renderIndustriesList() {
            const list = document.getElementById('industries-list');
            list.innerHTML = '';
            
            if (industries.length === 0) {
                list.innerHTML = '<div class="text-gray-500 text-center py-4">尚未新增產業分類</div>';
                return;
            }
            
            industries.forEach(industry => {
                const item = document.createElement('div');
                item.className = 'flex justify-between items-center p-3 bg-white rounded-md border border-gray-200 shadow-sm';
                item.innerHTML = `
                    <div class="flex items-center">
                        <div class="w-4 h-4 rounded-full mr-3" style="background-color: ${industry.color}"></div>
                        <div>
                            <h4 class="font-medium text-gray-800">${industry.name}</h4>
                            <span class="text-xs text-gray-500">${industry.code}</span>
                        </div>
                    </div>
                    <button class="delete-industry text-gray-400 hover:text-red-600 transition" data-id="${industry.id}">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                        </svg>
                    </button>
                `;
                
                list.appendChild(item);
                
                // 添加刪除事件
                item.querySelector('.delete-industry').addEventListener('click', function() {
                    deleteIndustry(this.dataset.id);
                });
            });
        }

        // 更新產業選擇下拉選單
        function updateIndustrySelects() {
            const selects = [
                document.getElementById('keyword-industry'),
                document.getElementById('competitor-industry'),
                document.getElementById('company-industry')
            ];
            
            selects.forEach(select => {
                if (select) {
                    // 保留第一個選項
                    const firstOption = select.options[0];
                    select.innerHTML = '';
                    select.appendChild(firstOption);
                    
                    // 添加產業選項
                    industries.forEach(industry => {
                        const option = document.createElement('option');
                        option.value = industry.code;
                        option.textContent = industry.name;
                        select.appendChild(option);
                    });
                }
            });
        }

        // 新增關鍵字
        function addKeyword() {
            const text = document.getElementById('keyword-input').value;
            const language = document.getElementById('keyword-language').value;
            const volume = document.getElementById('search-volume').value;
            const type = document.getElementById('keyword-type').value;
            const industry = document.getElementById('keyword-industry').value;
            
            if (!text) {
                showAlert('請輸入關鍵字', 'error');
                return;
            }
            
            // 檢查是否已存在相同關鍵字
            if (keywords.some(k => k.text === text && k.language === language)) {
                showAlert('此關鍵字已存在', 'warning');
                return;
            }
            
            const keyword = {
                id: Date.now().toString(),
                text,
                language,
                volume,
                type,
                industry
            };
            
            keywords.push(keyword);
            saveKeywords();
            
            // 清空輸入欄位
            document.getElementById('keyword-input').value = '';
            
            // 更新關鍵字表格
            renderKeywordsTable();
            
            showAlert('關鍵字已新增', 'success');
        }

        // 批量匯入關鍵字
        function importKeywords() {
            const text = document.getElementById('import-keywords').value;
            if (!text.trim()) {
                showAlert('請輸入關鍵字資料', 'error');
                return;
            }
            
            const lines = text.trim().split('\n');
            let imported = 0;
            
            lines.forEach(line => {
                const parts = line.split(',');
                if (parts.length >= 5) {
                    const [text, language, volume, type, industry] = parts;
                    
                    // 檢查是否已存在相同關鍵字
                    if (!keywords.some(k => k.text === text && k.language === language)) {
                        const keyword = {
                            id: Date.now().toString() + Math.random().toString(36).substr(2, 5),
                            text,
                            language,
                            volume,
                            type,
                            industry
                        };
                        
                        keywords.push(keyword);
                        imported++;
                    }
                }
            });
            
            if (imported > 0) {
                saveKeywords();
                renderKeywordsTable();
                showAlert(`成功匯入 ${imported} 個關鍵字`, 'success');
            } else {
                showAlert('沒有匯入任何關鍵字', 'warning');
            }
            
            // 關閉對話框
            document.getElementById('import-dialog').classList.add('hidden');
        }

        // 刪除關鍵字
        function deleteKeyword(id) {
            keywords = keywords.filter(k => k.id !== id);
            saveKeywords();
            renderKeywordsTable();
            showAlert('關鍵字已刪除', 'success');
        }

        // 儲存關鍵字到本地儲存
        function saveKeywords() {
            localStorage.setItem(`marketIntelKeywords_${currentProject.folder || 'default'}`, JSON.stringify(keywords));
        }

        // 載入關鍵字
        function loadKeywords() {
            const savedKeywords = localStorage.getItem(`marketIntelKeywords_${currentProject.folder || 'default'}`);
            if (savedKeywords) {
                keywords = JSON.parse(savedKeywords);
            } else {
                keywords = [];
            }
            renderKeywordsTable();
        }

        // 渲染關鍵字表格
        function renderKeywordsTable(filter = 'all') {
            const table = document.getElementById('keywords-table');
            table.innerHTML = '';
            
            let filteredKeywords = keywords;
            if (filter !== 'all') {
                filteredKeywords = keywords.filter(k => k.type === filter);
            }
            
            if (filteredKeywords.length === 0) {
                const emptyRow = document.createElement('tr');
                emptyRow.innerHTML = `
                    <td colspan="7" class="py-4 text-center text-gray-500">尚未新增關鍵字</td>
                `;
                table.appendChild(emptyRow);
                return;
            }
            
            const volumeLabels = {
                'high': '高',
                'medium': '中',
                'low': '低'
            };
            
            const typeLabels = {
                'brand': '品牌字',
                'product': '產品字',
                'question': '問題字',
                'general': '一般字',
                'location': '地點字'
            };
            
            const languageLabels = {
                'zh-TW': '繁體中文',
                'zh-CN': '簡體中文',
                'en': '英文',
                'ja': '日文',
                'ko': '韓文'
            };
            
            filteredKeywords.forEach(keyword => {
                const row = document.createElement('tr');
                row.className = 'table-row';
                
                // 找到對應的產業
                const industry = industries.find(ind => ind.code === keyword.industry);
                
                row.innerHTML = `
                    <td class="py-3 px-4 whitespace-nowrap">${keyword.text}</td>
                    <td class="py-3 px-4 whitespace-nowrap">${languageLabels[keyword.language] || keyword.language}</td>
                    <td class="py-3 px-4 whitespace-nowrap">
                        <span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium ${keyword.volume === 'high' ? 'bg-green-100 text-green-800' : keyword.volume === 'medium' ? 'bg-yellow-100 text-yellow-800' : 'bg-red-100 text-red-800'}">
                            ${volumeLabels[keyword.volume] || keyword.volume}
                        </span>
                    </td>
                    <td class="py-3 px-4 whitespace-nowrap">${typeLabels[keyword.type] || keyword.type}</td>
                    <td class="py-3 px-4 whitespace-nowrap">
                        ${industry ? `<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium" style="background-color: ${industry.color}20; color: ${industry.color}">${industry.name}</span>` : ''}
                    </td>
                    <td class="py-3 px-4 text-right">
                        <button class="edit-keyword text-gray-500 hover:```
