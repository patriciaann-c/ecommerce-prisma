# 🛍️ E-Commerce Database with Prisma ORM

Hey there! 👋 Welcome to my **E-Commerce Prisma** project! 

This repository is a hands-on exploration of relational database modeling for a modern e-commerce backend. Built using **Prisma ORM 7** and **MySQL / MariaDB**, it showcases how to design a solid, scalable, and type-safe database schema from scratch—step by step, migration by migration.

Whether you're curious about how Prisma manages complex entity relationships or looking for a clean reference schema for an online shop, you're in the right place! ✨

---

## 🌟 Highlights of the Project

- **📐 Complete E-Commerce Domain**: Models catalog products, categories, tags, customer accounts, profiles, and order history.
- **🔗 All Relationship Types Covered**:
  - **1-to-1**: User ↔ Profile (optional profile with unique foreign key constraint)
  - **1-to-Many**: Category ↔ Products
  - **Many-to-Many (Explicit)**: Products ↔ Tags (with composite unique constraints)
  - **Order Lines**: Orders ↔ Products via `OrderProduct` pivot capturing historical snapshot prices.
- **🔄 Step-by-Step Versioned Migrations**: Built iteratively through 7 distinct migrations, documenting the evolution of the schema.
- **⚡ Modern Prisma 7 Features**: Utilizing `prisma.config.ts` configuration, shadow database verification, and `@prisma/adapter-mariadb` driver adapter with connection pooling.
- **🛡️ Data Integrity First**: Smart cascade behaviors (`onDelete: SetNull`), unique constraints, and enums (`OrderStatus`).

---

## 🗄️ Database Architecture & ER Diagram

Here is a quick visual overview of how all the pieces connect together:

```
  ┌──────────────┐                 ┌──────────────┐
  │   Category   │ 1 ─────────── * │   Product    │
  └──────────────┘                 └──────┬───────┘
                                          │
                   ┌──────────────────────┼──────────────────────┐
                   │ *                                         * │
            ┌──────┴──────┐                             ┌────────┴──────┐
            │ ProductTag  │                             │ OrderProduct  │
            └──────┬──────┘                             └────────┬──────┘
                   │ *                                         * │
                   │                                             │
            ┌──────┴──────┐                             ┌────────┴──────┐
            │     Tag     │                             │     Order     │
            └─────────────┘                             └────────┬──────┘
                                                                 │ *
  ┌──────────────┐                 ┌──────────────┐              │
  │   Profile    │ 1 ─────────── 1 │     User     │ 1 ───────────┘
  └──────────────┘                 └──────────────┘
```

---

## 🔍 Deep Dive into the Models

Here is how the schema is structured in [`prisma/schema.prisma`](prisma/schema.prisma) and why each decision was made:

### 1. `Product` 📦
The heart of our e-commerce catalog.
- **Fields**: `id`, `name`, `price` (Decimal 10,2 for precise monetary calculations), `description`, `createdAt`, `inStock` (default `true`), `isFeatured` (default `false`), and `categoryId`.
- **Why `SetNull` on Category?** If a category is ever removed (e.g., "Seasonal Specials"), we don't want all linked products to be accidentally wiped out! Instead, `categoryId` simply resets to `null`.

### 2. `Category` 📂
Groups products into logical departments (e.g., Electronics, Clothing).
- Has a `unique` constraint on `name` so categories aren't duplicated.
- Connects to `Product` via a **1-to-Many** relation.

### 3. `Tag` & `ProductTag` 🏷️
Allows products to have multiple flexible badges (e.g., `eco-friendly`, `new-arrival`, `bestseller`).
- We used an **explicit join table** (`ProductTag`) rather than an implicit many-to-many.
- **Pro Tip**: The composite constraint `@@unique([productId, tagId])` ensures you can never accidentally tag the exact same product with the same tag twice!

### 4. `User` & `Profile` 👤
Separating authentication credentials from rich user profile details.
- **`User`**: Holds core authentication info (`email` unique, `name`, `password`).
- **`Profile`**: Optional profile containing `bio` and `avatar`.
- **1-to-1 Relationship**: `Profile.userId` has `@unique`, guaranteeing each user has at most one profile, while keeping the user account lightweight.

### 5. `Order`, `OrderProduct` & `OrderStatus` 🛒
Handling customer purchases and carts.
- **`OrderStatus` Enum**: Cleanly tracks order progress:
  ```
  PENDING  ➜  PAID  ➜  SHIPPED  (or CANCELLED)
  ```
- **`OrderProduct` (The Order Line Secret 💡)**:
  Why not just read `Product.price` when viewing an old order? Because catalog prices change over time! 
  `OrderProduct` stores `quantity` and a snapshot `unitPrice` recorded at the exact moment of checkout. This ensures customer receipts and revenue reports remain 100% accurate forever.

