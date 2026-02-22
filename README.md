<!DOCTYPE html>
<html lang="bn" class="scroll-smooth">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>OMR Pro</title>
    
    <!-- Libraries -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/js/all.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <link href="https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@300;400;500;600;700&display=swap" rel="stylesheet">

    <!-- Config & Styles -->
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: { sans: ['Hind Siliguri', 'sans-serif'] },
                    colors: {
                        primary: '#6366f1',
                        secondary: '#ec4899',
                        darkBg: '#0f172a',
                        cardDark: '#1e293b'
                    }
                }
            }
        }
    </script>

    <style>
        /* Security */
        .no-select { -webkit-touch-callout: none; -webkit-user-select: none; user-select: none; }

        /* Scrollbar */
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: #6366f1; border-radius: 10px; }
        
        /* 3D Button */
        .btn-3d { transition: all 0.1s; box-shadow: 0px 4px 0px 0px #3730a3; transform: translateY(0); }
        .btn-3d:active { transform: translateY(4px); box-shadow: 0px 0px 0px 0px #3730a3; }

        /* Glassmorphism */
        .glass-card { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.6); }
        .dark .glass-card { background: rgba(30, 41, 59, 0.95); border: 1px solid rgba(255, 255, 255, 0.1); }

        /* OMR Bubbles */
        .bubble { transition: all 0.2s ease; }
        .bubble.selected { transform: scale(1.15); box-shadow: 0 0 8px currentColor; }
        .bubble.locked { opacity: 0.6; cursor: not-allowed; }

        /* Animations */
        @keyframes float { 0% { transform: translateY(0px); } 50% { transform: translateY(-8px); } 100% { transform: translateY(0px); } }
        .anim-float { animation: float 3s ease-in-out infinite; }
        
        @keyframes pulse-ring { 0% { transform: scale(0.8); opacity: 0.5; } 100% { transform: scale(1.3); opacity: 0; } }
        .loader-ring { position: absolute; width: 100%; height: 100%; border-radius: 50%; border: 4px solid #fff; animation: pulse-ring 1.5s cubic-bezier(0.215, 0.61, 0.355, 1) infinite; }

        /* View Control */
        .view-section { display: none; }
        .view-section.active-view { display: block; animation: fadeIn 0.4s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Splash Scroll Fix */
        #splashSection { overflow-y: auto; -webkit-overflow-scrolling: touch; }
    </style>
</head>
<body class="bg-gray-100 dark:bg-darkBg text-gray-800 dark:text-gray-100 transition-colors duration-300 pb-20 no-select overflow-x-hidden">

    <!-- ================= 1. SPLASH / SETUP (Updated Contrast) ================= -->
    <div id="splashSection" class="hidden fixed inset-0 z-[100] bg-gradient-to-br from-indigo-600 via-purple-600 to-pink-600">
        <div class="min-h-screen flex flex-col items-center justify-center p-6 py-10">
            <div class="glass-card p-8 rounded-2xl w-full max-w-md shadow-2xl text-center">
                <div class="w-28 h-28 mx-auto bg-white rounded-full flex items-center justify-center mb-6 shadow-lg anim-float relative group">
                    <i class="fa-solid fa-graduation-cap text-5xl text-indigo-600" id="splashDefaultIcon"></i>
                    <img id="splashPreview" class="w-28 h-28 rounded-full object-cover hidden border-4 border-white">
                    <label for="profileUpload" class="absolute bottom-1 right-1 bg-pink-500 p-2.5 rounded-full cursor-pointer hover:bg-pink-600 transition shadow-lg hover:scale-110">
                        <i class="fa-solid fa-camera text-white text-sm"></i>
                    </label>
                    <input type="file" id="profileUpload" class="hidden" accept="image/*" onchange="previewImage(event, 'splashPreview', 'splashDefaultIcon')">
                </div>
                <h1 class="text-3xl font-bold text-white mb-2 drop-shadow-md">OMR Pro</h1>
                <p class="text-indigo-100 mb-6 text-sm font-medium">আপনার ডিজিটাল এক্সাম পার্টনার</p>
                
                <!-- Updated Inputs for Better Visibility -->
                <div class="space-y-4 text-left">
                    <div>
                        <label class="text-indigo-100 text-xs font-bold ml-1">আপনার নাম</label>
                        <input type="text" id="startName" placeholder="Type your name" class="w-full p-3 rounded-xl bg-white text-gray-800 border border-gray-200 focus:outline-none focus:ring-4 focus:ring-indigo-300 transition shadow-sm">
                    </div>
                    <div>
                        <label class="text-indigo-100 text-xs font-bold ml-1">শ্রেণী / ব্যাচ</label>
                        <input type="text" id="startClass" placeholder="Ex: HSC 24" class="w-full p-3 rounded-xl bg-white text-gray-800 border border-gray-200 focus:outline-none focus:ring-4 focus:ring-indigo-300 transition shadow-sm">
                    </div>
                    <div>
                        <label class="text-indigo-100 text-xs font-bold ml-1">প্রতিষ্ঠান</label>
                        <input type="text" id="startInst" placeholder="College Name" class="w-full p-3 rounded-xl bg-white text-gray-800 border border-gray-200 focus:outline-none focus:ring-4 focus:ring-indigo-300 transition shadow-sm">
                    </div>
                </div>

                <button onclick="firstTimeSetup()" class="w-full mt-8 bg-white text-indigo-600 font-bold py-3.5 rounded-xl shadow-xl btn-3d text-lg hover:bg-gray-50">
                    Continue <i class="fa-solid fa-arrow-right ml-2"></i>
                </button>
            </div>
        </div>
    </div>

    <!-- ================= 2. LOADING SCREEN ================= -->
    <div id="loadingSection" class="hidden fixed inset-0 z-[100] bg-gradient-to-br from-indigo-600 to-purple-700 flex flex-col items-center justify-center">
        <div class="relative w-24 h-24 mb-6">
            <div class="loader-ring"></div>
            <div class="w-24 h-24 bg-white rounded-full flex items-center justify-center shadow-lg relative z-10">
                <i class="fa-solid fa-pen-nib text-4xl text-indigo-600"></i>
            </div>
        </div>
        <h2 class="text-2xl font-bold text-white tracking-widest animate-pulse">OMR PRO</h2>
        <div class="w-48 h-1.5 bg-white/20 rounded-full mt-4 overflow-hidden">
            <div class="h-full bg-white animate-[width_2s_ease-in-out_forwards]" style="width: 0%"></div>
        </div>
    </div>

    <!-- ================= MAIN APP CONTAINER ================= -->
    <div id="mainApp" class="hidden min-h-screen relative">
        
        <!-- HEADER -->
        <header class="sticky top-0 z-40 glass-card px-4 py-3 flex justify-between items-center shadow-sm">
            <div class="flex items-center gap-3">
                <div class="w-9 h-9 bg-gradient-to-r from-indigo-500 to-pink-500 rounded-xl flex items-center justify-center text-white font-bold shadow-md transform rotate-3">
                    <i class="fa-solid fa-pen-nib text-sm"></i>
                </div>
                <h1 class="text-xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-indigo-600 to-pink-600 dark:from-indigo-400 dark:to-pink-400">OMR Pro</h1>
            </div>
            <button onclick="toggleDarkMode()" class="w-10 h-10 rounded-full bg-gray-100 dark:bg-gray-700 flex items-center justify-center hover:bg-gray-200 dark:hover:bg-gray-600 transition">
                <i class="fa-solid fa-moon text-gray-600 dark:text-yellow-400 text-lg" id="themeIcon"></i>
            </button>
        </header>

        <!-- ADSTERRA BANNER -->
        <div class="w-full flex justify-center my-3">
           <script>
  atOptions = {
    'key' : 'c1dba33ec4e483e2feb78412669f621d',
    'format' : 'iframe',
    'height' : 50,
    'width' : 320,
    'params' : {}
  };
</script>
<script src="https://www.highperformanceformat.com/c1dba33ec4e483e2feb78412669f621d/invoke.js"></script>
        </div>

        <!-- ================= 3. HOME VIEW ================= -->
        <div id="homeView" class="view-section active-view p-4 pb-24">
            
            <div class="mb-6 flex items-center gap-4 bg-white dark:bg-cardDark p-4 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-700">
                <img id="homeProfileImg" src="https://via.placeholder.com/150" class="w-14 h-14 rounded-full border-2 border-indigo-500 object-cover shadow-sm p-0.5">
                <div>
                    <p class="text-xs font-bold text-gray-400 uppercase tracking-wide">Welcome Back</p>
                    <h2 id="homeUserName" class="text-xl font-bold text-gray-800 dark:text-white leading-tight">Student</h2>
                </div>
            </div>

            <!-- Give Exam Card -->
            <div onclick="openSetupView()" class="cursor-pointer group relative overflow-hidden rounded-2xl bg-gradient-to-r from-indigo-600 to-blue-600 p-6 shadow-xl shadow-indigo-500/20 transform transition active:scale-95 mb-4">
                <div class="absolute -right-6 -top-6 w-32 h-32 bg-white/10 rounded-full blur-2xl group-hover:bg-white/20 transition"></div>
                <div class="relative z-10 flex justify-between items-center">
                    <div>
                        <h3 class="text-2xl font-bold text-white mb-1">OMR এ পরীক্ষা দিই</h3>
                        <p class="text-indigo-100 text-sm opacity-90">Custom Exam Setup</p>
                    </div>
                    <div class="w-12 h-12 bg-white/20 rounded-full flex items-center justify-center backdrop-blur-sm shadow-inner">
                        <i class="fa-solid fa-play text-white text-xl pl-1"></i>
                    </div>
                </div>
            </div>

            <!-- PDF Zone -->
            <a href="https://t.me/pdfnexusclass=""" target="_blank" class="block cursor-pointer group relative overflow-hidden rounded-2xl bg-gradient-to-r from-pink-500 to-rose-500 p-6 shadow-xl shadow-pink-500/20 transform transition active:scale-95 mb-4">
                <div class="absolute top-3 right-3 text-white/90 animate-pulse">
                    <i class="fa-brands fa-telegram text-2xl"></i>
                </div>
                <div class="relative z-10">
                    <h3 class="text-xl font-bold text-white mb-1">PDF Zone</h3>
                    <p class="text-pink-100 text-sm opacity-90">SSC, HSC & Admission</p>
                </div>
            </a>

            <!-- Manual Button -->
            <button onclick="showManual()" class="w-full bg-white dark:bg-cardDark border border-gray-200 dark:border-gray-700 p-3 rounded-xl flex items-center justify-center gap-2 text-gray-600 dark:text-gray-300 font-bold text-sm shadow-sm hover:bg-gray-50 dark:hover:bg-gray-800 mb-6">
                <i class="fa-solid fa-book-open"></i> User Manual
            </button>

            <!-- SOCIAL ADS -->
            <div class="mb-4">
                <h3 class="text-xs font-bold text-gray-400 mb-2 px-1 uppercase tracking-wider"></h3>
               <script src="https://pl25271999.effectivegatecpm.com/3f/c4/4a/3fc44a02e9322868afb4be6ebc1bba64.js"></script>
            </div>

            <!-- SMART STUDY TRACKER CARD -->
            <div onclick="openTrackerView()" class="cursor-pointer group relative overflow-hidden rounded-2xl bg-gradient-to-r from-teal-500 to-emerald-500 p-6 shadow-xl shadow-teal-500/20 transform transition active:scale-95 mb-6">
                <div class="absolute -left-6 -bottom-6 w-32 h-32 bg-white/10 rounded-full blur-2xl group-hover:bg-white/20 transition"></div>
                <div class="relative z-10 flex justify-between items-center">
                    <div>
                        <h3 class="text-xl font-bold text-white mb-1">Smart Study Tracker</h3>
                        <p class="text-teal-100 text-sm opacity-90">Daily Analytics & Graphs</p>
                    </div>
                    <div class="w-10 h-10 bg-white/20 rounded-full flex items-center justify-center backdrop-blur-sm shadow-inner">
                        <i class="fa-solid fa-chart-simple text-white text-lg"></i>
                    </div>
                </div>
            </div>

            <!-- NATIVE ADS -->
            <div class="mb-6">
                <h3 class="text-xs font-bold text-gray-400 mb-2 px-1 uppercase tracking-wider">Sponsored</h3>
                <script async="async" data-cfasync="false" src="https://pl25272314.effectivegatecpm.com/77853915d68a40c947c4ad0442acfe1b/invoke.js"></script> <div id="container-77853915d68a40c947c4ad0442acfe1b"></div>
            </div>

        </div>

        <!-- ================= 4. TRACKER VIEW ================= -->
        <div id="trackerView" class="view-section p-4 pb-24">
            <h2 class="text-2xl font-bold mb-4 text-teal-600 dark:text-teal-400">Study Analytics</h2>
            <div class="glass-card p-5 rounded-2xl shadow-lg mb-6">
                <h3 class="text-sm font-bold mb-3 text-gray-600 dark:text-gray-300">আজকের পড়া যুক্ত করুন</h3>
                <div class="space-y-3">
                    <input type="text" id="trkSubject" placeholder="Subject (e.g. Math)" class="w-full p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 text-sm focus:outline-none focus:border-teal-500">
                    <div class="flex gap-3">
                        <input type="number" id="trkHours" placeholder="Hours" class="w-1/2 p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 text-sm focus:outline-none focus:border-teal-500">
                        <select id="trkRating" class="w-1/2 p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 text-sm focus:outline-none focus:border-teal-500">
                            <option value="5">Excellent (5)</option>
                            <option value="4">Good (4)</option>
                            <option value="3">Average (3)</option>
                            <option value="2">Bad (2)</option>
                        </select>
                    </div>
                    <button onclick="saveStudyLog()" class="w-full bg-teal-500 text-white font-bold py-3 rounded-xl shadow-lg btn-3d">
                        Add Log <i class="fa-solid fa-plus ml-1"></i>
                    </button>
                </div>
            </div>
            <div class="grid grid-cols-2 gap-3 mb-6">
                <div class="bg-indigo-100 dark:bg-indigo-900/30 p-4 rounded-xl border border-indigo-200 dark:border-indigo-800 text-center">
                    <p class="text-xs font-bold text-indigo-500 mb-1">Today's Study</p>
                    <h4 class="text-2xl font-black text-indigo-700 dark:text-indigo-300" id="statDaily">0h</h4>
                </div>
                <div class="bg-pink-100 dark:bg-pink-900/30 p-4 rounded-xl border border-pink-200 dark:border-pink-800 text-center">
                    <p class="text-xs font-bold text-pink-500 mb-1">Monthly Total</p>
                    <h4 class="text-2xl font-black text-pink-700 dark:text-pink-300" id="statMonthly">0h</h4>
                </div>
            </div>
            <div class="glass-card p-4 rounded-xl shadow-sm mb-6 space-y-3">
                <div class="flex justify-between items-center border-b dark:border-gray-700 pb-2">
                    <span class="text-xs font-bold text-gray-500">🔥 Most Studied</span>
                    <span class="font-bold text-teal-600 dark:text-teal-400" id="statMost">-</span>
                </div>
                <div class="flex justify-between items-center">
                    <span class="text-xs font-bold text-gray-500">⚠️ Least Focused</span>
                    <span class="font-bold text-red-500" id="statLeast">-</span>
                </div>
            </div>
            <div class="bg-white dark:bg-cardDark p-4 rounded-2xl shadow-lg border border-gray-100 dark:border-gray-700">
                <h4 class="text-xs font-bold text-gray-400 uppercase mb-4">Weekly Progress</h4>
                <div class="h-40">
                    <canvas id="studyChart"></canvas>
                </div>
            </div>
            <button onclick="goToView('homeView')" class="mt-6 w-full py-3 bg-gray-200 dark:bg-gray-700 rounded-xl font-bold text-sm text-gray-600 dark:text-gray-300">Back to Home</button>
        </div>

        <!-- ================= 5. SETUP VIEW ================= -->
        <div id="setupView" class="view-section p-4 pb-24">
            <h2 class="text-2xl font-bold mb-4 bg-clip-text text-transparent bg-gradient-to-r from-indigo-600 to-pink-600">Exam Configuration</h2>
            <div class="glass-card p-5 rounded-2xl space-y-5 shadow-lg">
                <input type="text" id="exName" placeholder="পরীক্ষার নাম (e.g. Physics Ch-1)" class="w-full p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 focus:ring-2 focus:ring-indigo-500 focus:outline-none transition">
                <div class="grid grid-cols-2 gap-4">
                    <input type="number" id="exQues" placeholder="Questions" class="p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 focus:ring-2 focus:ring-indigo-500 focus:outline-none transition">
                    <input type="number" id="exTime" placeholder="Time (Min)" class="p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 focus:ring-2 focus:ring-indigo-500 focus:outline-none transition">
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <input type="number" id="exNeg" placeholder="Neg. Mark (0.25)" step="0.01" class="p-3 rounded-xl bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 focus:ring-2 focus:ring-indigo-500 focus:outline-none transition">
                    <div class="flex flex-col justify-center gap-1 px-1">
                         <label class="text-[10px] font-bold text-gray-500 uppercase">Change Answer?</label>
                         <div class="flex gap-2">
                             <label class="flex items-center gap-1 text-xs cursor-pointer"><input type="radio" name="changeOpt" value="allow" checked class="accent-indigo-500"> Allow</label>
                             <label class="flex items-center gap-1 text-xs cursor-pointer"><input type="radio" name="changeOpt" value="notallow" class="accent-pink-500"> Not Allow</label>
                         </div>
                    </div>
                </div>
                <div>
                    <label class="block text-xs font-bold mb-2 text-gray-500 uppercase">অপশন টাইপ</label>
                    <div class="grid grid-cols-3 gap-2">
                        <button onclick="setOptionType('bangla', this)" class="opt-type-btn active p-3 border rounded-xl text-center text-sm font-bold bg-indigo-100 dark:bg-indigo-900 border-indigo-500 text-indigo-700 dark:text-indigo-200 transition">ক,খ,গ,ঘ</button>
                        <button onclick="setOptionType('english', this)" class="opt-type-btn p-3 border rounded-xl text-center text-sm font-bold border-gray-200 dark:border-gray-600 text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700 transition">A,B,C,D</button>
                        <button onclick="setOptionType('roman', this)" class="opt-type-btn p-3 border rounded-xl text-center text-sm font-bold border-gray-200 dark:border-gray-600 text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700 transition">i,ii,iii</button>
                    </div>
                </div>
                <button onclick="generateOMR()" class="w-full mt-2 bg-gradient-to-r from-indigo-600 to-purple-600 text-white font-bold py-3.5 rounded-xl shadow-lg btn-3d">
                    Generate OMR <i class="fa-solid fa-wand-magic-sparkles ml-2"></i>
                </button>
            </div>
        </div>

        <!-- ================= 6. EXAM VIEW ================= -->
        <div id="examView" class="view-section p-4 pb-24 min-h-screen">
            <div class="sticky top-[70px] z-30 bg-white/90 dark:bg-darkBg/95 backdrop-blur-md p-3 rounded-xl shadow-md flex justify-between items-center mb-4 border border-indigo-100 dark:border-gray-700">
                <span id="displayExName" class="font-bold text-indigo-600 dark:text-indigo-400 truncate max-w-[50%]">Exam</span>
                <div class="flex items-center gap-2 bg-red-50 dark:bg-red-900/40 border border-red-100 dark:border-red-800 px-3 py-1 rounded-full">
                    <i class="fa-regular fa-clock text-red-500 animate-pulse"></i>
                    <span id="timerDisplay" class="font-mono font-bold text-red-600 dark:text-red-400">00:00</span>
                </div>
            </div>
            <div id="omrSheet" class="space-y-3"></div>
            <div class="fixed bottom-24 left-4 right-4 z-40">
                <button onclick="submitExam()" class="w-full bg-gradient-to-r from-green-500 to-emerald-600 text-white font-bold py-4 rounded-xl shadow-2xl btn-3d text-lg tracking-wide">SUBMIT EXAM</button>
            </div>
        </div>

        <!-- ================= 7. SOLUTION VIEW ================= -->
        <div id="solutionView" class="view-section p-4 pb-24">
            <div class="bg-yellow-50 dark:bg-yellow-900/20 p-4 rounded-xl border border-yellow-200 dark:border-yellow-700 mb-4 text-center">
                <h3 class="font-bold text-yellow-700 dark:text-yellow-400">উত্তরপত্র তৈরি করুন</h3>
            </div>
            <div id="solutionSheet" class="space-y-3 mb-20"></div>
            <div class="fixed bottom-24 left-4 right-4 z-40">
                <button onclick="processResult()" class="w-full bg-gradient-to-r from-purple-600 to-pink-600 text-white font-bold py-4 rounded-xl shadow-2xl btn-3d text-lg">SEE RESULT</button>
            </div>
        </div>

        <!-- ================= 8. RESULT VIEW ================= -->
        <div id="resultView" class="view-section p-4 pb-24">
            <div class="glass-card p-6 rounded-3xl shadow-lg mb-6 text-center relative overflow-hidden">
                <div class="h-64 relative mb-2 flex items-center justify-center">
                    <canvas id="resultChart"></canvas>
                </div>
                <div class="mt-2">
                     <span class="text-gray-400 text-[10px] font-bold tracking-widest uppercase">Total Score</span><br>
                     <span class="text-3xl font-black text-gray-800 dark:text-white" id="finalScoreDisplay">0.0</span>
                </div>
            </div>
            <div class="flex justify-between gap-2 mb-4 overflow-x-auto pb-2 scrollbar-hide">
                <button onclick="filterResult('all')" class="flex-1 min-w-[80px] px-3 py-2 bg-white dark:bg-gray-700 rounded-lg text-xs font-bold border border-gray-200 dark:border-gray-600">ALL</button>
                <button onclick="filterResult('right')" class="flex-1 min-w-[80px] px-3 py-2 bg-green-50 dark:bg-green-900/30 text-green-600">RIGHT</button>
                <button onclick="filterResult('wrong')" class="flex-1 min-w-[80px] px-3 py-2 bg-red-50 dark:bg-red-900/30 text-red-600">WRONG</button>
            </div>
            <div id="resultDetails" class="space-y-2"></div>
            <button onclick="goToView('homeView')" class="fixed top-4 right-4 bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-1.5 rounded-full text-xs font-bold shadow-lg z-50">Home</button>
        </div>

        <!-- ================= 9. RECORDS VIEW ================= -->
        <div id="recordsView" class="view-section p-4 pb-24">
            <h2 class="text-2xl font-bold mb-4">History</h2>
            <div id="historyContainer" class="space-y-4"></div>
        </div>

        <!-- ================= 10. DASHBOARD VIEW (Updated Labels) ================= -->
        <div id="dashboardView" class="view-section p-4 pb-24">
            <h2 class="text-2xl font-bold mb-4">Dashboard</h2>
            <div class="glass-card p-6 rounded-2xl shadow-lg relative">
                <div id="dashDisplayMode" class="text-center">
                    <img id="dashProfileImg" src="" class="w-24 h-24 rounded-full mx-auto mb-4 border-4 border-indigo-500 object-cover shadow-md">
                    <h3 id="dashName" class="text-xl font-bold text-gray-800 dark:text-white">User Name</h3>
                    <p id="dashDetails" class="text-sm text-gray-500 mb-6 font-medium">Class | Inst</p>
                    <button onclick="toggleEditMode(true)" class="bg-pink-500 hover:bg-pink-600 text-white px-6 py-2.5 rounded-xl shadow btn-3d text-sm font-bold">Edit Profile <i class="fa-solid fa-pen ml-1"></i></button>
                </div>
                
                <!-- Updated Edit Form with Labels & Placeholders -->
                <div id="dashEditMode" class="hidden text-left space-y-4">
                    <div class="text-center mb-4">
                        <div class="w-20 h-20 mx-auto relative">
                             <img id="editPreview" src="" class="w-20 h-20 rounded-full object-cover border-2 border-indigo-500">
                             <label for="editUpload" class="absolute bottom-0 right-0 bg-gray-800 text-white p-1 rounded-full text-xs cursor-pointer"><i class="fa-solid fa-camera"></i></label>
                             <input type="file" id="editUpload" class="hidden" accept="image/*" onchange="previewImage(event, 'editPreview', null)">
                        </div>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-gray-500 mb-1">Name</label>
                        <input type="text" id="editName" placeholder="Enter Name" class="w-full p-2 rounded-lg bg-gray-100 dark:bg-gray-700 border dark:border-gray-600 text-sm">
                    </div>
                    <div>
                         <label class="block text-xs font-bold text-gray-500 mb-1">Class</label>
                         <input type="text" id="editClass" placeholder="Enter Class" class="w-full p-2 rounded-lg bg-gray-100 dark:bg-gray-700 border dark:border-gray-600 text-sm">
                    </div>
                    <div>
                         <label class="block text-xs font-bold text-gray-500 mb-1">Institute</label>
                         <input type="text" id="editInst" placeholder="Enter Institute" class="w-full p-2 rounded-lg bg-gray-100 dark:bg-gray-700 border dark:border-gray-600 text-sm">
                    </div>
                    
                    <div class="flex gap-2 pt-2">
                        <button onclick="toggleEditMode(false)" class="flex-1 bg-gray-300 dark:bg-gray-700 py-2 rounded-lg font-bold text-sm">Cancel</button>
                        <button onclick="saveProfileUpdate()" class="flex-1 bg-indigo-600 text-white py-2 rounded-lg font-bold text-sm shadow-md">Save</button>
                    </div>
                </div>
                <div class="mt-8 border-t dark:border-gray-600 pt-4 text-center">
                    <button onclick="if(confirm('Reset App? All data will be lost.')){localStorage.clear(); location.reload()}" class="text-red-500 hover:text-red-600 text-xs font-bold py-2 px-4">Clear All Data</button>
                </div>
            </div>
        </div>

        <!-- ================= BOTTOM NAVIGATION ================= -->
        <nav class="fixed bottom-0 w-full glass-card border-t border-gray-200 dark:border-gray-700 flex justify-around py-3 pb-5 z-50 shadow-[0_-5px_20px_rgba(0,0,0,0.05)] backdrop-blur-xl bg-white/90 dark:bg-darkBg/90">
            <button onclick="goToView('homeView')" class="nav-btn text-indigo-600 flex flex-col items-center transform transition active:scale-90">
                <i class="fa-solid fa-house text-xl mb-1"></i>
                <span class="text-[10px] font-bold">Home</span>
            </button>
            <button onclick="loadHistory(); goToView('recordsView')" class="nav-btn text-gray-400 hover:text-pink-500 flex flex-col items-center transform transition active:scale-90">
                <i class="fa-solid fa-list-check text-xl mb-1"></i>
                <span class="text-[10px] font-bold">Records</span>
            </button>
            <button onclick="updateDashboardUI(); goToView('dashboardView')" class="nav-btn text-gray-400 hover:text-indigo-500 flex flex-col items-center transform transition active:scale-90">
                <i class="fa-solid fa-user-gear text-xl mb-1"></i>
                <span class="text-[10px] font-bold">Dashboard</span>
            </button>
        </nav>

    </div>

    <!-- ================= JAVASCRIPT ================= -->
    <script>
        // Config Chart DataLabels
        Chart.register(ChartDataLabels);

        // Security
        document.addEventListener('contextmenu', e => e.preventDefault());
        document.onkeydown = e => { if(e.keyCode == 123 || (e.ctrlKey && e.shiftKey && (e.keyCode == 73 || e.keyCode == 67 || e.keyCode == 74)) || (e.ctrlKey && e.keyCode == 85)) return false; }

        // State
        let userProfile = JSON.parse(localStorage.getItem('omr_user')) || null;
        let currentExam = {};
        let userAnswers = {};
        let solutionKeys = {};
        let optionType = 'bangla';
        let timerInterval;
        const optionLabels = { 'bangla': ['ক', 'খ', 'গ', 'ঘ'], 'english': ['A', 'B', 'C', 'D'], 'roman': ['i', 'ii', 'iii', 'iv'] };

        // Init
        window.onload = () => {
            if(localStorage.getItem('theme') === 'dark') document.documentElement.classList.add('dark');
            if (userProfile) {
                document.getElementById('loadingSection').classList.remove('hidden');
                setTimeout(() => {
                    document.getElementById('loadingSection').classList.add('hidden');
                    document.getElementById('mainApp').classList.remove('hidden');
                    applyProfileData();
                    goToView('homeView');
                }, 2000); 
            } else {
                document.getElementById('splashSection').classList.remove('hidden');
            }
        };

        // Navigation
        function goToView(viewId) {
            document.getElementById('splashSection').classList.add('hidden');
            document.getElementById('loadingSection').classList.add('hidden');
            document.getElementById('mainApp').classList.remove('hidden');
            document.querySelectorAll('.view-section').forEach(el => el.classList.remove('active-view'));
            document.getElementById(viewId).classList.add('active-view');
            updateNavIcons(viewId);
            window.scrollTo(0,0);
        }
        function openSetupView() {
            document.getElementById('exName').value = '';
            document.getElementById('exQues').value = '';
            document.getElementById('exTime').value = '';
            document.getElementById('exNeg').value = '';
            document.querySelector('input[name="changeOpt"][value="allow"]').checked = true;
            goToView('setupView');
        }
        function updateNavIcons(activeId) {
            const btns = document.querySelectorAll('.nav-btn');
            btns.forEach(b => b.className = "nav-btn text-gray-400 flex flex-col items-center transform transition active:scale-90");
            if(activeId === 'homeView') btns[0].className = "nav-btn text-indigo-600 flex flex-col items-center transform transition active:scale-90";
            if(activeId === 'recordsView') btns[1].className = "nav-btn text-pink-500 flex flex-col items-center transform transition active:scale-90";
            if(activeId === 'dashboardView') btns[2].className = "nav-btn text-indigo-600 flex flex-col items-center transform transition active:scale-90";
        }
        function showManual() { alert('1. Home থেকে "OMR এ পরীক্ষা দিই" তে ক্লিক করুন।\n2. প্রশ্নের সংখ্যা ও সময় সেট করে OMR তৈরি করুন।\n3. পরীক্ষা শেষ হলে Submit করুন।\n4. এরপর গাইডের সঠিক উত্তর দেখে ২য় OMR টি পূরণ করুন।\n5. See Result দিলে ফলাফল দেখতে পাবেন।'); }

        // --- SMART STUDY TRACKER ---
        function openTrackerView() {
            renderTrackerStats();
            goToView('trackerView');
        }

        function saveStudyLog() {
            const sub = document.getElementById('trkSubject').value;
            const hrs = parseFloat(document.getElementById('trkHours').value);
            const rate = parseInt(document.getElementById('trkRating').value);

            if(!sub || !hrs) return alert("Please fill all fields");

            const log = { id: Date.now(), date: new Date().toLocaleDateString(), sub, hrs, rate };
            let logs = JSON.parse(localStorage.getItem('omr_study_logs')) || [];
            logs.push(log);
            localStorage.setItem('omr_study_logs', JSON.stringify(logs));

            document.getElementById('trkSubject').value = '';
            document.getElementById('trkHours').value = '';
            alert("Log Added!");
            renderTrackerStats();
        }

        let studyChartInstance = null;
        function renderTrackerStats() {
            const logs = JSON.parse(localStorage.getItem('omr_study_logs')) || [];
            const today = new Date().toLocaleDateString();

            let daily = 0, monthly = 0;
            logs.forEach(l => {
                if(l.date === today) daily += l.hrs;
                monthly += l.hrs;
            });
            document.getElementById('statDaily').innerText = daily + 'h';
            document.getElementById('statMonthly').innerText = monthly + 'h';

            const counts = {};
            logs.forEach(l => { counts[l.sub] = (counts[l.sub] || 0) + l.hrs; });
            
            let maxHrs = 0, mostSub = '-';
            for(let s in counts) { if(counts[s] > maxHrs) { maxHrs = counts[s]; mostSub = s; } }
            document.getElementById('statMost').innerText = mostSub;

            let minHrs = 9999, leastSub = '-';
            for(let s in counts) { if(counts[s] < minHrs) { minHrs = counts[s]; leastSub = s; } }
            if(minHrs === 9999) leastSub = '-';
            document.getElementById('statLeast').innerText = leastSub;

            const ctx = document.getElementById('studyChart').getContext('2d');
            const recentLogs = logs.slice(-7);
            const labels = recentLogs.map(l => l.sub.substring(0,3));
            const data = recentLogs.map(l => l.hrs);

            if(studyChartInstance) studyChartInstance.destroy();
            studyChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{ label: 'Hours', data: data, backgroundColor: '#2dd4bf', borderRadius: 5 }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false }, datalabels: { display: false } },
                    scales: { y: { beginAtZero: true, grid: { display: false } }, x: { grid: { display: false } } }
                }
            });
        }

        // --- EXAM LOGIC ---
        function setOptionType(type, btn) {
            optionType = type;
            document.querySelectorAll('.opt-type-btn').forEach(b => b.className = "opt-type-btn p-3 border rounded-xl text-center text-sm font-bold border-gray-200 dark:border-gray-600 text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700 transition");
            btn.className = "opt-type-btn active p-3 border rounded-xl text-center text-sm font-bold bg-indigo-100 dark:bg-indigo-900 border-indigo-500 text-indigo-700 dark:text-indigo-200 transition";
        }

        function generateOMR() {
            const allowChange = document.querySelector('input[name="changeOpt"]:checked').value === 'allow';
            currentExam = {
                name: document.getElementById('exName').value || 'Test Exam',
                total: parseInt(document.getElementById('exQues').value) || 25,
                time: parseInt(document.getElementById('exTime').value) || 20,
                neg: parseFloat(document.getElementById('exNeg').value) || 0,
                allowChange: allowChange
            };
            const container = document.getElementById('omrSheet');
            container.innerHTML = '';
            userAnswers = {};
            for(let i=1; i<=currentExam.total; i++) {
                let row = document.createElement('div');
                row.className = "glass-card p-3 rounded-xl flex items-center justify-between animate-[fadeIn_0.5s_ease_forwards]";
                row.style.animationDelay = `${i * 0.02}s`;
                row.innerHTML = `<span class="w-8 font-bold text-gray-600 dark:text-gray-300">${i < 10 ? '0'+i : i}</span>`;
                let bubbleCont = document.createElement('div');
                bubbleCont.className = "flex gap-3";
                optionLabels[optionType].forEach(opt => {
                    let b = document.createElement('div');
                    b.className = "bubble w-9 h-9 rounded-full border-2 border-gray-300 dark:border-gray-600 flex items-center justify-center cursor-pointer font-bold text-xs text-gray-500 dark:text-gray-400 hover:border-indigo-400";
                    b.innerText = opt;
                    b.onclick = () => selectBubble(i, opt, b, 'exam');
                    bubbleCont.appendChild(b);
                });
                row.appendChild(bubbleCont);
                container.appendChild(row);
            }
            startTimer(currentExam.time);
            document.getElementById('displayExName').innerText = currentExam.name;
            goToView('examView');
        }

        function selectBubble(qNum, opt, el, mode) {
            const store = mode === 'exam' ? userAnswers : solutionKeys;
            if (mode === 'exam' && !currentExam.allowChange && store[qNum]) return;
            el.parentElement.querySelectorAll('.bubble').forEach(b => {
                b.classList.remove('selected', 'bg-gray-800', 'text-white', 'dark:bg-white', 'dark:text-black', 'border-transparent');
            });
            el.classList.add('selected', 'bg-gray-800', 'text-white', 'dark:bg-white', 'dark:text-black', 'border-transparent');
            if (mode === 'exam' && !currentExam.allowChange) {
                 el.parentElement.querySelectorAll('.bubble').forEach(b => { if(b !== el) b.classList.add('locked'); });
            }
            store[qNum] = opt;
        }

        function startTimer(minutes) {
            let seconds = minutes * 60;
            const display = document.getElementById('timerDisplay');
            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                let m = Math.floor(seconds / 60);
                let s = seconds % 60;
                display.innerText = `${m}:${s < 10 ? '0'+s : s}`;
                if (seconds <= 0) { clearInterval(timerInterval); submitExam(); }
                seconds--;
            }, 1000);
        }

        function submitExam() {
            clearInterval(timerInterval);
            const container = document.getElementById('solutionSheet');
            container.innerHTML = '';
            solutionKeys = {};
            for(let i=1; i<=currentExam.total; i++) {
                let row = document.createElement('div');
                row.className = "bg-white dark:bg-cardDark p-2 rounded-lg flex items-center justify-between border border-gray-100 dark:border-gray-700";
                row.innerHTML = `<span class="w-6 font-bold text-sm text-gray-500">${i}</span>`;
                let bubbleCont = document.createElement('div');
                bubbleCont.className = "flex gap-2";
                optionLabels[optionType].forEach(opt => {
                    let b = document.createElement('div');
                    b.className = "bubble w-8 h-8 rounded-full border border-gray-300 flex items-center justify-center cursor-pointer text-xs";
                    b.innerText = opt;
                    b.onclick = () => selectBubble(i, opt, b, 'solution');
                    bubbleCont.appendChild(b);
                });
                row.appendChild(bubbleCont);
                container.appendChild(row);
            }
            goToView('solutionView');
        }

        function processResult() {
            let right = 0, wrong = 0, skipped = 0;
            for(let i=1; i<=currentExam.total; i++) {
                let u = userAnswers[i];
                let c = solutionKeys[i];
                if(!u) skipped++; else if(u === c) right++; else wrong++;
            }
            let score = (right * 1) - (wrong * currentExam.neg);
            saveHistory(score);
            renderChart(right, wrong, skipped);
            document.getElementById('finalScoreDisplay').innerText = score.toFixed(2);
            filterResult('all');
            goToView('resultView');
        }

        let chartInstance = null;
        function renderChart(r, w, s) {
            const ctx = document.getElementById('resultChart').getContext('2d');
            if(chartInstance) chartInstance.destroy();
            const total = r + w + s;
            const rP = Math.round((r/total)*100) || 0;
            const wP = Math.round((w/total)*100) || 0;
            const sP = Math.round((s/total)*100) || 0;

            chartInstance = new Chart(ctx, {
                type: 'pie',
                data: {
                    labels: [`${rP}% Correct`, `${wP}% Wrong`, `${sP}% Skip`],
                    datasets: [{ 
                        data: [r, w, s], 
                        backgroundColor: ['#2dd4bf', '#ef4444', '#facc15'], 
                        borderWidth: 2,
                        borderColor: '#ffffff'
                    }]
                },
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false, 
                    plugins: { 
                        legend: { position: 'bottom', labels: { usePointStyle: true } },
                        datalabels: {
                            color: '#fff',
                            font: { weight: 'bold', size: 14 },
                            formatter: (value, ctx) => {
                                if(value === 0) return '';
                                let sum = 0;
                                let dataArr = ctx.chart.data.datasets[0].data;
                                dataArr.map(data => { sum += data; });
                                let percentage = (value*100 / sum).toFixed(0)+"%";
                                return percentage;
                            }
                        }
                    }
                }
            });
        }

        function filterResult(filter) {
            const container = document.getElementById('resultDetails');
            container.innerHTML = '';
            for(let i=1; i<=currentExam.total; i++) {
                let u = userAnswers[i] || '-';
                let c = solutionKeys[i] || '-';
                let status = (u === c && u !== '-') ? 'right' : (u === '-') ? 'skipped' : 'wrong';
                if(filter !== 'all' && status !== filter) continue;
                let colorClass = status === 'right' ? 'bg-green-50 border-green-200' : status === 'wrong' ? 'bg-red-50 border-red-200' : 'bg-yellow-50 border-yellow-200';
                let div = document.createElement('div');
                div.className = `p-3 rounded-xl border ${colorClass} dark:bg-gray-800 dark:border-gray-700 flex justify-between items-center`;
                div.innerHTML = `<div><span class="font-bold text-gray-600 dark:text-gray-300">Q${i}</span></div><div class="text-right text-xs font-bold"><span class="text-gray-500">You: ${u}</span> | <span class="text-indigo-500">Ans: ${c}</span></div>`;
                container.appendChild(div);
            }
        }
        function saveHistory(score) {
            let hist = JSON.parse(localStorage.getItem('omr_history')) || [];
            hist.unshift({ name: currentExam.name, score: score, date: new Date().toLocaleDateString(), total: currentExam.total });
            localStorage.setItem('omr_history', JSON.stringify(hist));
        }
        function loadHistory() {
            const list = document.getElementById('historyContainer');
            list.innerHTML = '';
            let hist = JSON.parse(localStorage.getItem('omr_history')) || [];
            if(!hist.length) list.innerHTML = '<p class="text-center text-gray-400 mt-10">No records yet</p>';
            hist.forEach(h => {
                let percent = ((h.score / h.total) * 100).toFixed(1);
                let starCount = Math.round((percent / 100) * 5);
                let stars = '';
                for(let i=0; i<5; i++) stars += i < starCount ? '<i class="fa-solid fa-star"></i>' : '<i class="fa-regular fa-star opacity-50"></i>';
                let div = document.createElement('div');
                div.className = "bg-gradient-to-r from-orange-500 to-red-500 text-white p-5 rounded-3xl shadow-lg relative overflow-hidden transform transition hover:scale-[1.02] active:scale-95";
                div.innerHTML = `
                    <div class="absolute top-0 right-0 w-32 h-32 bg-white/10 rounded-full blur-3xl -mr-10 -mt-10 pointer-events-none"></div>
                    <div class="flex justify-between items-start relative z-10">
                         <div class="w-10 h-10 bg-white/20 backdrop-blur-md rounded-xl flex items-center justify-center text-lg"><i class="fa-solid fa-chart-pie"></i></div>
                         <div class="text-right">
                             <p class="text-[10px] font-bold uppercase opacity-80 tracking-wider">Live Score</p>
                             <h3 class="text-3xl font-black">${percent}%</h3>
                         </div>
                    </div>
                    <div class="mt-6 flex justify-between items-end relative z-10">
                        <div><p class="text-sm font-bold mb-1 truncate max-w-[150px]">${h.name}</p><div class="flex text-yellow-300 text-xs gap-1">${stars}</div></div>
                        <div class="text-right"><p class="text-[10px] opacity-70 mb-0.5">Rating</p><p class="text-xs font-bold bg-black/20 px-2 py-1 rounded-lg">Score: ${h.score}</p></div>
                    </div>
                `;
                list.appendChild(div);
            });
        }
        // Profile & Utils
        function previewImage(event, imgId, iconId) {
            const file = event.target.files[0];
            if(file){
                const reader = new FileReader();
                reader.onload = function(e) {
                    const img = document.getElementById(imgId);
                    img.src = e.target.result;
                    img.classList.remove('hidden');
                    if(iconId) document.getElementById(iconId).classList.add('hidden');
                }
                reader.readAsDataURL(file);
            }
        }
        function firstTimeSetup() {
            const name = document.getElementById('startName').value;
            const uClass = document.getElementById('startClass').value;
            const inst = document.getElementById('startInst').value;
            const img = document.getElementById('splashPreview').src;
            if(!name) return alert("Please enter your name");
            userProfile = { name, uClass, inst, img };
            localStorage.setItem('omr_user', JSON.stringify(userProfile));
            applyProfileData();
            goToView('homeView');
        }
        function applyProfileData() {
            document.getElementById('homeUserName').innerText = userProfile.name;
            document.getElementById('dashName').innerText = userProfile.name;
            document.getElementById('dashDetails').innerText = `${userProfile.uClass || ''} ${userProfile.inst ? '| ' + userProfile.inst : ''}`;
            if(userProfile.img && userProfile.img.startsWith('data')) {
                document.getElementById('homeProfileImg').src = userProfile.img;
                document.getElementById('dashProfileImg').src = userProfile.img;
            }
        }
        function toggleEditMode(showEdit) {
            document.getElementById('dashDisplayMode').classList.toggle('hidden', showEdit);
            document.getElementById('dashEditMode').classList.toggle('hidden', !showEdit);
            if(showEdit){
                document.getElementById('editName').value = userProfile.name;
                document.getElementById('editClass').value = userProfile.uClass;
                document.getElementById('editInst').value = userProfile.inst;
                document.getElementById('editPreview').src = userProfile.img.startsWith('data') ? userProfile.img : '';
            }
        }
        function saveProfileUpdate() {
            userProfile = { name: document.getElementById('editName').value, uClass: document.getElementById('editClass').value, inst: document.getElementById('editInst').value, img: document.getElementById('editPreview').src };
            localStorage.setItem('omr_user', JSON.stringify(userProfile));
            applyProfileData();
            toggleEditMode(false);
        }
        function updateDashboardUI(){ applyProfileData(); }
        function toggleDarkMode() {
            document.documentElement.classList.toggle('dark');
            localStorage.setItem('theme', document.documentElement.classList.contains('dark') ? 'dark' : 'light');
        }
    </script>
</body>
</html>
