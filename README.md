# MozLink

MozLink is a backend system for managing multiple businesses (e.g., scrap, commodities, takeaway) under one umbrella, 
starting with a scrap company use case. The system supports branches, clients, material tracking, financial balances, 
expense approvals, fund requests, and reporting.

## Features

- **Multi-company & multi-branch support**
- **Wallet balances** (Cash, Emola, M-Pesa) per branch
- **Client & material tracking**
- **Scrap purchase recording** with quantity, price, and total
- **Expense approval workflow**
- **Fund request workflow**
- **Material transfer tracking** between branches and main yard
- **Weekly, monthly, and quarterly reports** for balances and material movement
- **Audit-ready transactional consistency**

## Tech Stack

- **Node.js** (Express / NestJS)
- **PostgreSQL** (Primary database)
- **Prisma ORM** (Database toolkit)
- **Docker** (Optional, for local Postgres instance)

## Project Structure

```
mozlink/
├── prisma/
│   ├── schema.prisma       # Database schema
│   └── migrations/         # Migration history
├── src/
│   ├── db.ts               # Prisma client
│   ├── index.ts            # App entry point
│   ├── routes/             # API routes
│   ├── controllers/        # Request handlers
│   ├── services/           # Business logic
│   ├── middlewares/        # Middleware functions
│   └── utils/              # Helpers
├── package.json
├── .env
└── README.md
```

## Setup Instructions

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/mozlink.git
cd mozlink
```

### 2. Install dependencies
```bash
npm install
```

### 3. Configure environment variables
Create a `.env` file:
```
DATABASE_URL="postgresql://user:password@localhost:5432/mozlinkdb?schema=public"
```

### 4. Run PostgreSQL (Docker)
```bash
docker run --name mozlink-pg -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=mozlinkdb -p 5432:5432 -d postgres:16
```

### 5. Apply Prisma migrations
```bash
npx prisma migrate dev --name init
npx prisma generate
```

### 6. Run the server
```bash
npm run dev
```

## Example Commands

Create a company:
```ts
await prisma.company.create({ data: { name: "ScrapCo" } });
```

Create a branch with initial wallet balances:
```ts
const branch = await prisma.branch.create({
  data: { name: "Matola", companyId: companyId },
});

await prisma.balance.createMany({
  data: [
    { branchId: branch.id, walletType: "CASH", amount: 0 },
    { branchId: branch.id, walletType: "EMOLA", amount: 0 },
    { branchId: branch.id, walletType: "MPESA", amount: 0 },
  ],
});
```

## Reporting Examples

Weekly scrap purchase totals per branch:
```sql
SELECT b.name AS branch,
       date_trunc('week', sp.ts) AS week,
       SUM(sp.total)::numeric(14,2) AS purchase_total
FROM "ScrapPurchase" sp
JOIN "Branch" b ON b.id = sp."branchId"
WHERE b."companyId" = $1
GROUP BY b.name, date_trunc('week', sp.ts)
ORDER BY week DESC;
```

