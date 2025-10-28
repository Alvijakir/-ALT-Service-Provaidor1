 <html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>কারিগর খোঁজ - আপনার ঘরের সমস্যার সমাধান</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Firebase Imports (Required for standard app environment) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Initialize Firebase
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof _firebase_config !== 'undefined' ? JSON.parse(_firebase_config) : {};
        
        if (Object.keys(firebaseConfig).length > 0) {
            const app = initializeApp(firebaseConfig);
            const db = getFirestore(app);
            const auth = getAuth(app);
            setLogLevel('Debug');
            
            // Global variables for Firebase services
            window.db = db;
            window.auth = auth;
            
            // Global utility to handle authentication state
            window.setupAuth = () => {
                const loginScreen = document.getElementById('login-screen');
                const mainAppContent = document.getElementById('main-app-content');
                const authButton = document.getElementById('auth-button');
                const userIdDisplay = document.getElementById('user-id-display');
                const dashboardToggleBtn = document.getElementById('dashboard-toggle-btn');

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        // User is signed in. Show main content.
                        mainAppContent.classList.remove('hidden');
                        loginScreen.classList.add('hidden');
                        dashboardToggleBtn.classList.remove('hidden'); // Show toggle button
                        
                        authButton.textContent = 'লগআউট';
                        authButton.onclick = () => window.handleLogout(auth);
                        userIdDisplay.textContent = ইউজার আইডি: ${user.uid};
                        window.toggleView('customer'); // Default to customer view after login
                    } else {
                        // No user is signed in. Show login screen.
                        mainAppContent.classList.add('hidden');
                        loginScreen.classList.remove('hidden');
                        dashboardToggleBtn.classList.add('hidden'); // Hide toggle button
                        
                        authButton.textContent = 'লগইন';
                        authButton.onclick = () => window.handleLogin(auth);
                        userIdDisplay.textContent = ইউজার আইডি: অতিথি;

                        // Attempt automatic sign-in only if necessary for the environment
                        if (typeof __initial_auth_token !== 'undefined' && !user) {
                            try {
                                await signInWithCustomToken(auth, __initial_auth_token);
                            } catch (error) {
                                console.error("Custom Token Sign-In Failed, proceeding to anonymous sign-in:", error);
                                await signInAnonymously(auth);
                            }
                        } else if (!user) {
                            // Fallback to anonymous sign-in if no token is available
                            await signInAnonymously(auth);
                        }
                    }
                });
            };

            // Public functions for login/logout (simplified for this environment)
            window.handleLogin = async (auth) => {
                // In a real app, this would use email/password. Here we use anonymous sign-in to demonstrate the transition.
                try {
                    await signInAnonymously(auth);
                } catch (error) {
                    window.showMessage("লগইন ত্রুটি", লগইন ব্যর্থ হয়েছে: ${error.message}, false);
                }
            };

            window.handleLogout = async (auth) => {
                try {
                    await signOut(auth);
                    window.showMessage(" alt sievice , "আপনি সফলভাবে লগআউট করেছেন।", true);
                } catch (error) {
                    window.showMessage("লগআউট ত্রুটি", লগআউট ব্যর্থ হয়েছে: ${error.message}, false);
                }
            };

            window.setupAuth(); // Start listening for auth changes
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f9fb;
        }
        .container-shadow {
            box-shadow: 0 10px 25px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        /* Custom modal styles */
        .modal {
            transition: opacity 0.3s ease, transform 0.3s ease;
        }
        .modal-hidden {
            opacity: 0;
            pointer-events: none;
            transform: translateY(-20px);
        }
        .status-toggle:checked {
            background-color: #10B981; /* green-500 */
        }
        .status-toggle {
            transition: all 0.3s ease;
        }
        .status-toggle:checked + .slider:before {
            transform: translateX(1.5rem);
            border-color: #10B981;
        }
    </style>
</head>
<body>

    <!-- Header Section -->
    <header class="bg-blue-600 text-white shadow-lg p-4">
        <div class="max-w-4xl mx-auto flex justify-between items-center">
            <h1 class="text-3xl font-bold tracking-tight">
                কারিগর খোঁজ
            </h1>
            <div class="flex items-center space-x-3 sm:space-x-4">
                <span id="user-id-display" class="text-xs sm:text-sm bg-blue-700 px-3 py-1 rounded-full hidden sm:block"></span>

                <!-- NEW: Dashboard Toggle Button -->
                <button id="dashboard-toggle-btn" onclick="toggleMode()"
                    class="hidden bg-yellow-400 text-gray-900 px-3 py-1 rounded-full font-semibold hover:bg-yellow-500 transition duration-150 text-sm sm:text-base">
                    মেকানিক ড্যাশবোর্ড
                </button>

                <button id="auth-button"
                    class="bg-white text-blue-600 px-3 py-1 rounded-full font-semibold hover:bg-blue-100 transition duration-150">
                    লগইন
                </button>
            </div>
        </div>
    </header>

    <!-- 1. LOGIN SCREEN (Initially Visible if not authenticated) -->
    <div id="login-screen" class="min-h-screen flex items-center justify-center p-4">
        <div class="bg-white container-shadow rounded-2xl p-8 max-w-md w-full text-center border-t-8 border-blue-600">
            <h2 class="text-3xl font-bold text-gray-900 mb-6">স্বাগতম! লগইন করুন</h2>
            <p class="text-gray-500 mb-8">আপনার বাড়ির কাজের জন্য বিশ্বস্ত মেকানিক খুঁজুন।</p>

            <!-- Simulated Login Form -->
            <form id="login-form" onsubmit="event.preventDefault(); window.handleLogin(window.auth);">
                <input type="email" placeholder="ইমেল ঠিকানা"
                       class="w-full p-4 mb-4 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 text-gray-700 placeholder-gray-400 text-base">
                <input type="password" placeholder 11223311
                       class="w-full p-4 mb-6 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 text-gray-700 placeholder-gray-400 text-base">

                <button type="submit"
                        class="w-full bg-blue-600 text-white px-8 py-3 rounded-xl font-bold text-lg hover:bg-blue-700 transition duration-300 ease-in-out shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-500 focus:ring-opacity-50">
                    লগইন/সাইনআপ করুন (অতিথি হিসেবে প্রবেশ)
                </button>
            </form>
            <p class="mt-4 text-sm text-gray-500">
                এই ডেমো-তে, লগইন বাটনটি আপনাকে অতিথি হিসেবে সাইন ইন করবে।
            </p>
        </div>
    </div>


    <!-- 2. MAIN APPLICATION CONTENT CONTAINER -->
    <div id="main-app-content" class="hidden">
        
        <!-- CUSTOMER VIEW -->
        <div id="customer-view" class="max-w-4xl mx-auto p-4 sm:p-8">
            <div class="text-center my-10">
                <h2 class="text-4xl sm:text-5xl font-extrabold text-gray-900 mb-3">
                    আপনার ঘরের সমস্যার সমাধান এখন হাতের মুঠোয়
                </h2>
                <p class="text-gray-500 text-lg">
                    দক্ষ ও যাচাইকৃত ইলেকট্রনিক মেকানিক খুঁজুন
                </p>
            </div>

            <div class="bg-white container-shadow rounded-2xl p-6 sm:p-10 mb-12">
                <!-- Search Form -->
                <div class="flex flex-col gap-4">
                    <input id="problem-description" type="text" placeholder="আপনার সমস্যার বর্ণনা দিন (যেমন: এসি ঠান্ডা হচ্ছে না, ফ্যান চলছে না)"
                           class="flex-1 p-4 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 text-gray-700 placeholder-gray-400 text-base">

                    <div class="flex flex-col md:flex-row gap-4">
                        <!-- Gemini API Diagnosis Button -->
                        <button id="diagnose-button"
                                class="flex-1 bg-green-500 text-white px-6 py-3 rounded-xl font-bold text-lg hover:bg-green-600 transition duration-300 ease-in-out shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-green-500 focus:ring-opacity-50 flex items-center justify-center">
                            <span id="diagnose-button-text">✨ সমস্যার এআই বিশ্লেষণ</span>
                            <svg id="diagnose-loading-spinner" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white hidden" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                            </svg>
                        </button>

                        <!-- Original Search Button -->
                        <button id="search-button"
                                class="flex-1 bg-blue-600 text-white px-8 py-3 rounded-xl font-bold text-lg hover:bg-blue-700 transition duration-300 ease-in-out shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-500 focus:ring-opacity-50 flex items-center justify-center">
                            <span id="search-button-text">মেকানিক খুঁজুন</span>
                            <svg id="search-loading-spinner" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white hidden" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                            </svg>
                        </button>
                    </div>
                </div>
            </div>

            <!-- Service Categories Section -->
            <h3 class="text-2xl font-bold text-gray-800 mb-6">জনপ্রিয় সার্ভিস ক্যাটাগরি</h3>

            <div class="grid grid-cols-2 md:grid-cols-4 gap-6">
                <!-- AC Repair -->
                <div onclick="selectCategory('এসি সার্ভিসিং')" class="service-card cursor-pointer bg-white container-shadow rounded-xl p-5 text-center transition duration-200 hover:scale-[1.03] hover:shadow-xl border-t-4 border-blue-500">
                    <svg class="h-10 w-10 mx-auto text-blue-500 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.942 3.313.805 2.583 2.338a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.942 1.543-.805 3.313-2.338 2.583a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.942-3.313-.805-2.583-2.338a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.942-1.543.805-3.313 2.338-2.583a1.724 1.724 0 002.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                    <p class="text-lg font-semibold text-gray-700">এসি সার্ভিসিং</p>
                </div>

                <!-- TV Repair -->
                <div onclick="selectCategory('টিভি মেরামত')" class="service-card cursor-pointer bg-white container-shadow rounded-xl p-5 text-center transition duration-200 hover:scale-[1.03] hover:shadow-xl border-t-4 border-green-500">
                    <svg class="h-10 w-10 mx-auto text-green-500 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.75 17L9 20l-1 1h8l-1-1v-3.25m-7.5-3.333V17l8.5-4.5-8.5-4.5v3.333a5.75 5.75 0 01-5.75 5.75H12"></path></svg>
                    <p class="text-lg font-semibold text-gray-700">টিভি মেরামত</p>
                </div>

                <!-- Fan/Light Fitting -->
                <div onclick="selectCategory('ফ্যান/লাইট ওয়্যারিং')" class="service-card cursor-pointer bg-white container-shadow rounded-xl p-5 text-center transition duration-200 hover:scale-[1.03] hover:shadow-xl border-t-4 border-yellow-500">
                    <svg class="h-10 w-10 mx-auto text-yellow-500 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"></path></svg>
                    <p class="text-lg font-semibold text-gray-700">ফ্যান/লাইট ফিটিং</p>
                </div>

                <!-- General Wiring -->
                <div onclick="selectCategory('সাধারণ ওয়্যারিং')" class="service-card cursor-pointer bg-white container-shadow rounded-xl p-5 text-center transition duration-200 hover:scale-[1.03] hover:shadow-xl border-t-4 border-red-500">
                    <svg class="h-10 w-10 mx-auto text-red-500 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
                    <p class="text-lg font-semibold text-gray-700">সাধারণ ওয়্যারিং</p>
                </div>
            </div>
        </div>
        
        <!-- MECHANIC DASHBOARD VIEW -->
        <div id="mechanic-dashboard" class="hidden max-w-4xl mx-auto p-4 sm:p-8">
            <h2 class="text-4xl font-extrabold text-gray-900 mb-6 border-b-2 pb-2">মেকানিক ড্যাশবোর্ড</h2>

            <!-- Mechanic Info Card & Status Toggle -->
            <div class="bg-white container-shadow rounded-2xl p-6 mb-8 flex flex-col sm:flex-row justify-between items-center">
                <div class="mb-4 sm:mb-0">
                    <h3 class="text-2xl font-bold text-blue-600">স্বাগতম, মেকানিক মোস্তাফিজ!</h3>
                    <p class="text-gray-600">রেটিং: <span class="text-yellow-500 font-bold">৪.৮</span> (১৩০টি কাজ)</p>
                </div>
                
                <div class="flex items-center space-x-3">
                    <label for="status-toggle" class="text-lg font-semibold text-gray-700">কাজের জন্য প্রস্তুত:</label>
                    <label class="relative inline-flex items-center cursor-pointer">
                        <input type="checkbox" id="status-toggle" class="sr-only peer status-toggle" checked>
                        <div class="slider w-14 h-8 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[4px] after:left-[4px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all"></div>
                        <span id="availability-status" class="ml-3 text-sm font-medium text-green-600">উপলব্ধ</span>
                    </label>
                </div>
            </div>

            <!-- KPI Cards -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
                <!-- Total Jobs -->
                <div class="bg-white container-shadow rounded-xl p-6 border-l-4 border-blue-500">
                    <p class="text-sm font-medium text-blue-600">মোট কাজ সম্পন্ন</p>
                    <p class="text-3xl font-bold text-gray-900 mt-1">১৭৫</p>
                </div>
                
                <!-- Pending Jobs -->
                <div class="bg-white container-shadow rounded-xl p-6 border-l-4 border-yellow-500">
                    <p class="text-sm font-medium text-yellow-600">নতুন অনুরোধ</p>
                    <p class="text-3xl font-bold text-gray-900 mt-1" id="pending-jobs-count">৪</p>
                </div>

                <!-- Earnings -->
                <div class="bg-white container-shadow rounded-xl p-6 border-l-4 border-green-500">
                    <p class="text-sm font-medium text-green-600">মোট উপার্জন (এই মাসে)</p>
                    <p class="text-3xl font-bold text-gray-900 mt-1">৳২৫,০০০</p>
                </div>
            </div>

            <!-- New Job Requests List -->
            <h3 class="text-2xl font-bold text-gray-800 mb-4">নতুন কাজের অনুরোধ</h3>
            <div id="job-requests-list" class="space-y-4">
                <!-- Jobs will be rendered here by JS -->
            </div>
            
        </div>
    </div>


    <!-- Success/Message Modal -->
    <div id="modal" class="modal modal-hidden fixed inset-0 bg-gray-900 bg-opacity-75 flex items-center justify-center p-4 z-50">
        <div id="modal-content" class="bg-white rounded-xl p-8 max-w-lg w-full shadow-2xl transform transition duration-300 ease-out">
            <h3 id="modal-title" class="text-2xl font-bold text-blue-600 mb-4"></h3>
            <div id="modal-message" class="text-gray-700 mb-6"></div>
            <button id="close-modal-button" class="w-full bg-blue-600 text-white py-3 rounded-lg font-semibold hover:bg-blue-700 transition duration-300">
                 <lock>mun >
            </button>
        </div>
    </div>

    <script>
        // Global variables for API key and URL
        const apiKey = "";
        const apiUrl = https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey};
