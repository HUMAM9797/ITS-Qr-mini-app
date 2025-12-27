    <script>
        var authCode = '';
        var token = '';

        function authenticate() {
            my.getAuthCode({
                scopes: ['auth_base', 'USER_ID'],
                success: (res) => {
                    authCode = res.authCode;
                    document.getElementById('authCode').textContent = authCode;

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
                        my.alert({
                            content: "Login successful",
                        });
                    }).catch(err => {
                        let errorDetails = '';
                        if (err && typeof err === 'object') {
                            errorDetails = JSON.stringify(err, null, 2);
                        } else {
                            errorDetails = String(err);
                        }
                        my.alert({
                            content: "Error: " + errorDetails,
                        });
                    });
                },
                fail: (res) => {
                    console.log(res.authErrorScopes)
                },
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

    function scan()
        {
        my.scan({
        type: 'qr',
        success: (res) => {
            my.alert({ title: res.code });
        },
    });
        }
    </script>

    <main>

       <button
        onclick={authenticate}
        class=" border border-amber-950 bg-blue-900  py-4"
    >
        Auth
    </button>
    <p id="authCode" class="text-sm text-gray-100 mt-2">No auth code</p>
    
    <button 
    onclick={pay}
    class=" border border-amber-950 bg-blue-900  py-4">
        Pay
    </button>
    <button onclick={scan}>
        scan
    </button>
    </main>