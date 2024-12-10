<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI NFT Creator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/5.7.2/ethers.umd.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: #f3f4f6;
        }

        .gradient-bg {
            background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
        }

        .card {
            background: white;
            border-radius: 16px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            padding: 24px;
            margin-bottom: 24px;
            transition: all 0.3s ease;
        }

        .card:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 12px -1px rgba(0, 0, 0, 0.15);
        }

        .btn-primary {
            background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
            color: white;
            padding: 12px 24px;
            border-radius: 8px;
            border: none;
            cursor: pointer;
            transition: all 0.3s;
            font-weight: 500;
        }

        .btn-primary:hover {
            opacity: 0.9;
            transform: translateY(-1px);
        }

        .btn-primary:disabled {
            background: #d1d5db;
            cursor: not-allowed;
            transform: none;
        }

        .input-field {
            width: 100%;
            padding: 12px;
            border: 1px solid #e5e7eb;
            border-radius: 8px;
            margin-bottom: 16px;
            transition: border-color 0.2s;
        }

        .input-field:focus {
            outline: none;
            border-color: #6366f1;
        }

        .preview-image {
            width: 100%;
            height: 300px;
            object-fit: cover;
            border-radius: 8px;
            border: 2px dashed #e5e7eb;
            background: #f8fafc;
        }

        .loading {
            position: relative;
        }

        .loading::after {
            content: '';
            position: absolute;
            width: 100%;
            height: 100%;
            top: 0;
            left: 0;
            background: rgba(255, 255, 255, 0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 8px;
        }
    </style>
</head>

<body class="min-h-screen pb-8">
    <nav class="gradient-bg text-white p-4 mb-8">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-2xl font-bold">AI NFT Creator</h1>
            <div id="walletInfo" class="text-sm"></div>
        </div>
    </nav>

    <main class="container mx-auto px-4 max-w-4xl">
        <!-- Wallet Setup -->
        <div class="card" id="walletSetup">
            <h2 class="text-xl font-bold mb-4">🔐 Wallet Setup</h2>
            <div class="flex gap-4 flex-wrap">
                <button onclick="createNewWallet()" id="createWalletBtn" class="btn-primary">
                    Create New Wallet
                </button>
                <button onclick="importWallet()" id="importWalletBtn" class="btn-primary">
                    Import Wallet
                </button>
            </div>
            <input type="text" id="privateKey" placeholder="Enter your private key (with 0x prefix)"
                class="input-field mt-4" style="display: none;">
            <div id="walletDetails" class="mt-4 text-sm text-gray-600"></div>
        </div>

        <!-- Smart Account Setup -->
        <div class="card" id="smartAccountSetup">
            <h2 class="text-xl font-bold mb-4">🔑 Smart Account Setup</h2>
            <button onclick="setupSmartAccount()" id="setupBtn" disabled class="btn-primary w-full">
                Setup Smart Account
            </button>
            <div id="smartAccountInfo" class="mt-4 text-sm text-gray-600"></div>
        </div>

        <!-- NFT Creation -->
        <div class="card" id="nftCreation">
            <h2 class="text-xl font-bold mb-4">🎨 Create AI-Generated NFT</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                    <textarea id="promptInput" placeholder="Describe your NFT artwork in detail..." class="input-field"
                        rows="4"></textarea>
                    <select id="styleSelect" class="input-field">
                        <option value="realistic">Realistic Photography</option>
                        <option value="abstract">Abstract Art</option>
                        <option value="digital">Digital Art</option>
                        <option value="3d">3D Render</option>
                        <option value="pixel">Pixel Art</option>
                        <option value="cartoon">Cartoon Style</option>
                    </select>
                    <button onclick="generateArtwork()" id="generateBtn" class="btn-primary w-full">
                        Generate Artwork
                    </button>
                </div>
                <div>
                    <img id="artworkPreview" class="preview-image"
                        src="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='400' height='300' viewBox='0 0 400 300'%3E%3Crect width='400' height='300' fill='%23f8fafc'/%3E%3Ctext x='50%25' y='50%25' text-anchor='middle' dy='.3em' font-family='Arial' font-size='14' fill='%2394a3b8'%3EArtwork preview will appear here%3C/text%3E%3C/svg%3E"
                        alt="Artwork preview">
                </div>
            </div>
        </div>

        <!-- NFT Details -->
        <div class="card" id="nftDetails">
            <h2 class="text-xl font-bold mb-4">📝 NFT Details</h2>
            <input type="text" id="nftName" placeholder="NFT Name" class="input-field">
            <textarea id="nftDescription" placeholder="NFT Description" class="input-field" rows="3"></textarea>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                <div>
                    <label class="block text-sm font-medium mb-2">💰 Base Price (ETH)</label>
                    <input type="number" id="basePrice" placeholder="0.0" step="0.001" min="0" class="input-field">
                    <p id="priceRecommendation" class="text-sm text-purple-600"></p>
                </div>
                <div>
                    <label class="block text-sm font-medium mb-2">🤝 Royalty Split</label>
                    <div class="flex gap-2">
                        <input type="text" id="collaboratorAddress" placeholder="Collaborator Address (0x...)"
                            class="input-field flex-grow">
                        <input type="number" id="royaltyPercentage" placeholder="%" min="0" max="100"
                            class="input-field w-24">
                    </div>
                </div>
            </div>

            <button onclick="mintNFT()" id="mintBtn" class="btn-primary w-full" disabled>
                Mint NFT (Gasless)
            </button>
        </div>

        <!-- Transaction Status -->
        <div class="card" id="transactionStatus" style="display: none;">
            <h2 class="text-xl font-bold mb-4">📊 Transaction Status</h2>
            <div id="statusText" class="text-gray-600"></div>
            <div id="transactionHash" class="text-sm text-blue-600 mt-2"></div>
        </div>
    </main>

    <script type="module">
        import { createSmartAccountClient, PaymasterMode } from 'https://cdn.jsdelivr.net/npm/@0xgasless/smart-account@0.0.11/+esm';

        const CONFIG = {
            rpcUrl: "https://avax-mainnet.g.alchemy.com/v2/VcifYAbO7TUL14eUvHmbjSklgVzWEMXk",
            bundlerUrl: "https://bundler.0xgasless.com/43114",
            paymasterUrl: "https://paymaster.0xgasless.com/v1/43114/rpc/35b99953-edde-4f28-9661-c41898916471",
            chainId: 43114,
            nftContractAddress: "0xe7337B97F380A19A860068f0dACa74F50AF86548",
            pinata: {
                jwt: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySW5mb3JtYXRpb24iOnsiaWQiOiIxOTc0ZWM3ZS1hNjk5LTQzM2QtODk1MS00YTEyMDcyOGY2MzkiLCJlbWFpbCI6ImFkaXMyMTEwNEBnbWFpbC5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwicGluX3BvbGljeSI6eyJyZWdpb25zIjpbeyJkZXNpcmVkUmVwbGljYXRpb25Db3VudCI6MSwiaWQiOiJGUkExIn0seyJkZXNpcmVkUmVwbGljYXRpb25Db3VudCI6MSwiaWQiOiJOWUMxIn1dLCJ2ZXJzaW9uIjoxfSwibWZhX2VuYWJsZWQiOmZhbHNlLCJzdGF0dXMiOiJBQ1RJVkUifSwiYXV0aGVudGljYXRpb25UeXBlIjoic2NvcGVkS2V5Iiwic2NvcGVkS2V5S2V5IjoiNzhjOTM3NDMwNGMzYmRjNmYyYTciLCJzY29wZWRLZXlTZWNyZXQiOiIxYTUxMzAwY2ZhYWE2ZDY0M2JkOTYyMWIxNjA3ZmViYWUwNjQzMzA4NTk3Y2EyNDBhZTdjMzk3N2FmMDI0M2RhIiwiZXhwIjoxNzY1MjkwNDEwfQ.mGeY9zom2wbgZjlKHkrDQNisAKhv93-vk_X2D2mCZow',
                apiEndpoint: 'https://api.pinata.cloud/pinning'
            }
        };

        let smartWallet;
        let provider;
        let wallet;

        // Initialize event listeners
        document.addEventListener('DOMContentLoaded', () => {
            document.getElementById('privateKey').addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    importWalletFromKey(e.target.value);
                }
            });
        });

        window.createNewWallet = async function () {
            try {
                const createWalletBtn = document.getElementById('createWalletBtn');
                const importWalletBtn = document.getElementById('importWalletBtn');
                const setupBtn = document.getElementById('setupBtn');

                createWalletBtn.disabled = true;
                importWalletBtn.disabled = true;

                provider = new ethers.providers.JsonRpcProvider(CONFIG.rpcUrl);
                wallet = ethers.Wallet.createRandom().connect(provider);

                document.getElementById('walletDetails').innerHTML = `
                    <div class="p-4 bg-green-50 rounded-lg">
                        <p><strong>EOA Address:</strong> ${wallet.address}</p>
                        <p><strong>Private Key:</strong> ${wallet.privateKey}</p>
                        <p class="text-red-500 font-bold mt-2">⚠️ Save this private key securely!</p>
                    </div>
                `;

                setupBtn.disabled = false;
                createWalletBtn.disabled = false;
                importWalletBtn.disabled = false;
            } catch (error) {
                alert(`Error creating wallet: ${error.message}`);
                createWalletBtn.disabled = false;
                importWalletBtn.disabled = false;
            }
        };

        window.importWallet = function () {
            const privateKeyInput = document.getElementById('privateKey');
            privateKeyInput.style.display = privateKeyInput.style.display === 'none' ? 'block' : 'none';
            if (privateKeyInput.style.display === 'block') {
                privateKeyInput.focus();
            }
        };

        async function importWalletFromKey(privateKey) {
            try {
                if (!privateKey.startsWith('0x')) {
                    throw new Error('Private key must start with 0x');
                }

                provider = new ethers.providers.JsonRpcProvider(CONFIG.rpcUrl);
                wallet = new ethers.Wallet(privateKey, provider);

                document.getElementById('walletDetails').innerHTML = `
                    <div class="p-4 bg-green-50 rounded-lg">
                        <p><strong>EOA Address:</strong> ${wallet.address}</p>
                        <p><strong>Status:</strong> Wallet imported successfully!</p>
                    </div>
                `;
                document.getElementById('setupBtn').disabled = false;
                document.getElementById('privateKey').style.display = 'none';
                document.getElementById('mintBtn').disabled = false;
            } catch (error) {
                alert(`Error importing wallet: ${error.message}`);
            }
        }

        window.setupSmartAccount = async function () {
            try {
                const setupBtn = document.getElementById('setupBtn');
                setupBtn.disabled = true;

                smartWallet = await createSmartAccountClient({
                    signer: wallet,
                    paymasterUrl: CONFIG.paymasterUrl,
                    bundlerUrl: CONFIG.bundlerUrl,
                    chainId: CONFIG.chainId
                });

                const address = await smartWallet.getAddress();
                document.getElementById('smartAccountInfo').innerHTML = `
                    <div class="p-4 bg-green-50 rounded-lg">
                        <p><strong>Smart Account Address:</strong> ${address}</p>
                        <p><strong>Status:</strong> Ready for transactions</p>
                    </div>
                `;

                // Show NFT creation sections after smart account setup
                document.getElementById('nftCreation').style.display = 'block';
                document.getElementById('nftDetails').style.display = 'block';

                setupBtn.disabled = true; // Keep disabled after setup
            } catch (error) {
                document.getElementById('smartAccountInfo').innerHTML = `
                    <div class="p-4 bg-red-50 rounded-lg text-red-600">
                        Error: ${error.message}
                    </div>
                `;
                setupBtn.disabled = false;
            }
        };

        window.generateArtwork = async function () {
            try {
                const generateBtn = document.getElementById('generateBtn');
                generateBtn.disabled = true;

                const prompt = document.getElementById('promptInput').value;
                const style = document.getElementById('styleSelect').value;

                if (!prompt) {
                    throw new Error('Please enter a description for your artwork');
                }

                // Format the prompt based on the selected style
                const formattedPrompt = formatPrompt(prompt, style);
                console.log("formattedPrompt are : ", formattedPrompt);

                // Call Stable Diffusion API
                // const response = await fetch('https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image', {
                    const response = await fetch('https://api-inference.huggingface.co/models/prompthero/openjourney-v4', {
                    method: 'POST',
                    headers: {
                        'Authorization': "Bearer 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",
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        "inputs": formattedPrompt
                    })
                });
                console.log("respose are : ", response);

                if (!response.ok) {
                    throw new Error(`Generation failed: ${response.statusText}`);
                }

                const blob = await response.blob();
                const imageUrl = URL.createObjectURL(blob);

                // Update the preview image
                const imageElement = document.getElementById('artworkPreview');
                imageElement.src = imageUrl;

                // Get AI-recommended price based on style and complexity
                const recommendedPrice = await getRecommendedPrice();
                document.getElementById('priceRecommendation').textContent =
                    `🤖 AI Recommended Price: ${recommendedPrice} ETH`;

                // Enable the mint button after successful generation
                document.getElementById('mintBtn').disabled = false;

            } catch (error) {
                console.error("Error generating artwork:", error);
                alert(`Failed to generate artwork: ${error.message}`);
            } finally {
                generateBtn.disabled = false;
            }
        };

        // Helper function to format prompts based on style
        function formatPrompt(prompt, style) {
            const stylePrompts = {
                'realistic': `ultra realistic photograph, professional photography, ${prompt}, 8k uhd, high detail`,
                'abstract': `abstract art style, modern art, ${prompt}, vibrant colors, artistic expression`,
                'digital': `digital art, concept art, ${prompt}, detailed, vibrant, high resolution`,
                '3d': `3D rendered art, octane render, ${prompt}, high detail, realistic lighting`,
                'pixel': `pixel art style, 16-bit, ${prompt}, retro gaming aesthetic`,
                'cartoon': `cartoon style, illustration, ${prompt}, vibrant colors, clean lines`
            };

            return stylePrompts[style.toLowerCase()] || prompt;
        }

        async function uploadToIPFS(metadata) {
            try {
                // First, upload the image to Pinata
                const imageBlob = await fetch(document.getElementById('artworkPreview').src).then(r => r.blob());
                const imageFormData = new FormData();
                imageFormData.append('file', imageBlob, 'artwork.png');

                const imageResponse = await fetch('https://api.pinata.cloud/pinning/pinFileToIPFS', {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${CONFIG.pinata.jwt}`  // Replace with your Pinata JWT
                    },
                    body: imageFormData
                });

                if (!imageResponse.ok) {
                    throw new Error('Failed to upload image to IPFS');
                }

                const imageResult = await imageResponse.json();
                const imageIPFSHash = imageResult.IpfsHash;

                // Update metadata with the IPFS image URL
                metadata.image = `ipfs://${imageIPFSHash}`;

                // Then upload the metadata JSON to Pinata
                const metadataResponse = await fetch('https://api.pinata.cloud/pinning/pinJSONToIPFS', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${CONFIG.pinata.jwt}`  // Replace with your Pinata JWT
                    },
                    body: JSON.stringify(metadata)
                });

                if (!metadataResponse.ok) {
                    throw new Error('Failed to upload metadata to IPFS');
                }

                const metadataResult = await metadataResponse.json();
                return `ipfs://${metadataResult.IpfsHash}`;
            } catch (error) {
                console.error('Error uploading to IPFS:', error);
                throw new Error('Failed to upload to IPFS: ' + error.message);
            }
        }

        window.mintNFT = async function () {
            try {
                const mintBtn = document.getElementById('mintBtn');
                mintBtn.disabled = true;

                const name = document.getElementById('nftName').value;
                const description = document.getElementById('nftDescription').value;
                const price = document.getElementById('basePrice').value;
                const collaborator = document.getElementById('collaboratorAddress').value;
                const royalty = document.getElementById('royaltyPercentage').value;

                if (!name || !description || !price) {
                    throw new Error('Please fill in all required fields');
                }

                const statusDiv = document.getElementById('transactionStatus');
                statusDiv.style.display = 'block';
                document.getElementById('statusText').textContent = 'Uploading to IPFS...';

                // Create and upload metadata
                const metadata = await createNFTMetadata(name, description);
                const tokenURI = await uploadToIPFS(metadata);

                document.getElementById('statusText').textContent = 'Preparing to mint NFT...';

                // Create the mint transaction with the correct function name from the ABI
                const nftInterface = new ethers.utils.Interface(nftABI);
                const data = nftInterface.encodeFunctionData("mintNFT", [tokenURI]);

                const tx = {
                    to: CONFIG.nftContractAddress,
                    data: data,
                    value: "0x0"
                };

                try {
                    const userOpResponse = await smartWallet.sendTransaction(tx, {
                        paymasterServiceData: {
                            mode: PaymasterMode.SPONSORED
                        }
                    });

                    document.getElementById('statusText').textContent = 'Minting in progress...';
                    document.getElementById('transactionHash').innerHTML = `
                        Transaction Hash: <a href="https://subnets.avax.network/c-chain/tx/${userOpResponse.hash}" 
                        target="_blank" class="underline">${userOpResponse.hash}</a>
                    `;

                    // Add timeout handling for receipt
                    const receipt = await Promise.race([
                        userOpResponse.wait(),
                        new Promise((_, reject) => 
                            setTimeout(() => reject(new Error('Transaction confirmation timeout')), 60000)
                        )
                    ]);

                    if (receipt.success) {
                        document.getElementById('statusText').innerHTML = `
                            <div class="text-green-600">
                                ✅ NFT Minted Successfully!<br>
                                Name: ${name}<br>
                                Price: ${price} ETH<br>
                                ${collaborator ? `Collaborator: ${collaborator} (${royalty}%)` : ''}
                            </div>
                        `;

                        // Add link to view NFT
                        document.getElementById('transactionHash').innerHTML += `
                            <br>View your NFT on: <a href="https://nft.avax.network/collection/${CONFIG.nftContractAddress}" 
                            target="_blank" class="underline">Avalanche NFT Marketplace</a>
                        `;
                    } else {
                        throw new Error('Transaction failed');
                    }
                } catch (txError) {
                    if (txError.message.includes('timeout')) {
                        document.getElementById('statusText').innerHTML = `
                            <div class="text-yellow-600">
                                ⚠️ Transaction submitted but confirmation is taking longer than expected.<br>
                                You can check the status later using the transaction hash.
                            </div>
                        `;
                    } else {
                        throw txError;
                    }
                }

                mintBtn.disabled = false;
            } catch (error) {
                console.error("Minting error:", error);
                document.getElementById('statusText').innerHTML = `<div class="text-red-600">
                        ❌ Error: ${error.message}
                    </div>
                `;
                document.getElementById('mintBtn').disabled = false;
            }
        };

        // NFT Contract ABI
        const nftABI = [
            {
                "inputs": [],
                "stateMutability": "nonpayable",
                "type": "constructor"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "sender",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    },
                    {
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    }
                ],
                "name": "ERC721IncorrectOwner",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "operator",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "ERC721InsufficientApproval",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "approver",
                        "type": "address"
                    }
                ],
                "name": "ERC721InvalidApprover",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "operator",
                        "type": "address"
                    }
                ],
                "name": "ERC721InvalidOperator",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    }
                ],
                "name": "ERC721InvalidOwner",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "receiver",
                        "type": "address"
                    }
                ],
                "name": "ERC721InvalidReceiver",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "sender",
                        "type": "address"
                    }
                ],
                "name": "ERC721InvalidSender",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "ERC721NonexistentToken",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    }
                ],
                "name": "OwnableInvalidOwner",
                "type": "error"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "account",
                        "type": "address"
                    }
                ],
                "name": "OwnableUnauthorizedAccount",
                "type": "error"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "approved",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "Approval",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "operator",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "bool",
                        "name": "approved",
                        "type": "bool"
                    }
                ],
                "name": "ApprovalForAll",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "_fromTokenId",
                        "type": "uint256"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "_toTokenId",
                        "type": "uint256"
                    }
                ],
                "name": "BatchMetadataUpdate",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "_tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "MetadataUpdate",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "string",
                        "name": "uri",
                        "type": "string"
                    }
                ],
                "name": "NFTMinted",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "previousOwner",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "newOwner",
                        "type": "address"
                    }
                ],
                "name": "OwnershipTransferred",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "collaborator",
                        "type": "address"
                    },
                    {
                        "indexed": false,
                        "internalType": "uint256",
                        "name": "percentage",
                        "type": "uint256"
                    }
                ],
                "name": "RoyaltySet",
                "type": "event"
            },
            {
                "anonymous": false,
                "inputs": [
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "indexed": true,
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "Transfer",
                "type": "event"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "approve",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    }
                ],
                "name": "balanceOf",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "burn",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "getApproved",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "",
                        "type": "address"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "getRoyaltyInfo",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "getTokenCounter",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "owner",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "operator",
                        "type": "address"
                    }
                ],
                "name": "isApprovedForAll",
                "outputs": [
                    {
                        "internalType": "bool",
                        "name": "",
                        "type": "bool"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "string",
                        "name": "uri",
                        "type": "string"
                    }
                ],
                "name": "mintNFT",
                "outputs": [
                    {
                        "internalType": "uint256",
                        "name": "",
                        "type": "uint256"
                    }
                ],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "name",
                "outputs": [
                    {
                        "internalType": "string",
                        "name": "",
                        "type": "string"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "owner",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "",
                        "type": "address"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "ownerOf",
                "outputs": [
                    {
                        "internalType": "address",
                        "name": "",
                        "type": "address"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "renounceOwnership",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "safeTransferFrom",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    },
                    {
                        "internalType": "bytes",
                        "name": "data",
                        "type": "bytes"
                    }
                ],
                "name": "safeTransferFrom",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "operator",
                        "type": "address"
                    },
                    {
                        "internalType": "bool",
                        "name": "approved",
                        "type": "bool"
                    }
                ],
                "name": "setApprovalForAll",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    },
                    {
                        "internalType": "address",
                        "name": "collaborator",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "percentage",
                        "type": "uint256"
                    }
                ],
                "name": "setRoyalties",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "bytes4",
                        "name": "interfaceId",
                        "type": "bytes4"
                    }
                ],
                "name": "supportsInterface",
                "outputs": [
                    {
                        "internalType": "bool",
                        "name": "",
                        "type": "bool"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [],
                "name": "symbol",
                "outputs": [
                    {
                        "internalType": "string",
                        "name": "",
                        "type": "string"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "tokenURI",
                "outputs": [
                    {
                        "internalType": "string",
                        "name": "",
                        "type": "string"
                    }
                ],
                "stateMutability": "view",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "from",
                        "type": "address"
                    },
                    {
                        "internalType": "address",
                        "name": "to",
                        "type": "address"
                    },
                    {
                        "internalType": "uint256",
                        "name": "tokenId",
                        "type": "uint256"
                    }
                ],
                "name": "transferFrom",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            },
            {
                "inputs": [
                    {
                        "internalType": "address",
                        "name": "newOwner",
                        "type": "address"
                    }
                ],
                "name": "transferOwnership",
                "outputs": [],
                "stateMutability": "nonpayable",
                "type": "function"
            }
        ];

        // Helper function to validate Ethereum address
        function isValidAddress(address) {
            try {
                ethers.utils.getAddress(address);
                return true;
            } catch {
                return false;
            }
        }

        // Helper function to create metadata
        async function createNFTMetadata(name, description) {
            const imageUrl = document.getElementById('artworkPreview').src;
            return {
                name,
                description,
                image: imageUrl, // This will be replaced with IPFS URL in uploadToIPFS
                attributes: [
                    {
                        trait_type: "Generator",
                        value: "AI Art Creator v1.0"
                    },
                    {
                        trait_type: "Style",
                        value: document.getElementById('styleSelect').value
                    },
                    {
                        trait_type: "Creation Date",
                        value: new Date().toISOString()
                    }
                ]
            };
        }

        // Initialize price recommendation system
        async function getRecommendedPrice() {
            // In a real implementation, this would use AI to analyze similar NFTs
            // For now, we'll return a random price
            const basePrice = 0.05;
            const variance = Math.random() * 0.1 - 0.05;
            return (basePrice + variance).toFixed(3);
        }

        // Setup event listeners for input validation
        document.getElementById('collaboratorAddress').addEventListener('input', function (e) {
            const address = e.target.value;
            if (address && !isValidAddress(address)) {
                e.target.style.borderColor = '#ef4444';
            } else {
                e.target.style.borderColor = '#e5e7eb';
            }
        });

        document.getElementById('royaltyPercentage').addEventListener('input', function (e) {
            const value = parseFloat(e.target.value);
            if (isNaN(value) || value < 0 || value > 100) {
                e.target.style.borderColor = '#ef4444';
            } else {
                e.target.style.borderColor = '#e5e7eb';
            }
        });

        // Add loading indicators
        function setLoading(elementId, isLoading) {
            const element = document.getElementById(elementId);
            if (isLoading) {
                element.classList.add('loading');
            } else {
                element.classList.remove('loading');
            }
        }

        // Initialize tooltips
        const tooltips = document.querySelectorAll('[data-tooltip]');
        tooltips.forEach(element => {
            element.addEventListener('mouseenter', e => {
                const tooltip = document.createElement('div');
                tooltip.className = 'absolute bg-gray-800 text-white p-2 rounded text-sm';
                tooltip.textContent = e.target.dataset.tooltip;
                document.body.appendChild(tooltip);

                const rect = e.target.getBoundingClientRect();
                tooltip.style.left = rect.left + 'px';
                tooltip.style.top = (rect.bottom + 5) + 'px';

                e.target.addEventListener('mouseleave', () => tooltip.remove());
            });
        });

    </script>

    <!-- Add a global error boundary -->
    <div id="errorBoundary"
        class="fixed bottom-4 right-4 max-w-sm bg-red-100 border-l-4 border-red-500 text-red-700 p-4"
        style="display: none;">
        <div class="flex">
            <div class="py-1">
                <svg class="h-6 w-6 text-red-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
            </div>
            <div class="ml-3">
                <p class="text-sm" id="errorMessage"></p>
            </div>
        </div>
    </div>

    <!-- Add a feedback widget -->
    <div class="fixed bottom-4 left-4">
        <button onclick="showFeedback()"
            class="bg-purple-600 text-white rounded-full p-3 shadow-lg hover:bg-purple-700 transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24"
                stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z" />
            </svg>
        </button>
    </div>

</body>

</html>