---

## 📜 Migration Timeline: How the Schema Evolved

Rather than dumping a finished schema all at once, this project was developed incrementally using Prisma migrations:

| # | Migration | What was added |
|---|---|---|
| 1️⃣ | `init` | Created baseline models (`Product`, `User`, `Profile`) |
| 2️⃣ | `add_in_stock` | Added the `inStock` boolean flag to `Product` |
| 3️⃣ | `add_is_featured` | Added the `isFeatured` flag for storefront highlights |
| 4️⃣ | `add_user_profile` | Established the 1-to-1 relationship between `User` and `Profile` |
| 5️⃣ | `add_category` | Added `Category` model and 1-to-N relation with `Product` |
| 6️⃣ | `add_tags` | Added `Tag` and explicit `ProductTag` join table |
| 7️⃣ | `add_orders` | Added `Order`, `OrderProduct`, and the `OrderStatus` enum |

You can explore each migration's raw SQL inside the [`prisma/migrations/`](prisma/migrations/) directory.

---

## 🛠️ Tech Stack & Tools

- **ORM**: [Prisma 7](https://www.prisma.io/)
- **Database**: MySQL / MariaDB
- **Database Driver Adapter**: `@prisma/adapter-mariadb`
- **Runtime**: Node.js (ES Modules, `"type": "module"`)
- **Package Manager**: [pnpm](https://pnpm.io/)
- **Configuration**: `dotenv` + `prisma.config.ts`

---

## 🚀 Quickstart Guide

Want to run this project on your local machine? Follow these simple steps:

### 1. Prerequisites
- **Node.js** (v18+ recommended)
- **pnpm** installed (`npm i -g pnpm`)
- A local **MySQL** or **MariaDB** instance running (e.g. via XAMPP, Docker, or native service)

### 2. Clone & Install
```bash
# Clone the repository
git clone <your-repo-url>
cd ecommerce-prisma

# Install dependencies
pnpm install
```

### 3. Setup Environment Variables
Create your local `.env` file from the example template:
```bash
cp .env.example .env
```
Open `.env` and fill in your database credentials:
```env
DATABASE_URL="mysql://root:password@localhost:3306/ecommerce_prisma"
SHADOW_DATABASE_URL="mysql://root:password@localhost:3306/ecommerce_prisma_shadow"

DATABASE_HOST="127.0.0.1"
DATABASE_PORT="3306"
DATABASE_USER="root"
DATABASE_PASSWORD="password"
DATABASE_NAME="ecommerce_prisma"
```
> 💡 **Note on Shadow Database**: Prisma uses the shadow database during migrations to safely verify schema diffs. Make sure you create an empty database named `ecommerce_prisma_shadow` (or whichever name you prefer)!

### 4. Apply Migrations & Generate Client
```bash
# Run migrations to build the tables in your database
pnpm migrate

# Generate the type-safe Prisma Client
pnpm generate
```

### 5. Launch Prisma Studio 🎉
Browse, filter, and insert records through an awesome visual GUI in your browser:
```bash
pnpm studio
```

---

## 💻 Handy Commands Cheat Sheet

| Command | What it does |
|---|---|
| `pnpm validate` | Checks your `schema.prisma` file for syntax & relation errors |
| `pnpm generate` | Compiles your schema into the type-safe `@prisma/client` |
| `pnpm migrate` | Applies pending migrations in development (`prisma migrate dev`) |
| `pnpm reset` | Drops the database and replays all migrations from scratch |
| `pnpm studio` | Opens the web-based Prisma Studio data explorer |

---

## 🧑‍💻 How to Use the Client in Code

We have a pre-configured client ready to use in [`prisma-client.js`](prisma-client.js), leveraging connection pooling and the MariaDB driver adapter:

```javascript
import { client } from "./prisma-client.js";

// Fetch featured products with their category and tags included!
const featuredProducts = await client.product.findMany({
  where: {
    isFeatured: true,
    inStock: true
  },
  include: {
    category: true,
    tags: {
      include: {
        tag: true
      }
    }
  }
});

console.log("Featured items:", featuredProducts);
```

---

## 💭 Key Takeaways & Lessons Learned

1. **Explicit vs Implicit Many-to-Many**: Using `ProductTag` and `OrderProduct` explicitly gave us total control over metadata, such as unique constraints and purchasing snapshot prices.
2. **Prisma 7 Configuration**: In Prisma 7, connection URLs and shadow database settings live in `prisma.config.ts`, keeping `schema.prisma` purely declarative and environment-agnostic.
3. **Price Integrity**: Always capture unit prices in order lines to protect historical financial records when catalog prices update.

---

## 🤝 Feedback & Contributions

Feel free to fork this project, experiment with new models (like Reviews, Discounts, or Shipping Addresses), or open an issue if you have suggestions!

Happy coding! 🚀
