---
name: supabase-integration
description: 'Integrate Supabase as a backend-as-a-service for authentication, real-time database, storage, and edge functions. Covers setup, Row Level Security policies, real-time subscriptions, and common application patterns with code examples.'
---

# Supabase Integration (ผสานรวม Supabase)

Use Supabase as your complete backend with authentication, database, storage, and realtime features.

## Initial Setup

```bash
# Install Supabase client
npm install @supabase/supabase-js

# For TypeScript type generation
npx supabase gen types typescript --project-id YOUR_PROJECT_ID > src/types/supabase.ts
```

```typescript
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from './types/supabase'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey)
```

## Authentication Patterns

### Email/Password Auth

```typescript
// Sign up
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password',
  options: {
    data: {
      full_name: 'ชื่อ นามสกุล',
    }
  }
})

// Sign in
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'secure-password',
})

// Sign out
await supabase.auth.signOut()

// Get current session
const { data: { session } } = await supabase.auth.getSession()
```

### OAuth (Google, GitHub, etc.)

```typescript
// Sign in with Google
const { error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    redirectTo: `${window.location.origin}/auth/callback`,
    queryParams: {
      access_type: 'offline',
      prompt: 'consent',
    },
  },
})

// Auth state change listener
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    console.log('User signed in:', session?.user.email)
  }
  if (event === 'SIGNED_OUT') {
    // Redirect to login
  }
})
```

## Database Operations

### CRUD with TypeScript

```typescript
// Create
const { data, error } = await supabase
  .from('products')
  .insert({
    name: 'สินค้าใหม่',
    price: 299.00,
    category: 'electronics',
    user_id: session.user.id
  })
  .select()
  .single()

// Read with filters
const { data: products } = await supabase
  .from('products')
  .select('id, name, price, category(*)')
  .eq('user_id', session.user.id)
  .order('created_at', { ascending: false })
  .range(0, 9)  // Pagination: first 10 items

// Update
const { data, error } = await supabase
  .from('products')
  .update({ price: 349.00 })
  .eq('id', productId)
  .eq('user_id', session.user.id)  // RLS double-check
  .select()
  .single()

// Delete
const { error } = await supabase
  .from('products')
  .delete()
  .eq('id', productId)
```

## Row Level Security (RLS) Policies

```sql
-- Enable RLS on a table
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Users can only see their own products
CREATE POLICY "Users view own products"
ON products FOR SELECT
USING (auth.uid() = user_id);

-- Users can only insert their own products
CREATE POLICY "Users insert own products"
ON products FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- Users can only update their own products
CREATE POLICY "Users update own products"
ON products FOR UPDATE
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- Users can only delete their own products
CREATE POLICY "Users delete own products"
ON products FOR DELETE
USING (auth.uid() = user_id);

-- Admins can see all
CREATE POLICY "Admins see all products"
ON products FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM user_roles
    WHERE user_id = auth.uid()
    AND role = 'admin'
  )
);
```

## Realtime Subscriptions

```typescript
// Subscribe to new messages in a chat room
const channel = supabase
  .channel('room-messages')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'messages',
      filter: `room_id=eq.${roomId}`,
    },
    (payload) => {
      console.log('New message:', payload.new)
      setMessages(prev => [...prev, payload.new as Message])
    }
  )
  .subscribe()

// Cleanup on component unmount
return () => {
  supabase.removeChannel(channel)
}
```

## Storage

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/profile.jpg`, file, {
    cacheControl: '3600',
    upsert: true
  })

// Get public URL
const { data: { publicUrl } } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${userId}/profile.jpg`)

// Delete file
const { error } = await supabase.storage
  .from('avatars')
  .remove([`${userId}/profile.jpg`])
```

## Edge Functions

```typescript
// supabase/functions/send-notification/index.ts
import { serve } from 'https://deno.land/std@0.208.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const { userId, message } = await req.json()

  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )

  // Your notification logic here
  const { error } = await supabase
    .from('notifications')
    .insert({ user_id: userId, message, read: false })

  return new Response(
    JSON.stringify({ success: !error }),
    { headers: { 'Content-Type': 'application/json' } }
  )
})
```

## Best Practices

1. **Use TypeScript types** - Generate types from your database schema
2. **Enable RLS** - Always enable Row Level Security on user data tables
3. **Use service role key only server-side** - Never expose in client code
4. **Implement proper error handling** - Always check `error` responses
5. **Clean up subscriptions** - Unsubscribe from realtime when component unmounts
6. **Index frequently queried columns** - Add indexes for columns used in WHERE/ORDER clauses
