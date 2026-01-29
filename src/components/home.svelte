    <script>
        let token = '';
        let view = 'main'; 
        let scannedCode = '';
        let garageName = '';
        let parkingTime = '';

        function authenticate() {
            return new Promise((resolve, reject) => {
                my.getAuthCode({
                    scopes: ['auth_base', 'USER_ID'],
                    success: (res) => {
                        const authCode = res.authCode;
                        my.alert({ content: 'Got auth code: ' + authCode });

                        fetch('https://its.mouamle.space/api/auth-with-superQi', {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify({ token: authCode })
                        }).then(r => {
                            if (!r.ok) throw new Error('Auth API error ' + r.status);
                            return r.json();
                        }).then(data => {
                            token = data.token;
                            my.alert({ content: 'Login successful' });
                            resolve(token);
                        }).catch(err => {
                            my.alert({ content: 'Authentication failed: ' + (err && err.message ? err.message : JSON.stringify(err)) });
                            reject(err);
                        });
                    },
                    fail: (res) => {
                        my.alert({ content: 'getAuthCode failed: ' + JSON.stringify(res) });
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
                my.tradePay({ paymentUrl: data.url, success: () => my.alert({ content: 'Payment successful' }) });
            }).catch(err => {
                my.alert({ content: 'Payment failed' });
            });
        }

        function scan() {
            my.confirm({
                title: 'Authentication required',
                content: 'Allow authentication to proceed with scanning?',
                confirmButtonText: 'Allow',
                cancelButtonText: 'Cancel',
                success: (res) => {
                    my.alert({ content: 'Confirm result: ' + JSON.stringify(res) });
                    const allowed = res && (
                        res.confirm === true ||
                        res === true ||
                        res.confirm === 'confirm' ||
                        res.ok === true
                    );
                    if (!allowed) return;
                    my.alert({ content: 'Proceeding to authenticate...' });
                    authenticate().then(() => {
                        my.scan({
                            type: 'qr',
                            success: (res) => {
                                scannedCode = res.code;

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
                                }).catch(() => {
                                    garageName = 'Garage ' + scannedCode;
                                    parkingTime = '1 hour';
                                    view = 'details';
                                });
                            },
                            fail: () => my.alert({ content: 'Scan failed' })
                        });
                    }).catch(() => my.alert({ content: 'Authentication required to scan' }));
                }
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
                <button onclick={pay} class="border border-black bg-blue-900 px-6 py-2 text-white rounded">Pay</button>
                <button onclick={backToMain} class="border border-black bg-gray-300 px-6 py-2 rounded">Back</button>
            </div>
        </div>
    {/if}

    </main>