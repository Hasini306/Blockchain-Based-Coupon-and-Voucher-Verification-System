# Blockchain-Based Coupon and Voucher Verification System (BlockVoucher)

BlockVoucher is a full-stack demonstration system for creating, assigning, verifying, and redeeming digital coupons. It combines a React web application, an Express API, MongoDB application data, and an Ethereum smart contract. Coupon validity and redemption are checked on-chain, while MongoDB stores user, coupon, and redemption metadata.

## Technologies

- **Frontend:** React 18, React Router, Vite, Axios, QR code generation and scanning
- **Backend:** Node.js, Express, MongoDB with Mongoose, JWT authentication
- **Blockchain:** Solidity, Hardhat, ethers.js
- **Development and security middleware:** npm, Nodemon, Helmet, CORS

## Features

- Register and authenticate customer and merchant accounts; administer the system through a seeded admin account.
- Create coupons, assign them to customers, set expiry dates, and generate downloadable QR codes.
- Check coupon validity and redemption status through the Ethereum contract.
- Let authorized merchants scan or upload a customer's QR code to redeem a coupon.
- Prevent repeat redemption of a one-time coupon and record confirmed redemptions.
- Manage merchant authorization and review customer, merchant, transaction, and statistics views.
- Link a merchant's Ethereum wallet address and view transaction history.

## Requirements

- Node.js 20 or newer and npm
- MongoDB Community running locally, or a MongoDB connection URI
- MetaMask is optional for connecting a browser wallet. In the local demo, the API signs blockchain writes using a local Hardhat account.

## Installation and configuration

Run these commands from the repository root in PowerShell:

```powershell
npm install
npm --prefix backend install
npm --prefix frontend install
Copy-Item .env.example backend\.env
```

Edit `backend\.env` before starting the API:

- Set `JWT_SECRET` to a long, random secret.
- Keep the default local values for `PORT`, `MONGODB_URI`, and `BLOCKCHAIN_RPC_URL`, or adjust them for your environment.
- For the local demo, set `BLOCKCHAIN_PRIVATE_KEY` to the first Hardhat development account's private key and `DEMO_MERCHANT_PRIVATE_KEY` to the second account's private key.
- Configure `ADMIN_EMAIL` and `ADMIN_PASSWORD` for the admin account to be seeded. Use an admin password of at least 12 characters.

The demo merchant's Ethereum address must be linked to a merchant account and authorized by the admin. These Hardhat keys are public development keys for local demonstrations only: never use them on a public network. Keep all private keys in the backend environment file; never expose them through frontend variables or commit them to source control.

## Run the local demonstration

Start each long-running service in its own terminal, from the repository root:

1. Start a local Ethereum node:

   ```powershell
   npx hardhat node
   ```

2. In another terminal, compile and deploy the contract:

   ```powershell
   npm run compile
   npm run deploy:local
   ```

   The deploy script writes the contract ABI and address to `backend/config/contract.json`.

3. Start MongoDB. The default database URI is `mongodb://127.0.0.1:27017/blockvoucher`.
4. Start the API:

   ```powershell
   npm run dev:backend
   ```

5. In another terminal, create the admin account using `ADMIN_EMAIL` and `ADMIN_PASSWORD` from `backend\.env`:

   ```powershell
   npm run seed:admin
   ```

6. Start the frontend:

   ```powershell
   npm run dev:frontend
   ```

7. Open [http://localhost:5173](http://localhost:5173).

You can also start the backend and frontend together with `npm run dev`, after the local chain, contract deployment, MongoDB, and admin seed are ready.

Public registration creates customer or merchant accounts only. Register those accounts through the application; the repository does not include demo customer or merchant credentials.

## Working process

1. Register a customer and a merchant, then sign in as the seeded admin.
2. Create a coupon, choose the customer who will receive it, and set a future expiry. The system assigns a unique ID, registers the one-time coupon on-chain, and provides a QR code to download.
3. The assigned customer signs in to view the coupon and download its QR image. Coupons assigned to other customers are not displayed.
4. The merchant links an Ethereum address in the merchant dashboard. The admin authorizes the merchant, and that authorization is recorded in the contract.
5. Sign in as the merchant and upload or scan the customer's QR image. The API submits redemption using the configured local demo merchant signer. After the transaction confirms on-chain, the page reports **Coupon redeemed successfully**.
6. Upload the same QR image again. Since the one-time coupon has already been used, the contract rejects the second redemption and the page reports **Invalid — already used**.

The backend verifies the mined redemption event before it records redemption metadata. The demo merchant private key is used only by the backend and is never sent to the browser.

## API overview

The API defaults to `http://localhost:4000`. Protected endpoints require an `Authorization: Bearer <JWT>` header. Admin accounts are created only with the seed script.

| Method | Endpoint | Purpose / access |
| --- | --- | --- |
| `GET` | `/api/health` | API health check |
| `GET` | `/api/chain` | Contract address, ABI, and local chain ID |
| `POST` | `/api/auth/register` | Register a customer or merchant |
| `POST` | `/api/auth/login` | Sign in and obtain an authentication token |
| `GET` | `/api/auth/profile` | Get the authenticated user's profile |
| `GET` | `/api/coupons` | List coupons visible to the authenticated user |
| `GET` | `/api/coupons/:id` | Get a coupon |
| `POST` | `/api/coupons` | Create a coupon (admin) |
| `POST` | `/api/coupons/verify` | Verify a coupon (merchant) |
| `POST` | `/api/coupons/redeem` | Redeem a coupon (merchant) |
| `POST` | `/api/coupons/:id/deactivate` | Deactivate a coupon (admin) |
| `GET` | `/api/admin/merchants` | List merchants (admin) |
| `GET` | `/api/admin/customers` | List customers (admin) |
| `GET` | `/api/admin/transactions` | List transactions (admin) |
| `GET` | `/api/admin/stats` | Get dashboard statistics (admin) |
| `POST` | `/api/admin/authorize` | Authorize a merchant (admin) |
| `GET` | `/api/merchants` | List merchants (admin) |
| `POST` | `/api/merchants/authorize` | Authorize a merchant (admin) |
| `GET` | `/api/transactions` | List transactions (admin or merchant) |
| `PUT` | `/api/users/wallet` | Update the authenticated user's wallet |
| `POST` | `/api/contact` | Submit a contact message |

## Project structure

```text
BlockVoucher/
├── backend/
│   ├── config/          # MongoDB and blockchain configuration; deployed contract data
│   ├── controllers/     # Authentication, coupon, and admin logic
│   ├── middleware/      # Authentication and role checks
│   ├── models/          # User, coupon, redemption, and contact data models
│   ├── routes/          # Express API routes
│   ├── scripts/         # Admin seeding script
│   └── server.js        # API entry point
├── contracts/           # Solidity coupon verification contract
├── frontend/
│   └── src/             # React application, pages, components, styles, and API client
├── scripts/             # Local contract deployment script
├── test/                # Hardhat contract tests
├── hardhat.config.js
└── package.json         # Root scripts and Hardhat dependencies
```

## Testing

Run the smart contract tests and build the frontend from the repository root:

```powershell
npm run test:contract
npm --prefix frontend run build
```

The contract tests cover coupon creation, existence and expiry checks, merchant authorization, duplicate redemption, and admin-only operations.

## Security notes

- Never use a Hardhat development key on a public network.
- Do not put private keys in frontend configuration or commit them to source control.
- Set a unique, long `JWT_SECRET` before running the application.
- Admin accounts are provisioned with `npm run seed:admin`; public registration does not create admin accounts.
