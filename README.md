<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IP Address Details</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212; /* Dark background */
            color: #e0e0e0; /* Light text color */
            margin: 0;
            padding: 20px;
        }
        h1 {
            color: #ffffff; /* White color for header */
            text-align: center;
        }
        input[type="text"] {
            padding: 10px;
            width: 300px;
            margin-bottom: 10px;
            border: 1px solid #555; /* Darker border */
            border-radius: 5px;
            background-color: #1e1e1e; /* Darker input background */
            color: #ffffff; /* White text in input */
            font-family: 'Courier New', Courier, monospace; /* Monospace font */
        }
        button {
            padding: 10px 20px;
            background-color: #007bff; /* Blue button */
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #0056b3; /* Darker blue on hover */
        }
        .output-container {
            margin-top: 20px;
            background: #1e1e1e; /* Dark background for output */
            padding: 15px;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5); /* Darker shadow */
            overflow: auto; /* Scrollable if content overflows */
        }
        pre {
            background: #2a2a2a; /* Darker preformatted text background */
            padding: 10px;
            border-radius: 5px;
            color: #ffffff; /* White text color */
            font-family: 'Courier New', Courier, monospace; /* Monospace font */
            font-size: 14px; /* Font size similar to code editor */
            white-space: pre-wrap; /* Wrap long lines */
            line-height: 1.5; /* Line height for better readability */
            border: 1px solid #555; /* Border around preformatted text */
        }
        /* Colored output for JSON keys */
        .key { color: #8abeb7; } /* Teal for keys */
        .string { color: #f8c555; } /* Yellow for strings */
        .number { color: #b4b5ff; } /* Light blue for numbers */
        .boolean { color: #f92672; } /* Red for boolean */
        .null { color: #75715e; } /* Gray for null */
        .abuse { color: #ff5555; font-weight: bold; } /* Highlight for abuse information */
        .error { color: #ff5555; font-weight: bold; } /* Highlight for error messages */
    </style>
</head>
<body>
    <h1>IP Address Details</h1>
    <input type="text" id="ipInput" placeholder="Enter IP address (e.g., 175.157.240.222)">
    <input type="text" id="tokenInput" placeholder="Enter API Token" required>
    <button onclick="getIPDetails()">Get Details</button>
    
    <div class="output-container">
        <h2 id="outputHeader">Details:</h2>
        <pre id="output"></pre>
    </div>

    <script>
        async function getIPDetails() {
            const ip = document.getElementById("ipInput").value;
            const token = document.getElementById("tokenInput").value;
            const output = document.getElementById("output");
            const outputHeader = document.getElementById("outputHeader");

            if (!token) {
                output.innerHTML = `<span class="error">Error: Please enter an API token.</span>`;
                return; // Stop execution if no token is provided
            }

            try {
                // Fetch the user's IP information if no IP is provided
                const response = await fetch(`https://ipinfo.io/${ip || 'json'}?token=${token}`);
                if (!response.ok) throw new Error('IP address not found.');

                const data = await response.json();

                // Set the output header based on user input
                if (ip) {
                    outputHeader.innerText = `Target IP Details for: ${ip}`;
                } else {
                    outputHeader.innerText = `User's IP Details`;
                }

                // Extract potential cybercrime indicators
                const abuseInfo = data.abuse || { address: "N/A", country: "N/A", email: "N/A", name: "N/A", network: "N/A", phone: "N/A" };
                const privacyInfo = data.privacy || { vpn: false, proxy: false, tor: false };

                // Append additional data if not present
                data.abuse = abuseInfo;
                data.privacy = privacyInfo;

                output.innerHTML = formatJSON(data, privacyInfo);
            } catch (error) {
                output.innerHTML = `<span class="error">Error: ${error.message}</span>`;
            }
        }

        function formatJSON(json, privacy) {
            let jsonString = JSON.stringify(json, null, 2)
                .replace(/"([^"]+)":/g, `<span class="key">"$1":</span>`) // Color the keys
                .replace(/: "(.*?)"/g, `: <span class="string">"$1"</span>`) // Color the string values
                .replace(/: (\d+)/g, `: <span class="number">$1</span>`) // Color the numbers
                .replace(/: (true|false)/g, `: <span class="boolean">$1</span>`) // Color the booleans
                .replace(/: null/g, `: <span class="null">null</span>`); // Color null

            // Add abuse information with special highlighting
            if (privacy.vpn || privacy.proxy || privacy.tor) {
                jsonString += `<br><span class="abuse">Potentially suspicious activity detected! Anonymity services used:</span>`;
                if (privacy.vpn) jsonString += `<br><span class="key">VPN:</span> <span class="boolean">true</span>`;
                if (privacy.proxy) jsonString += `<br><span class="key">Proxy:</span> <span class="boolean">true</span>`;
                if (privacy.tor) jsonString += `<br><span class="key">Tor:</span> <span class="boolean">true</span>`;
            }

            return jsonString;
        }
    </script>
</body>
      </html>
