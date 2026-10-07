


# Azure CLI (Mac)
brew install azure-cli

# Azure CLI (Windows)
winget install Microsoft.AzureCLI

# Azure CLI (Linux)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Verify
node --version   # v20.x.x
git --version    # git version 2.x
docker --version # Docker version 24.x
az --version     # azure-cli 2.x

# create project folder
mkdir skywings-airways

# 1. Root folder-এ আছেন কিনা check
pwd
# /Users/Basanta/skywings-airways হলে ঠিক আছে

# 2. apps/web folder বানান
mkdir -p apps/web

# 3. সেখানে যান
cd apps/web

# 4. Next.js install (dot দিয়ে)
npx create-next-app@latest . --typescript --tailwind --eslint --app --no-src-dir --import-alias "@/*"

# 5. Prompt আসলে y চাপুন
# 6. বাকি options default রাখুন (Enter)

# Verify it works
npm run dev

# 7. Verify
ls -la

# app/, package.json, next.config.ts ইত্যাদি দেখবেন

# Step 1.2: Local PostgreSQL Setup
# Project root-এ docker-compose.yml বানান:

version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: skywings-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: skywings
      POSTGRES_PASSWORD: skywings_dev_password
      POSTGRES_DB: skywings_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    container_name: skywings-redis
    restart: unless-stopped
    ports:
      - "6379:6379"

volumes:
  postgres_data:

# Project root থেকে
cd ..   # apps/web থেকে বেরিয়ে

docker compose up -d

# Verify
docker ps
# skywings-db এবং skywings-redis running দেখা উচিত

cd apps/web

# Install Prisma
npm install prisma@6 @prisma/client@6
npm install -D @types/node
npx prisma --version

# Initialize Prisma
npx prisma init --datasource-provider postgresql

# edit .env
nano .env
# add inside .env
DATABASE_URL="postgresql://skywings:skywings_dev_password@localhost:5432/skywings_db?schema=public"
NEXTAUTH_SECRET="skywings-super-secret-key-min-32-chars-long-change-me"
NEXTAUTH_URL="http://localhost:3000"

# edit prisma.config.ts
vi prisma.config.ts

# add this inside prisma.config.ts

import 'dotenv/config';
import { defineConfig } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
});

# install dotenv

npm install dotenv

# update prisma/schema.prisma

>prisma/schema.prisma
vi prisma/schema.prisma

# পুরো content মুছে এইটা paste করুন:

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Flight {
  id             String    @id @default(cuid())
  flightNumber   String    @unique
  airline        String
  fromCode       String
  fromCity       String
  toCode         String
  toCity         String
  departureTime  DateTime
  arrivalTime    DateTime
  basePrice      Decimal   @db.Decimal(10, 2)
  currency       String    @default("INR")
  seatsAvailable Int
  aircraft       String
  createdAt      DateTime  @default(now())
  bookings       Booking[]

  @@index([fromCode, toCode, departureTime])
}

model Booking {
  id             String   @id @default(cuid())
  pnr            String   @unique
  flightId       String
  flight         Flight   @relation(fields: [flightId], references: [id])
  passengerName  String
  passengerEmail String
  passengerPhone String
  totalPrice     Decimal  @db.Decimal(10, 2)
  currency       String   @default("INR")
  status         String   @default("CONFIRMED")
  createdAt      DateTime @default(now())

  @@index([passengerEmail])
  @@index([pnr])
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  locale    String   @default("in")
  createdAt DateTime @default(now())
}

# Prisma Generate
npx prisma generate

# Migration চালান
npx prisma migrate dev --name init

# Prisma Studio-তে Verify
npx prisma studio

