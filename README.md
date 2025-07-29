# Next.js + Express + SQLite Caching Example

This project demonstrates a simple messaging app with a Next.js frontend and separate Express backend using SQLite for storage. It showcases caching strategies with Next.js app router caching (`revalidate`, cache tags) and efficient database access with `better-sqlite3`, including use of Next.js's `unstable_cache` for memoization.

---

## Project Structure

```

├── app # Next.js frontend app directory
│ ├── globals.css
│ ├── icon.png
│ ├── layout.js
│ ├── messages
│ │ ├── layout.js
│ │ ├── new
│ │ │ └── page.js
│ │ └── page.js
│ └── page.js
├── backend # Express backend
│ ├── app.js
│ ├── package.json
│ └── package-lock.json
├── components # React components
│ ├── header.js
│ └── messages.js
├── data # SQLite database file
│ └── messages.db
├── lib # Server utilities and DB functions
│ └── messages.js
├── package.json
├── next.config.mjs
├── README.md

```

---

## Getting Started

### Prerequisites

- Node.js (v16 or newer recommended)
- npm or yarn

### Install Dependencies

Install backend dependencies:

```

cd backend
npm install

```

Install frontend dependencies:

```

cd ../
npm install

```

### Run Backend Express Server

```

cd backend
npm start

```

Runs on http://localhost:8080 and provides the `/messages` API.

### Run Next.js Frontend

```

npm run dev

```

Runs on http://localhost:3000.

---

## Database

- SQLite DB file at `data/messages.db`.
- Initializes schema automatically via `lib/messages.js`.
- `messages` table schema:

| Column | Type    | Description                 |
| ------ | ------- | --------------------------- |
| id     | INTEGER | Primary key, auto-increment |
| text   | TEXT    | Message content             |

---

## API & Data Access

- Backend exposes message API with Express.
- Frontend consumes API or accesses DB directly.
- Use `addMessage(message)` to insert.
- Use `getMessages()` to query messages from DB.

---

## Caching Strategies

- Incremental Static Regeneration (ISR) with:

```

export const revalidate = 5; // revalidates cached page every 5 seconds

```

- Cache invalidation using Next.js cache tags on fetch requests.

- **React caching utilities:**

- Previously used `cache()` from React to memoize sync DB calls.
- **Now using Next.js `unstable_cache` for better control and async support:**

  - `unstable_cache` allows memoizing async functions with cache keys and integrates with Next.js cache invalidation.

- Use `unstable_noStore()` or `export const dynamic = 'force-dynamic'` to disable caching as needed.

---

## Updated Example DB Access Layer with `unstable_cache` (`lib/messages.js`)

```

import sql from "better-sqlite3";
import { unstable_cache } from "next/cache";

const db = new sql("data/messages.db");

function initDb() {
db.exec(`  CREATE TABLE IF NOT EXISTS messages (
      id INTEGER PRIMARY KEY,
      text TEXT
    )`);
}

initDb();

export function addMessage(message) {
db.prepare("INSERT INTO messages (text) VALUES (?)").run(message);
}

export const getMessages = unstable_cache(async () => {
console.log("Fetching messages from db via unstable_cache");
return db.prepare("SELECT \* FROM messages").all();
}, ["messages"]);

```

Usage in Next.js page component with ISR:

```

import Messages from "@/components/messages";
import { getMessages } from "@/lib/messages";

export const revalidate = 5;

export default async function MessagesPage() {
const messages = await getMessages();

if (!messages.length) {
return No messages found;
}

return ;
}

```

---

## License

This project is licensed under the [MIT License](LICENSE).
