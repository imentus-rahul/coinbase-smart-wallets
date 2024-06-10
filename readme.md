## Coinbase Smart Wallets

- Ref: https://www.coinbase.com/en-gb/wallet/smart-wallet (FAQs)
- Smart wallets are secured by passkeys stored on the user's device. Private keys are not directly exposed.
- Passkeys are backed up with passkey providers such as Apple, Chrome, or 1Password, or on hardware such as YubiKeys.
- `Passkey` signatures are `validated directly onchain` via an open source and audited smart contract: https://github.com/base-org/webauthn-sol
  - Base Sepolia Addresses:
    - coinbaseSmartWalletProxyBytecode ="0x363d3d373d3d363d7f360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc545af43d6000803e6038573d6000fd5b3d6000f3";
    - coinbaseSmartWalletV1Implementation = "0x000100abaad02f1cfC8Bbe32bD5a564817339E72";
    - **magicSpendAddress** = "0x011A61C07DbF256A68256B1cB51A5e246730aB92";
    - erc1967ProxyImplementationSlot = "0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc";

### Best Practices to Manage Smart Wallets

- Ref: https://www.coinbase.com/en-gb/wallet/smart-wallet
- Only `passkey` can become single point of failure, even though passkeys are great for ease of use, it's quite possible to lose a hardware key or accidentally delete a software key.
- The safest approach is to always have an additional backup method to access your funds. Users can additionally create and link a backup key which is a standard EOA account that is added as a signer for the wallet. Users can generate and regenerate as many backups as they want.
- Your passkey and recovery wallet (if you set one up) are the only methods to access your wallet. Without your passkey, no one can recover the wallet for you. Please avoid bulk clean ups and deletions of passkeys to avoid accidental deletion.
- Smart Wallet more expensive to use, consumes more gas per transaction than EOA wallets (Because of onchain signature verification). This means they should be avoided on networks like Ethereum mainnet.

### Features | Enhancing User Experience

- Allow users to spend funds `directly from their Coinbase (Centralized Exchange) account` by taking advantage of `MagicSpend`.
- Allow users to initiate transactions in their wallet, even if they don't currently have enough ETH (Native Currency) to pay. Coinbase Smart Wallet (ERC-4337 compliant)/Account Abstraction handles the rest.
- Allow users to Magic Spend. Smart Wallet users can connect their `coinbase.com` account and spend their coinbase.com held ETH from their Smart Account (Base only, for now).
- Smart Wallet is ERC-4337 compliant and Allows users to make Batch Transactions, and app-defined Paymasters.
- In order to improve user experience and limiting user to use coinbase wallet (as it will pay the native gas onchain), dev can prevent other wallets from connecting using `window.ethereum.isCoinbaseWallet`.

### References

- Coinbase Wallet SDK: https://www.smartwallet.dev/sdk/
- WAGMI Coinbase Integration: https://wagmi.sh/react/api/connectors/coinbaseWallet
- Blog - Batch Transactions with Smart Wallet using ThirdWeb: https://blog.thirdweb.com/guides/how-to-batch-transactions-with-the-thirdweb-sdk/
- Code - Next.js project using `create-wagmi` and updated for `Smart Wallet`: https://github.com/wilsoncusack/wagmi-scw/ | Demo - wagmi-scw.vercel.app
  - Making Tx on Base Sepolia, Contract Address: https://sepolia.basescan.org/address/0x119ea671030fbf79ab93b436d2e20af6ea469a19

### Coinbase Smart Wallet SDK

- Installation: `npm i @coinbase/wallet-sdk`
- Setting up SDK: https://www.smartwallet.dev/sdk/setup

```JS
import { CoinbaseWalletSDK } from '@coinbase/wallet-sdk'

const sdk = new CoinbaseWalletSDK({
  appName: 'My Custom App Name', // The app name will be displayed to users on connection, transacting, and signing requests.
  appChainIds: [11155111, 31337, 50524], // Array of chain IDs your app supports
  appLogoUrl: 'https://dancelogo.com/images/example/logo-static-420.png', // URL to your app's logo
});

```

- Using SDK: https://www.smartwallet.dev/sdk/makeWeb3Provider

```JS
import {sdk} from "./setup";

// Create provider
const provider = sdk.makeWeb3Provider({options: 'smartWalletOnly'}); // smartWalletOnly, eoaOnly, all
// Use provider
const addresses = provider.request({method: 'eth_requestAccounts'});
```

- Using WAGMI: https://www.smartwallet.dev/guides/create-app/using-wagmi#update-the-wagmi-config-and-coinbasewallet-connector-to-use-basesepolia

```JS
import { http, createConfig } from 'wagmi';
import { baseSepolia } from 'wagmi/chains';
import { coinbaseWallet } from 'wagmi/connectors';

export const config = createConfig({
  chains: [baseSepolia],
  connectors: [
    coinbaseWallet({ appName: 'Create Wagmi', preference: 'smartWalletOnly' }),
  ],
  transports: {
    [baseSepolia.id]: http(),
  },
});

declare module 'wagmi' {
  interface Register {
    config: typeof config;
  }
}
```

#### Notes: CoinbaseWalletSDK Changes (v3 to v4)