# Seed Data — Fake Flights যোগ করুন
vi prisma/seed.ts
# add contant to prisma/seed.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  console.log('🌱 Seeding flights...');

  const flights = [
    // Delhi → Mumbai
    {
      flightNumber: 'SW101',
      airline: 'SkyWings',
      fromCode: 'DEL', fromCity: 'Delhi',
      toCode: 'BOM', toCity: 'Mumbai',
      departureTime: new Date('2026-12-01T06:00:00Z'),
      arrivalTime: new Date('2026-12-01T08:15:00Z'),
      basePrice: 4500, currency: 'INR',
      seatsAvailable: 120, aircraft: 'Airbus A320',
    },
    {
      flightNumber: 'SW102',
      airline: 'SkyWings',
      fromCode: 'DEL', fromCity: 'Delhi',
      toCode: 'BOM', toCity: 'Mumbai',
      departureTime: new Date('2026-12-01T14:30:00Z'),
      arrivalTime: new Date('2026-12-01T16:45:00Z'),
      basePrice: 5200, currency: 'INR',
      seatsAvailable: 85, aircraft: 'Boeing 737',
    },
    // Mumbai → Goa
    {
      flightNumber: 'SW201',
      airline: 'SkyWings',
      fromCode: 'BOM', fromCity: 'Mumbai',
      toCode: 'GOA', toCity: 'Goa',
      departureTime: new Date('2026-12-02T09:00:00Z'),
      arrivalTime: new Date('2026-12-02T10:15:00Z'),
      basePrice: 2800, currency: 'INR',
      seatsAvailable: 150, aircraft: 'ATR 72',
    },
    // Delhi → Bangalore
    {
      flightNumber: 'SW301',
      airline: 'SkyWings',
      fromCode: 'DEL', fromCity: 'Delhi',
      toCode: 'BLR', toCity: 'Bangalore',
      departureTime: new Date('2026-12-01T10:00:00Z'),
      arrivalTime: new Date('2026-12-01T12:45:00Z'),
      basePrice: 6200, currency: 'INR',
      seatsAvailable: 95, aircraft: 'Airbus A321',
    },
    // Bangalore → Mumbai
    {
      flightNumber: 'SW401',
      airline: 'SkyWings',
      fromCode: 'BLR', fromCity: 'Bangalore',
      toCode: 'BOM', toCity: 'Mumbai',
      departureTime: new Date('2026-12-03T08:30:00Z'),
      arrivalTime: new Date('2026-12-03T10:30:00Z'),
      basePrice: 3800, currency: 'INR',
      seatsAvailable: 110, aircraft: 'Airbus A320',
    },
    // Mumbai → Delhi
    {
      flightNumber: 'SW501',
      airline: 'SkyWings',
      fromCode: 'BOM', fromCity: 'Mumbai',
      toCode: 'DEL', toCity: 'Delhi',
      departureTime: new Date('2026-12-04T18:00:00Z'),
      arrivalTime: new Date('2026-12-04T20:15:00Z'),
      basePrice: 4800, currency: 'INR',
      seatsAvailable: 75, aircraft: 'Boeing 737',
    },
  ];

  for (const flight of flights) {
    await prisma.flight.upsert({
      where: { flightNumber: flight.flightNumber },
      update: {},
      create: flight,
    });
  }

  console.log(`✅ Seeded ${flights.length} flights`);
}

