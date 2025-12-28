    <script>
        let authCode = '';
        let token = '';
        let view = 'main'; // 'main' | 'details'
        let scannedCode = '';
        let garageName = '';
        let parkingTime = '';

        function authenticate() {
            return new Promise((resolve, reject) => {
                my.getAuthCode({
                    scopes: ['auth_base', 'USER_ID'],
                    success: (res) => {
                        authCode = res.authCode;
                        const el = document.getElementById('authCode');
                        if (el) el.textContent = authCode;

                        fetch('https://its.mouamle.space/api/auth-with-superQi', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({
                                token: authCode
                            })
                        }).then(res => res.json()).then(data => {
                            token = data.token;
                            my.alert({ content: "Login successful" });
                            resolve(token);
                        }).catch(err => {
                            let errorDetails = '';
                            if (err && typeof err === 'object') {
                                errorDetails = JSON.stringify(err, null, 2);
                            } else {
                                errorDetails = String(err);
                            }
                            my.alert({ content: "Error: " + errorDetails });
                            reject(err);
                        });
                    },
                    fail: (res) => {
                        console.log(res.authErrorScopes);
                        reject(res);
                    }
                });
            });
        }

        function pay() {
            fetch('https://its.mouamle.space/api/payment', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': token
                },
            }).then(res => res.json()).then(data => {
                my.tradePay({
                    paymentUrl: data.url,
                    success: (res) => {
                        my.alert({
                            content: "Payment successful",
                        });
                    },
                });
            }).catch(err => {
                my.alert({
                    content: "Payment failed",
                });
            });
        }

        function scan() {
            // Ensure user is authenticated first
            authenticate().then(() => {
                my.scan({
                    type: 'qr',
                    success: (res) => {
                        scannedCode = res.code;

                        // Try getting more details from the backend; fallback to simple values
                        fetch('https://its.mouamle.space/api/garage-info', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json',
                                'Authorization': token
                            },
                            body: JSON.stringify({ code: scannedCode })
                        }).then(r => r.json()).then(data => {
                            garageName = data.name || ('Garage ' + scannedCode);
                            parkingTime = data.parkingTime || '1 hour';
                            view = 'details';
                        }).catch(err => {
                            // fallback
                            garageName = 'Garage ' + scannedCode;
                            parkingTime = '1 hour';
                            view = 'details';
                        });
                    },
                    fail: (err) => {
                        my.alert({ content: 'Scan failed' });
                    },
                });
            }).catch(err => {
                my.alert({ content: 'Authentication required to scan' });
            });
        }

        function selectBay() {
            // Simple action for the bay button - try reserve, otherwise show confirmation
            fetch('https://its.mouamle.space/api/select-bay', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': token
                },
                body: JSON.stringify({ code: scannedCode })
            }).then(r => r.json()).then(data => {
                my.alert({ content: data.message || 'Bay selected' });
            }).catch(err => {
                my.alert({ content: 'Bay selection failed' });
            });
        }

        function backToMain() {
            view = 'main';
        }

    </script>

    <main class="flex items-center justify-center min-h-screen">

    {#if view === 'main'}
        <div class="space-y-3 flex flex-col items-center">
            <button
                onclick={authenticate}
                class=" border border-black bg-blue-900 px-8 py-4"
            >
                Auth
            </button>
            <p id="authCode" class="text-sm text-gray-900 mt-2">{authCode || 'No auth code'}</p>

            <button 
                onclick={pay}
                class=" border border-black bg-blue-900 px-8  py-4">
                Pay
            </button>

            <button 
                onclick={scan} 
                class=" border border-black bg-blue-900 px-8  py-4">
                Scan
            </button>
        </div>
    {:else}
        <div class="p-6 bg-white rounded shadow text-center">
            <h2 class="text-xl font-bold mb-2">{garageName}</h2>
            <p class="mb-4">Parking time: <strong>{parkingTime}</strong></p>
            <div class="flex justify-center gap-4">
                <button onclick={selectBay} class="border border-black bg-green-600 px-6 py-2 text-white rounded">Bay</button>
                <button onclick={backToMain} class="border border-black bg-gray-300 px-6 py-2 rounded">Back</button>
            </div>
        </div>
    {/if}

    </main>