- Ref: https://www.smartwallet.dev/sdk/v3-to-v4-changes
- `appChainIds?: number[]` is supported only in the latest version of the SDK (v4).
- Deprecated in v4 (v3 only):

  - enableMobileWalletLink (enabled by default in v4)
  - jsonRpcUrl
  - reloadOnDisconnect
  - uiConstructor
  - overrideIsMetaMask
  - overrideIsCoinbaseWallet
  - diagnosticLogger
  - reloadOnDisconnect
  - headlessMode

- Deprecated functions:
  - CoinbaseWalletSDK.disconnect() is deprecated
    - dapps should call `CoinbaseWalletProvider.disconnect()` instead
  - CoinbaseWalletSDK.setAppInfo() is deprecated
    - Dapps should pass in appName and appLogoUrl via `CoinbaseWalletSDKOptions`

### Faucets

- Base Sepolia: https://sepolia.basescan.org/faucet

### Limitations

- Networks Supported: https://www.smartwallet.dev/FAQ#what-networks-are-supported
  - Base
  - Arbitrum
  - Optimism
  - Zora
  - Polygon
  - BNB
  - Avalanche
  - ETH mainnet (not preferred for use, due to gas cost)
- Networks Supported by Third Web - The bundler and paymaster services provided by thirdweb are supported on the following blockchains: https://support.thirdweb.com/connect/dwWCB7ZD5sNcHEAj4rFFui/account-abstraction-faqs/64y68nzTQkUZw6r6FryFgK
  - Polygon
  - Optimism
  - Base
  - Arbitrum
  - Linea
  - Goerli
  - Sepolia
  - Mumbai
  - Base Goerli
  - Optimism Goerli
  - Arbitrum Goerli
  - Arbitrum Sepolia
  - Linea Testnet
  - Celo Alfajores Testnet
- Supported Networks by WAGMI - https://wagmi.sh/react/api/chains#available-chains
- https://keys.coinbase.com/sign-in doesn't support custom networks, as confirmed by community.
- Requires PAYMASTER_SERVICE_URL | PAYMASTER_PROXY_SERVER_URL:

```JS
import { createClient, createPublicClient, http } from "viem";
import { baseSepolia } from "viem/chains";
import { ENTRYPOINT_ADDRESS_V06 } from "permissionless";
import { paymasterActionsEip7677 } from "permissionless/experimental";

export const client = createPublicClient({
  chain: baseSepolia,
  transport: http(),
});

const paymasterService = process.env.PAYMASTER_SERVICE_URL!;

export const paymasterClient = createClient({
  chain: baseSepolia,
  transport: http(paymasterService),
}).extend(paymasterActionsEip7677({ entryPoint: ENTRYPOINT_ADDRESS_V06 }));

```

### Making Signatures and Verifying Onchain (Wagmi)

```JS
import { useAccount, usePublicClient, useSignMessage } from "wagmi";
import { SiweMessage } from "siwe";

// message
const message = useMemo(() => {
    return new SiweMessage({
      domain: document.location.host,
      address: account.address,
      chainId: account.chainId,
      uri: document.location.origin,
      version: "1",
      statement: "Smart Wallet SIWE Example",
      nonce: "12345678",
    });
  }, []);

// sign message
const { signMessage } = useSignMessage({
    mutation: { onSuccess: (sig) => setSignatureState(sig) },
  });
<button onClick={() => signMessage({ message: message.prepareMessage() })}>
    Sign
</button>

// verify signature offchain
const checkValid = useCallback(async () => {
    if (!signature || !account.address) return;

    client
      .verifyMessage({
        address: account.address,
        message: message.prepareMessage(),
        signature,
      })
      .then((v) => setValidState(v));
  }, [signature, account]);

```

### Paymaster Paying the Gas Fee (WAGMI)

```JS
import { useCapabilities, useWriteContracts } from "wagmi/experimental";

  const { data: availableCapabilities } = useCapabilities({
    account: account.address,
  });

// // check if n/w has paymaster capabilities
  const capabilities = useMemo(() => {
    if (!availableCapabilities || !account.chainId) return;
    const capabilitiesForChain = availableCapabilities[account.chainId];
    if (
      capabilitiesForChain["paymasterService"] &&
      capabilitiesForChain["paymasterService"].supported
    ) {
      return {
        paymasterService: {
          url: process.env.PAYMASTER_PROXY_SERVER_URL || `${document.location.origin}/api/paymaster`,
        },
      };
    }
  }, [availableCapabilities]);
const capabilities = useMemo(() => {
    if (!availableCapabilities || !account.chainId) return;
    const capabilitiesForChain = availableCapabilities[account.chainId];
    if (
      capabilitiesForChain["paymasterService"] &&
      capabilitiesForChain["paymasterService"].supported
    ) {
      return {
        paymasterService: {
          url: process.env.PAYMASTER_PROXY_SERVER_URL || `${document.location.origin}/api/paymaster`,
        },
      };
    }
  }, [availableCapabilities]);

<button
    onClick={() => {
    writeContracts({
        contracts: [
        {
            address: myNFTAddress,
            abi: myNFTABI,
            functionName: "safeMint",
            args: [account.address],
        },
        ],
        capabilities,
    });
    }}
>
    Mint
</button>
```

### Paymaster Service | pm_ Namespace

- https://www.erc7677.xyz/reference/paymasters/getPaymasterStubData
  - pm_getPaymasterStubData
  - pm_getPaymasterData