main()
  .catch((e) => {
    console.error('❌ Seed error:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });

# tsx Install করুন
npm install -D tsx

# package.json এ "scripts" section-এ "lint": "eslint" line-এর পরে comma দিয়ে এই line যোগ করুন

  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "seed": "tsx prisma/seed.ts"
  },
  # ⚠️ গুরুত্বপূর্ণ: "lint": "eslint" line-এর শেষে , (comma) দিতে ভুলবেন না।

  # Verify করুন JSON valid:
  node -e "console.log(JSON.parse(require('fs').readFileSync('package.json')))"

  #  Seed চালান
  npm run seed

  # 📌 Step Flight Search API
  # API folder বানান এবং Search endpoint লিখুন
  mkdir -p app/api/flights/search
  mkdir -p app/api/bookings
  # Search API file বানান
  vi app/api/flights/search/route.ts
  # ei contant ta paste koro app/api/flights/search/route.ts te

import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const from = searchParams.get('from')?.toUpperCase();
  const to = searchParams.get('to')?.toUpperCase();
  const date = searchParams.get('date');

  if (!from || !to) {
    return NextResponse.json(
      { error: 'from and to parameters are required' },
      { status: 400 }
    );
  }

  try {
    const flights = await prisma.flight.findMany({
      where: {
        fromCode: from,
        toCode: to,
        ...(date && {
          departureTime: {
            gte: new Date(date),
            lt: new Date(new Date(date).getTime() + 24 * 60 * 60 * 1000),
          },
        }),
      },
      orderBy: { departureTime: 'asc' },
    });

    return NextResponse.json({
      success: true,
      count: flights.length,
      flights,
    });
  } catch (error) {
    console.error('Search error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

# Booking API file বানান
vi app/api/bookings/route.ts
# এই content paste করুন:
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

function generatePNR(): string {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let pnr = 'SW';
  for (let i = 0; i < 6; i++) {
    pnr += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return pnr;
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { flightId, passengerName, passengerEmail, passengerPhone } = body;

    // Validation
    if (!flightId || !passengerName || !passengerEmail) {
      return NextResponse.json(
        { error: 'flightId, passengerName, and passengerEmail are required' },
        { status: 400 }
      );
    }

    // Check flight exists
    const flight = await prisma.flight.findUnique({ where: { id: flightId } });
    if (!flight) {
      return NextResponse.json({ error: 'Flight not found' }, { status: 404 });
    }

    if (flight.seatsAvailable < 1) {
      return NextResponse.json(
        { error: 'No seats available on this flight' },
        { status: 400 }
      );
    }

    // Transaction: create booking + decrement seats
    const booking = await prisma.$transaction(async (tx) => {
      const newBooking = await tx.booking.create({
        data: {
          pnr: generatePNR(),
          flightId,
          passengerName,
          passengerEmail,
          passengerPhone: passengerPhone || '',
          totalPrice: flight.basePrice,
          currency: flight.currency,
          status: 'CONFIRMED',
        },
      });

      await tx.flight.update({
        where: { id: flightId },
        data: { seatsAvailable: { decrement: 1 } },
      });

      return newBooking;
    });

    return NextResponse.json({
      success: true,
      message: 'Booking confirmed!',
      booking,
    });
  } catch (error) {
    console.error('Booking error:', error);
    return NextResponse.json(
      { error: 'Booking failed. Please try again.' },
      { status: 500 }
    );
  }
}

export async function GET() {
  try {
    const bookings = await prisma.booking.findMany({
      include: { flight: true },
      orderBy: { createdAt: 'desc' },
      take: 20,
    });

    return NextResponse.json({
      success: true,
      count: bookings.length,
      bookings,
    });
  } catch (error) {
    console.error('Bookings fetch error:', error);
    return NextResponse.json(
      { error: 'Failed to fetch bookings' },
      { status: 500 }
    );
  }
}

# Dev Server চালু করে Test করুন
npm run dev

# Test 1: Search API
# Delhi → Mumbai search
curl "http://localhost:3000/api/flights/search?from=DEL&to=BOM"

# Test 2: Booking API Search থেকে flight ID copy করুন
curl "http://localhost:3000/api/flights/search?from=DEL&to=BOM" | grep -o '"id":"[^"]*"' | head -1

# Output থেকে ID পাবেন, যেমন: "id":"clx123abc456"

# এখন booking চালান (ID replace করুন):

curl -X POST http://localhost:3000/api/bookings \
  -H "Content-Type: application/json" \
  -d '{
    "flightId": "এখানে-আসল-ID-বসান",
    "passengerName": "Basanta Das",
    "passengerEmail": "basanta@test.com",
    "passengerPhone": "+919999999999"
  }'

 # All Flights (Database Check)
  curl http://localhost:3000/api/flights/search?from=BOM&to=GOA
  # Bookings List
  curl http://localhost:3000/api/bookings

  # Home Page বানান
  cd /Users/Basanta/skywings-airways/apps/web
  vi app/page.tsx

  # পুরো content মুছে এইটা paste করুন:

  import Link from 'next/link';

export default function Home() {
  return (
    <main className="min-h-screen bg-gradient-to-b from-sky-400 to-blue-600">
      {/* Navbar */}
      <nav className="bg-white shadow-md">
        <div className="max-w-6xl mx-auto px-4 py-4 flex justify-between items-center">
          <h1 className="text-2xl font-bold text-blue-600">✈️ SkyWings Airways</h1>
          <div className="space-x-6">
            <Link href="/search" className="text-gray-700 hover:text-blue-600 font-medium">
              Search Flights
            </Link>
            <Link href="/login" className="text-gray-700 hover:text-blue-600 font-medium">
              Login
            </Link>
          </div>
        </div>
      </nav>

      {/* Hero */}
      <section className="max-w-6xl mx-auto px-4 py-24 text-center text-white">
        <h2 className="text-5xl md:text-6xl font-bold mb-6">
          Fly the World with SkyWings
        </h2>
        <p className="text-xl md:text-2xl mb-10 opacity-90">
          Book flights to 50+ destinations worldwide
        </p>
        <Link
          href="/search"
          className="inline-block bg-white text-blue-600 px-10 py-4 rounded-lg font-semibold text-lg hover:bg-gray-100 transition shadow-lg"
        >
          Search Flights →
        </Link>
      </section>

      {/* Features */}
      <section className="max-w-6xl mx-auto px-4 pb-20 grid grid-cols-1 md:grid-cols-3 gap-6">
        {[
          { icon: '🛡️', title: 'Safe Travel', desc: 'IATA certified airline with global safety standards' },
          { icon: '💰', title: 'Best Prices', desc: 'AI-powered dynamic pricing for the best deals' },
          { icon: '🌍', title: '24/7 Support', desc: 'AI chatbot in 10+ languages, always available' },
        ].map((f) => (
          <div key={f.title} className="bg-white p-8 rounded-lg shadow-md text-center">
            <div className="text-4xl mb-4">{f.icon}</div>
            <h3 className="font-bold text-xl mb-2 text-gray-800">{f.title}</h3>
            <p className="text-gray-600">{f.desc}</p>
          </div>
        ))}
      </section>
    </main>
  );
}

# Step 2: Search Page বানান
# app/search/page.tsx file বানান:
mkdir -p app/search
vi app/search/page.tsx
# ei contant paste korun 
'use client';

import { useState } from 'react';
import Link from 'next/link';

interface Flight {
  id: string;
  flightNumber: string;
  airline: string;
  fromCode: string;
  fromCity: string;
  toCode: string;
  toCity: string;
  departureTime: string;
  arrivalTime: string;
  basePrice: string;
  currency: string;
  aircraft: string;
  seatsAvailable: number;
}

export default function SearchPage() {
  const [from, setFrom] = useState('DEL');
  const [to, setTo] = useState('BOM');
  const [date, setDate] = useState('2026-12-01');
  const [flights, setFlights] = useState<Flight[]>([]);
  const [loading, setLoading] = useState(false);
  const [searched, setSearched] = useState(false);

  const searchFlights = async () => {
    setLoading(true);
    setSearched(true);
    try {
      const res = await fetch(
        `/api/flights/search?from=${from}&to=${to}&date=${date}`
      );
      const data = await res.json();
      setFlights(data.flights || []);
    } catch (err) {
      console.error(err);
      setFlights([]);
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white shadow-md">
        <div className="max-w-6xl mx-auto px-4 py-4">
          <Link href="/" className="text-2xl font-bold text-blue-600">
            ✈️ SkyWings
          </Link>
        </div>
      </nav>

      <div className="max-w-6xl mx-auto p-6">
        {/* Search Form */}
        <div className="bg-white p-6 rounded-lg shadow-md mb-6">
          <h2 className="text-2xl font-bold mb-4 text-gray-800">Search Flights</h2>
          <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">From</label>
              <input
                value={from}
                onChange={(e) => setFrom(e.target.value.toUpperCase())}
                placeholder="DEL"
                maxLength={3}
                className="w-full border border-gray-300 p-3 rounded focus:outline-none focus:border-blue-500"
              />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">To</label>
              <input
                value={to}
                onChange={(e) => setTo(e.target.value.toUpperCase())}
                placeholder="BOM"
                maxLength={3}
                className="w-full border border-gray-300 p-3 rounded focus:outline-none focus:border-blue-500"
              />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Date</label>
              <input
                type="date"
                value={date}
                onChange={(e) => setDate(e.target.value)}
                className="w-full border border-gray-300 p-3 rounded focus:outline-none focus:border-blue-500"
              />
            </div>
            <div className="flex items-end">
              <button
                onClick={searchFlights}
                disabled={loading}
                className="w-full bg-blue-600 text-white py-3 rounded font-semibold hover:bg-blue-700 disabled:opacity-50 transition"
              >
                {loading ? 'Searching...' : 'Search'}
              </button>
            </div>
          </div>
          <p className="text-sm text-gray-500 mt-3">
            Try: DEL → BOM, BOM → GOA, DEL → BLR, BLR → BOM
          </p>
        </div>

        {/* Results */}
        <div className="space-y-4">
          {loading && (
            <div className="text-center py-12 text-gray-500">Loading flights...</div>
          )}

          {!loading && searched && flights.length === 0 && (
            <div className="bg-white p-8 rounded-lg shadow-md text-center text-gray-500">
              No flights found for this route. Try DEL → BOM.
            </div>
          )}

          {!loading &&
            flights.map((f) => (
              <div
                key={f.id}
                className="bg-white p-6 rounded-lg shadow-md flex flex-col md:flex-row justify-between items-center gap-4"
              >
                <div className="flex items-center gap-4">
                  <div className="bg-blue-100 text-blue-700 font-bold px-3 py-2 rounded">
                    {f.flightNumber}
                  </div>
                  <div>
                    <div className="font-semibold text-gray-800">{f.airline}</div>
                    <div className="text-sm text-gray-500">{f.aircraft}</div>
                  </div>
                </div>

                <div className="flex items-center gap-6">
                  <div className="text-center">
                    <div className="font-bold text-xl text-gray-800">
                      {new Date(f.departureTime).toLocaleTimeString('en-IN', {
                        hour: '2-digit',
                        minute: '2-digit',
                        hour12: false,
                      })}
                    </div>
                    <div className="text-gray-500 text-sm">{f.fromCode}</div>
                  </div>
                  <div className="text-gray-400 text-xl">→</div>
                  <div className="text-center">
                    <div className="font-bold text-xl text-gray-800">
                      {new Date(f.arrivalTime).toLocaleTimeString('en-IN', {
                        hour: '2-digit',
                        minute: '2-digit',
                        hour12: false,
                      })}
                    </div>
                    <div className="text-gray-500 text-sm">{f.toCode}</div>
                  </div>
                </div>

                <div className="text-right">
                  <div className="font-bold text-2xl text-blue-600">
                    ₹{parseInt(f.basePrice).toLocaleString('en-IN')}
                  </div>
                  <div className="text-xs text-gray-500 mb-2">
                    {f.seatsAvailable} seats left
                  </div>
                  <Link
                    href={`/booking/${f.id}`}
                    className="inline-block bg-green-600 text-white px-6 py-2 rounded font-semibold hover:bg-green-700 transition"
                  >
                    Book Now
                  </Link>
                </div>
              </div>
            ))}
        </div>
      </div>
    </main>
  );
}

# Step 3: Booking Page বানান
# app/booking/[id]/page.tsx file বানান then paste below contant:

'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import Link from 'next/link';

interface Flight {
  id: string;
  flightNumber: string;
  airline: string;
  fromCode: string;
  fromCity: string;
  toCode: string;
  toCity: string;
  departureTime: string;
  arrivalTime: string;
  basePrice: string;
  currency: string;
  aircraft: string;
}

export default function BookingPage({ params }: { params: Promise<{ id: string }> }) {
  const router = useRouter();
  const [flightId, setFlightId] = useState<string>('');
  const [flight, setFlight] = useState<Flight | null>(null);
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [phone, setPhone] = useState('');
  const [loading, setLoading] = useState(false);
  const [loadingFlight, setLoadingFlight] = useState(true);

  // Resolve params (Next.js 15+ async params)
  useEffect(() => {
    params.then((p) => setFlightId(p.id));
  }, [params]);

  // Fetch flight details
  useEffect(() => {
    if (!flightId) return;
    const fetchFlight = async () => {
      try {
        const res = await fetch(`/api/flights/search?from=DEL&to=BOM`);
        const data = await res.json();
        const found = data.flights?.find((f: Flight) => f.id === flightId);
        setFlight(found || null);
      } catch (err) {
        console.error(err);
      } finally {
        setLoadingFlight(false);
      }
    };
    fetchFlight();
  }, [flightId]);

  const confirmBooking = async () => {
    if (!name || !email) {
      alert('Please fill name and email');
      return;
    }
    setLoading(true);
    try {
      const res = await fetch('/api/bookings', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          flightId,
          passengerName: name,
          passengerEmail: email,
          passengerPhone: phone,
        }),
      });
      const data = await res.json();
      if (data.success) {
        alert(`✅ Booking Confirmed!\n\nPNR: ${data.booking.pnr}\nPassenger: ${data.booking.passengerName}\nTotal: ₹${data.booking.totalPrice}`);
        router.push('/');
      } else {
        alert('❌ ' + (data.error || 'Booking failed'));
      }
    } catch (err) {
      alert('❌ Booking failed. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  if (loadingFlight) {
    return (
      <main className="min-h-screen bg-gray-50 flex items-center justify-center">
        <div className="text-gray-500">Loading flight details...</div>
      </main>
    );
  }

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white shadow-md">
        <div className="max-w-6xl mx-auto px-4 py-4">
          <Link href="/" className="text-2xl font-bold text-blue-600">
            ✈️ SkyWings
          </Link>
        </div>
      </nav>

      <div className="max-w-2xl mx-auto p-6">
        {/* Flight Summary */}
        {flight && (
          <div className="bg-gradient-to-r from-blue-500 to-blue-600 text-white p-6 rounded-lg shadow-md mb-6">
            <div className="flex justify-between items-center mb-4">
              <div>
                <div className="text-sm opacity-90">{flight.airline}</div>
                <div className="font-bold text-xl">{flight.flightNumber}</div>
              </div>
              <div className="text-sm opacity-90">{flight.aircraft}</div>
            </div>
            <div className="flex justify-between items-center">
              <div>
                <div className="text-3xl font-bold">
                  {new Date(flight.departureTime).toLocaleTimeString('en-IN', {
                    hour: '2-digit',
                    minute: '2-digit',
                    hour12: false,
                  })}
                </div>
                <div className="opacity-90">{flight.fromCity} ({flight.fromCode})</div>
              </div>
              <div className="text-2xl">✈️</div>
              <div className="text-right">
                <div className="text-3xl font-bold">
                  {new Date(flight.arrivalTime).toLocaleTimeString('en-IN', {
                    hour: '2-digit',
                    minute: '2-digit',
                    hour12: false,
                  })}
                </div>
                <div className="opacity-90">{flight.toCity} ({flight.toCode})</div>
              </div>
            </div>
            <div className="mt-4 pt-4 border-t border-white/30 text-right">
              <div className="text-sm opacity-90">Total Price</div>
              <div className="text-2xl font-bold">
                ₹{parseInt(flight.basePrice).toLocaleString('en-IN')}
              </div>
            </div>
          </div>
        )}

        {/* Passenger Form */}
        <div className="bg-white p-8 rounded-lg shadow-md">
          <h1 className="text-2xl font-bold mb-6 text-gray-800">Passenger Details</h1>

          <div className="space-y-4">
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">
                Full Name *
              </label>
              <input
                value={name}
                onChange={(e) => setName(e.target.value)}
                placeholder="e.g., Basanta Das"
                className="w-full border border-gray-300 p-3 rounded focus:outline-none focus:border-blue-500"
              />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">
                Email *
              </label>
              <input
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                placeholder="you@example.com"
                className="w-full border border-gray-300 p-3 rounded focus:outline-none focus:border-blue-500"
              />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">
                Phone
              </label>
              <input
                value={phone}
                onChange={(e) => setPhone(e.target.value)}
                placeholder="+91 9999999999"
                className="w-full border border-gray-300 p-3 rounded focus:outline-none focus:border-blue-500"
              />
            </div>
          </div>

          <button
            onClick={confirmBooking}
            disabled={loading || !name || !email}
            className="w-full mt-6 bg-green-600 text-white py-3 rounded font-semibold hover:bg-green-700 disabled:opacity-50 transition"
          >
            {loading ? 'Confirming...' : `Confirm Booking${flight ? ' — ₹' + parseInt(flight.basePrice).toLocaleString('en-IN') : ''}`}
          </button>

          <p className="text-xs text-gray-500 mt-3 text-center">
            * This is a demo project. No real payment will be processed.
          </p>
        </div>
      </div>
    </main>
  );
}