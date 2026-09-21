<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rifa Áurea - Sorteo de iPad</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .number-btn {
            transition: all 0.2s ease-in-out;
            position: relative;
        }
        .number-btn:hover {
            transform: scale(1.05);
            z-index: 10;
        }
        .number-btn:active {
            transform: scale(0.95);
        }
        /* Tooltip style para mostrar nombres rápido al pasar el ratón */
        .tooltip {
            visibility: hidden;
            background-color: #333;
            color: #fff;
            text-align: center;
            border-radius: 6px;
            padding: 5px;
            position: absolute;
            z-index: 20;
            bottom: 125%; /* Aparece arriba del botón */
            left: 50%;
            margin-left: -60px;
            opacity: 0;
            transition: opacity 0.3s;
            width: max-content;
            max-width: 150px;
            font-size: 0.75rem;
            pointer-events: none;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
        .tooltip::after {
            content: "";
            position: absolute;
            top: 100%;
            left: 50%;
            margin-left: -5px;
            border-width: 5px;
            border-style: solid;
            border-color: #333 transparent transparent transparent;
        }
        .number-btn.occupied:hover .tooltip {
            visibility: visible;
            opacity: 1;
        }
        
        /* Modal animations */
        .modal-enter {
            opacity: 0;
            transform: scale(0.9);
        }
        .modal-enter-active {
            opacity: 1;
            transform: scale(1);
            transition: opacity 300ms, transform 300ms;
        }
        .modal-exit {
            opacity: 1;
            transform: scale(1);
        }
        .modal-exit-active {
            opacity: 0;
            transform: scale(0.9);
            transition: opacity 300ms, transform 300ms;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-indigo-50 via-white to-purple-100 min-h-screen font-sans text-gray-800 relative">
    <div class="max-w-6xl mx-auto p-4 sm:p-6 lg:p-8 pb-20">
        
        <!-- Header -->
        <header class="text-center mb-10 relative">
            <div class="inline-block mb-3 px-5 py-1.5 rounded-full bg-purple-100 text-purple-700 font-bold text-sm tracking-wider border border-purple-200 shadow-sm uppercase">
                ✨ Gran Sorteo Especial
            </div>
            <h1 class="text-5xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-blue-600 to-purple-600 tracking-tight mb-2 drop-shadow-sm">Rifa Áurea</h1>
            <h2 class="text-3xl font-bold text-gray-800 mb-3">¡Participa por un <span class="text-purple-600">iPad</span>!</h2>
            <p class="text-gray-500 font-medium text-lg">Controla los 180 números de tu sorteo y los datos de venta.</p>
        </header>

        <!-- Control Panel (Stats & Actions) -->
        <div class="bg-white rounded-xl shadow-lg border-t-4 border-purple-500 p-6 mb-8 flex flex-col sm:flex-row justify-between items-center gap-4">
            <div class="flex gap-6 text-center">
                <div>
                    <p class="text-sm text-gray-500 uppercase tracking-wider font-semibold">Total</p>
                    <p class="text-2xl font-bold text-gray-800">180</p>
                </div>
                <div>
                    <p class="text-sm text-green-600 uppercase tracking-wider font-semibold">Disponibles</p>
                    <p id="available-count" class="text-2xl font-bold text-green-700">180</p>
                </div>
                <div>
                    <p class="text-sm text-red-500 uppercase tracking-wider font-semibold">Ocupados</p>
                    <p id="occupied-count" class="text-2xl font-bold text-red-600">0</p>
                </div>
            </div>
            
            <div class="flex gap-3">
                <!-- Botón para ver la lista de ocupados -->
                <button onclick="toggleListView()" class="px-5 py-2.5 bg-gradient-to-r from-blue-500 to-purple-600 hover:from-blue-600 hover:to-purple-700 text-white rounded-lg font-medium transition-all shadow-md hover:shadow-lg focus:outline-none focus:ring-2 focus:ring-purple-400">
                    Ver Lista Detallada
                </button>
            </div>
        </div>

        <!-- Estadísticas por Vendedor -->
        <div class="bg-white rounded-xl shadow-md border-t-4 border-blue-400 p-4 sm:p-6 mb-8">
            <h3 class="text-sm text-gray-500 uppercase tracking-wider font-semibold mb-4 text-center">Números vendidos por persona</h3>
            <div id="seller-stats-container" class="flex flex-wrap justify-center gap-3 sm:gap-4">
                <!-- Las tarjetas de vendedores se generarán aquí -->
            </div>
        </div>

        <!-- Legend -->
        <div class="flex justify-center gap-6 mb-6 text-sm">
            <div class="flex items-center gap-2">
                <div class="w-6 h-6 rounded bg-white border-2 border-gray-300"></div>
                <span>Disponible</span>
            </div>
            <div class="flex items-center gap-2">
                <div class="w-6 h-6 rounded bg-red-500 border-2 border-red-600 shadow-inner"></div>
                <span>Ocupado (Pasa el ratón para ver datos)</span>
            </div>
        </div>

        <!-- Number Grid View -->
        <div id="grid-view" class="bg-white rounded-xl shadow-md p-4 sm:p-6 transition-all duration-300">
            
            <div class="flex flex-wrap justify-center gap-2 mb-6 border-b border-gray-100 pb-4">
                <button onclick="setFilter('all')" id="btn-filter-all" class="px-5 py-2 rounded-full text-sm font-bold bg-gray-800 text-white transition-colors shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-400">
                    Mostrar Todos
                </button>
                <button onclick="setFilter('available')" id="btn-filter-available" class="px-5 py-2 rounded-full text-sm font-bold bg-gray-100 text-gray-600 hover:bg-green-100 hover:text-green-800 transition-colors focus:outline-none focus:ring-2 focus:ring-green-400">
                    Solo Disponibles
                </button>
                <button onclick="setFilter('occupied')" id="btn-filter-occupied" class="px-5 py-2 rounded-full text-sm font-bold bg-gray-100 text-gray-600 hover:bg-red-100 hover:text-red-800 transition-colors focus:outline-none focus:ring-2 focus:ring-red-400">
                    Solo Ocupados
                </button>
            </div>

            <div id="number-grid" class="grid grid-cols-5 sm:grid-cols-10 md:grid-cols-12 lg:grid-cols-15 gap-2">
                <!-- Los números se generarán aquí -->
            </div>

            <div id="text-summary-container" class="hidden mt-6 p-4 bg-purple-50 rounded-lg border border-purple-100 shadow-inner">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-2 gap-2">
                    <h4 id="text-summary-title" class="font-bold text-purple-800 text-sm">Resumen de Números</h4>
                    <button id="btn-copy-wa" onclick="copySummaryText()" class="text-xs bg-purple-200 text-purple-800 px-3 py-1.5 rounded-lg hover:bg-purple-300 font-semibold transition shadow-sm">
                        Copiar para WhatsApp
                    </button>
                </div>
                <p id="text-summary-content" class="text-sm text-purple-900 leading-relaxed break-words font-mono"></p>
            </div>
        </div>

        <!-- List View (Hidden by default) -->
        <div id="list-view" class="hidden bg-white rounded-xl shadow-lg overflow-hidden transition-all duration-300 border-t-4 border-purple-500">
            <div class="px-6 py-4 border-b border-gray-200 flex justify-between items-center bg-gray-50">
                <h2 class="text-xl font-bold text-gray-800">Números Vendidos</h2>
                <button onclick="toggleListView()" class="text-sm text-purple-600 hover:text-purple-800 font-medium">Volver a la Cuadrícula</button>
            </div>
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Número</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Comprador</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Vendedor</th>
                            <th scope="col" class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">Acción</th>
                        </tr>
                    </thead>
                    <tbody id="list-body" class="bg-white divide-y divide-gray-200">
                        <!-- Las filas se generarán aquí -->
                    </tbody>
                </table>
                <div id="empty-list-msg" class="text-center py-8 text-gray-500 hidden">
                    Aún no hay números vendidos.
                </div>
            </div>
        </div>

    </div>

    <!-- Modal for Data Entry -->
    <div id="data-modal" class="fixed inset-0 z-50 hidden flex items-center justify-center bg-black bg-opacity-60 backdrop-blur-sm">
        <div class="bg-white rounded-xl shadow-2xl w-11/12 max-w-md overflow-hidden transform transition-all">
            
            <!-- Modal Header -->
            <div class="bg-gradient-to-r from-blue-600 to-purple-600 px-6 py-4 flex justify-between items-center">
                <h3 class="text-lg font-bold text-white flex items-center gap-2">
                    Asignar Número <span id="modal-number-display" class="bg-white text-purple-700 px-2 py-0.5 rounded-md text-xl shadow-sm"></span>
                </h3>
                <button onclick="closeModal()" class="text-blue-100 hover:text-white transition-colors focus:outline-none">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                </button>
            </div>

            <!-- Modal Body -->
            <div class="p-6">
                <input type="hidden" id="modal-number-index">
                
                <div class="mb-5">
                    <label for="buyer-name" class="block text-sm font-medium text-gray-700 mb-1">Nombre de quien COMPRA el boleto</label>
                    <input type="text" id="buyer-name" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-shadow" placeholder="Ej. Juan Pérez" autocomplete="off">
                </div>

                <div class="mb-6">
                    <label for="seller-name" class="block text-sm font-medium text-gray-700 mb-1">Nombre de quien VENDE el boleto</label>
                    <select id="seller-name" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-shadow">
                        <option value="">-- Seleccionar Vendedor --</option>
                        <option value="Barbara">Barbara</option>
                        <option value="Ashley">Ashley</option>
                        <option value="Kamila">Kamila</option>
                        <option value="Valeria">Valeria</option>
                        <option value="Juan">Juan</option>
                        <option value="Emmanuel">Emmanuel</option>
                        <option value="Daniela">Daniela</option>
                        <option value="Santiago">Santiago</option>
                        <option value="Mahia">Mahia</option>
                    </select>
                </div>

                <!-- Mensaje de error personalizado -->
                <p id="modal-error-msg" class="text-red-500 text-sm mb-3 hidden font-medium">Por favor ingresa al menos el nombre del comprador.</p>

                <!-- Modal Footer -->
                <div class="flex justify-end gap-3 pt-2">
                    <button type="button" onclick="closeModal()" class="px-4 py-2 text-gray-700 bg-gray-100 hover:bg-gray-200 rounded-lg font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-gray-300">
                        Cancelar
                    </button>
                    <!-- Botón para liberar el número (solo visible si ya estaba ocupado) -->
                    <button type="button" id="btn-liberar" onclick="freeNumberFromModal()" class="hidden px-4 py-2 text-red-700 bg-red-100 hover:bg-red-200 rounded-lg font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-red-400">
                        Liberar Número
                    </button>
                    <button type="button" onclick="saveNumberData()" class="px-5 py-2 text-white bg-gradient-to-r from-blue-500 to-purple-600 hover:from-blue-600 hover:to-purple-700 rounded-lg font-medium shadow-md transition-all focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-offset-2">
                        Guardar
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal de Confirmación para Liberar -->
    <div id="confirm-modal" class="fixed inset-0 z-50 hidden flex items-center justify-center bg-black bg-opacity-60 backdrop-blur-sm">
        <div class="bg-white rounded-xl shadow-2xl p-6 max-w-sm w-11/12 text-center transform transition-all">
            <h3 class="text-xl font-bold text-gray-800 mb-3">¿Liberar número?</h3>
            <p class="text-gray-600 mb-6 text-sm">Se borrarán los datos del comprador y vendedor para que vuelva a estar disponible. Esta acción no se puede deshacer.</p>
            <div class="flex justify-center gap-3">
                <button onclick="closeConfirmModal()" class="px-4 py-2 text-gray-700 bg-gray-100 hover:bg-gray-200 rounded-lg font-medium transition-colors">Cancelar</button>
                <button id="btn-confirm-action" class="px-4 py-2 text-white bg-red-600 hover:bg-red-700 rounded-lg font-medium shadow-md transition-all">Sí, liberar</button>
            </div>
        </div>
    </div>

    <!-- JavaScript Logic con Sincronización en la Nube -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const TOTAL_NUMBERS = 180;
        let raffleData = []; 
        let currentFilter = 'all';
        let actionToConfirm = null;

        // Variables de Firebase (El entorno inyecta la configuración automáticamente)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const rawConfig = typeof __firebase_config !== 'undefined' ? __firebase_config : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        let app, db, auth, user, docRef;
        let isFirebaseActive = false;

        async function initApp() {
            if (rawConfig) {
                try {
                    const firebaseConfig = JSON.parse(rawConfig);
                    app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }
                    user = auth.currentUser;
                    
                    if (user) {
                        isFirebaseActive = true;
                        // Ruta estricta para la base de datos colaborativa
                        docRef = doc(db, 'artifacts', appId, 'public', 'data', 'raffle', 'state');
                        
                        // Sincronización en tiempo real 
                        onSnapshot(docRef, (docSnap) => {
                            if (docSnap.exists()) {
                                const data = docSnap.data();
                                for (let i = 0; i < TOTAL_NUMBERS; i++) {
                                    raffleData[i] = data.tickets && data.tickets[i.toString()] 
                                        ? data.tickets[i.toString()] 
                                        : { isAvailable: true, buyer: '', seller: '' };
                                }
                                renderGrid();
                            } else {
                                initializeFirestoreDoc();
                            }
                        }, (error) => {
                            console.error("Error de sincronización en tiempo real:", error);
                        });
                    }
                } catch (e) {
                    console.error("Error iniciando Firebase, usando modo local:", e);
                    initLocalState();
                }
            } else {
                initLocalState();
            }
        }

        async function initializeFirestoreDoc() {
            if (!user) return;
            const initialTickets = {};
            for (let i = 0; i < TOTAL_NUMBERS; i++) {
                initialTickets[i.toString()] = { isAvailable: true, buyer: '', seller: '' };
            }
            try {
                // Verificamos antes de crear para evitar que dos personas lo sobreescriban a la vez
                const currentSnap = await getDoc(docRef);
                if (!currentSnap.exists()) {
                    await setDoc(docRef, { tickets: initialTickets });
                }
            } catch (e) {
                console.error("Error al crear el documento base:", e);
            }
        }

        function initLocalState() {
            const savedData = localStorage.getItem('rifaDataV2');
            if (savedData) {
                try {
                    raffleData = JSON.parse(savedData);
                    if(raffleData.length !== TOTAL_NUMBERS) fillEmptyData();
                } catch (e) { fillEmptyData(); }
            } else {
                fillEmptyData();
            }
            renderGrid();
        }

        function fillEmptyData() {
            raffleData = Array(TOTAL_NUMBERS).fill(null).map(() => ({ isAvailable: true, buyer: '', seller: '' }));
        }

        // Exportamos las funciones al entorno global para que HTML las pueda usar
        window.setFilter = setFilter;
        window.copySummaryText = copySummaryText;
        window.openModal = openModal;
        window.closeModal = closeModal;
        window.saveNumberData = saveNumberData;
        window.freeNumberFromModal = freeNumberFromModal;
        window.freeNumber = freeNumber;
        window.toggleListView = toggleListView;
        window.closeConfirmModal = closeConfirmModal;

        function setFilter(filter) {
            currentFilter = filter;
            const btnAll = document.getElementById('btn-filter-all');
            const btnAvail = document.getElementById('btn-filter-available');
            const btnOcc = document.getElementById('btn-filter-occupied');
            
            const inactiveClass = "px-5 py-2 rounded-full text-sm font-bold bg-gray-100 text-gray-600 transition-colors focus:outline-none focus:ring-2";
            btnAll.className = inactiveClass + " hover:bg-gray-200 hover:text-gray-800";
            btnAvail.className = inactiveClass + " hover:bg-green-100 hover:text-green-800";
            btnOcc.className = inactiveClass + " hover:bg-red-100 hover:text-red-800";

            if (filter === 'all') btnAll.className = "px-5 py-2 rounded-full text-sm font-bold bg-gray-800 text-white transition-colors shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-400";
            else if (filter === 'available') btnAvail.className = "px-5 py-2 rounded-full text-sm font-bold bg-green-600 text-white transition-colors shadow-sm focus:outline-none focus:ring-2 focus:ring-green-400";
            else if (filter === 'occupied') btnOcc.className = "px-5 py-2 rounded-full text-sm font-bold bg-red-600 text-white transition-colors shadow-sm focus:outline-none focus:ring-2 focus:ring-red-400";
            
            renderGrid();
        }

        function copySummaryText() {
            const textToCopy = document.getElementById('text-summary-content').innerText;
            const btn = document.getElementById('btn-copy-wa');
            const textArea = document.createElement("textarea");
            textArea.value = textToCopy;
            document.body.appendChild(textArea);
            textArea.select();
            try {
                document.execCommand('copy');
                const originalText = btn.innerText;
                btn.innerText = "¡Copiado! ✓";
                btn.classList.replace('bg-purple-200', 'bg-green-200');
                btn.classList.replace('text-purple-800', 'text-green-800');
                setTimeout(() => {
                    btn.innerText = originalText;
                    btn.classList.replace('bg-green-200', 'bg-purple-200');
                    btn.classList.replace('text-green-800', 'text-purple-800');
                }, 2000);
            } catch (err) {
                console.error("Error al copiar texto");
            }
            document.body.removeChild(textArea);
        }

        function renderGrid() {
            if (raffleData.length === 0) return; // Evitar render si aún no carga de Firebase
            const grid = document.getElementById('number-grid');
            grid.innerHTML = ''; 

            let availableNumbersList = [];
            let occupiedNumbersList = [];

            for (let i = 0; i < TOTAL_NUMBERS; i++) {
                const number = i + 1;
                const data = raffleData[i];
                
                if (data.isAvailable) availableNumbersList.push(number);
                else occupiedNumbersList.push(number);

                if (currentFilter === 'available' && !data.isAvailable) continue;
                if (currentFilter === 'occupied' && data.isAvailable) continue;

                const button = document.createElement('button');
                let baseClasses = 'number-btn w-full aspect-square rounded-lg font-bold text-lg sm:text-base flex items-center justify-center border-2 focus:outline-none focus:ring-2 focus:ring-offset-1 focus:ring-purple-500 cursor-pointer shadow-sm relative';
                
                if (data.isAvailable) {
                    button.className = `${baseClasses} bg-white border-gray-200 text-gray-700 hover:bg-purple-50 hover:border-purple-300`;
                    button.innerText = number;
                } else {
                    button.className = `${baseClasses} occupied bg-red-500 border-red-600 text-white shadow-inner`;
                    button.innerHTML = `${number} 
                        <span class="tooltip">
                            <span class="block font-bold border-b border-gray-500 mb-1 pb-1">Boleto #${number}</span>
                            <span class="block text-left"><span class="text-gray-300">Compró:</span> ${data.buyer || 'N/A'}</span>
                            <span class="block text-left"><span class="text-gray-300">Vendió:</span> ${data.seller || 'N/A'}</span>
                        </span>`;
                }

                button.onclick = () => openModal(i);
                grid.appendChild(button);
            }
            
            const summaryContainer = document.getElementById('text-summary-container');
            const summaryTitle = document.getElementById('text-summary-title');
            const summaryContent = document.getElementById('text-summary-content');
            
            if (currentFilter === 'all') {
                summaryContainer.classList.add('hidden');
            } else {
                summaryContainer.classList.remove('hidden');
                if (currentFilter === 'available') {
                    summaryTitle.innerText = `Números Disponibles (${availableNumbersList.length})`;
                    summaryContent.innerText = availableNumbersList.join(', ') || 'No hay números disponibles.';
                } else if (currentFilter === 'occupied') {
                    summaryTitle.innerText = `Números Ocupados (${occupiedNumbersList.length})`;
                    summaryContent.innerText = occupiedNumbersList.join(', ') || 'No hay números ocupados.';
                }
            }

            updateStats();
            renderList();
        }

        function openModal(index) {
            const data = raffleData[index];
            const number = index + 1;
            
            document.getElementById('modal-number-display').innerText = number;
            document.getElementById('modal-number-index').value = index;
            document.getElementById('modal-error-msg').classList.add('hidden'); // Ocultar error previo
            
            const buyerInput = document.getElementById('buyer-name');
            const sellerInput = document.getElementById('seller-name');
            const btnLiberar = document.getElementById('btn-liberar');
            
            buyerInput.value = data.buyer;
            sellerInput.value = data.seller;
            
            if (data.isAvailable) {
                btnLiberar.classList.add('hidden');
                const lastSeller = localStorage.getItem('lastSeller');
                if(lastSeller && !sellerInput.value) {
                    sellerInput.value = lastSeller;
                }
            } else {
                btnLiberar.classList.remove('hidden');
            }

            document.getElementById('data-modal').classList.remove('hidden');
            setTimeout(() => buyerInput.focus(), 50);
        }

        function closeModal() {
            document.getElementById('data-modal').classList.add('hidden');
        }

        async function saveNumberData() {
            const index = parseInt(document.getElementById('modal-number-index').value);
            const buyer = document.getElementById('buyer-name').value.trim();
            const seller = document.getElementById('seller-name').value.trim();
            const errorMsg = document.getElementById('modal-error-msg');

            // Validación mejorada sin usar el alert() del navegador
            if (!buyer && !seller) {
                errorMsg.classList.remove('hidden');
                return;
            }
            errorMsg.classList.add('hidden');

            const ticketData = { isAvailable: false, buyer: buyer, seller: seller };
            
            if(seller) localStorage.setItem('lastSeller', seller);
            closeModal();
            await syncState(index, ticketData);
        }

        function freeNumberFromModal() {
            const index = parseInt(document.getElementById('modal-number-index').value);
            closeModal();
            promptConfirm(index);
        }
        
        function freeNumber(index) {
            promptConfirm(index);
        }

        function promptConfirm(index) {
            actionToConfirm = index;
            document.getElementById('confirm-modal').classList.remove('hidden');
            document.getElementById('btn-confirm-action').onclick = async () => {
                closeConfirmModal();
                const ticketData = { isAvailable: true, buyer: '', seller: '' };
                await syncState(actionToConfirm, ticketData);
            };
        }

        function closeConfirmModal() {
            document.getElementById('confirm-modal').classList.add('hidden');
            actionToConfirm = null;
        }

        async function syncState(index, ticketData) {
            // Actualización optimista local (para que el usuario lo vea de inmediato)
            raffleData[index] = ticketData;
            renderGrid(); 

            // Subida a la nube
            if (isFirebaseActive && user && docRef) {
                try {
                    const updateObj = {};
                    updateObj[`tickets.${index}`] = ticketData;
                    await updateDoc(docRef, updateObj);
                } catch (error) {
                    console.error("Error guardando en la nube:", error);
                }
            } else {
                // Fallback si no hay conexión
                localStorage.setItem('rifaDataV2', JSON.stringify(raffleData));
            }
        }

        function toggleListView() {
            const gridView = document.getElementById('grid-view');
            const listView = document.getElementById('list-view');
            if (listView.classList.contains('hidden')) {
                gridView.classList.add('hidden');
                listView.classList.remove('hidden');
            } else {
                listView.classList.add('hidden');
                gridView.classList.remove('hidden');
            }
        }

        function renderList() {
            const listBody = document.getElementById('list-body');
            const emptyMsg = document.getElementById('empty-list-msg');
            listBody.innerHTML = '';
            
            let hasSoldItems = false;
            for (let i = 0; i < TOTAL_NUMBERS; i++) {
                const data = raffleData[i];
                if (!data.isAvailable) {
                    hasSoldItems = true;
                    const number = i + 1;
                    const row = document.createElement('tr');
                    row.className = "hover:bg-gray-50 transition-colors";
                    row.innerHTML = `
                        <td class="px-6 py-4 whitespace-nowrap"><span class="px-2 inline-flex text-sm leading-5 font-bold rounded-full bg-red-100 text-red-800 border border-red-200">#${number}</span></td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900 font-medium">${data.buyer || '<span class="text-gray-400 italic">No especificado</span>'}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${data.seller || '<span class="text-gray-400 italic">No especificado</span>'}</td>
                        <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                            <button onclick="openModal(${i})" class="text-purple-600 hover:text-purple-900 mr-3">Editar</button>
                            <button onclick="freeNumber(${i})" class="text-red-600 hover:text-red-900">Liberar</button>
                        </td>
                    `;
                    listBody.appendChild(row);
                }
            }
            if (hasSoldItems) {
                emptyMsg.classList.add('hidden');
                listBody.parentElement.classList.remove('hidden');
            } else {
                emptyMsg.classList.remove('hidden');
                listBody.parentElement.classList.add('hidden');
            }
        }

        function updateStats() {
            const availableCount = raffleData.filter(data => data.isAvailable).length;
            document.getElementById('available-count').innerText = availableCount;
            document.getElementById('occupied-count').innerText = TOTAL_NUMBERS - availableCount;
            updateSellerStats();
        }

        function updateSellerStats() {
            const sellers = ["Barbara", "Ashley", "Kamila", "Valeria", "Juan", "Emmanuel", "Daniela", "Santiago", "Mahia"];
            const counts = {};
            // Inicializar contadores en 0
            sellers.forEach(s => counts[s] = 0);
            
            // Contar ventas
            raffleData.forEach(ticket => {
                if (!ticket.isAvailable && ticket.seller) {
                    if (counts[ticket.seller] !== undefined) {
                        counts[ticket.seller]++;
                    }
                }
            });

            // Dibujar las tarjetas
            const container = document.getElementById('seller-stats-container');
            if (container) {
                container.innerHTML = '';
                // Ordenar por el que tiene más ventas a menos
                sellers.sort((a, b) => counts[b] - counts[a]).forEach(seller => {
                    const count = counts[seller];
                    container.innerHTML += `
                        <div class="bg-gray-50 rounded-xl p-3 border border-gray-100 flex flex-col items-center shadow-sm min-w-[85px] flex-1 max-w-[120px]">
                            <span class="text-xs font-semibold text-gray-600 truncate w-full text-center">${seller}</span>
                            <span class="text-2xl font-bold ${count > 0 ? 'text-blue-600' : 'text-gray-300'}">${count}</span>
                        </div>
                    `;
                });
            }
        }

        // Cierre de modales al clickear fuera
        document.getElementById('data-modal').addEventListener('click', function(e) { if (e.target === this) closeModal(); });
        document.getElementById('confirm-modal').addEventListener('click', function(e) { if (e.target === this) closeConfirmModal(); });

        window.onload = initApp;

    </script>
</body>
</html>
