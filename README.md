The purpose of this project is to demonstrate the issue where when using a Magic Link in a Supabase local development setup, the `sb-localhost-auth-token` cookie does not get set.

This project was setup following the instructions on the [Setting Up Server-Side Auth For Next.js](https://supabase.com/docs/guides/auth/server-side/nextjs) page in the Supabase docs.

After confirming that everything worked with regular signup and login, I updated the app to use a Magic Link. I receive the Magic Link in Inbucket, and everything seems to be working except that the `sb-localhost-auth-token` cookie does not get set.

## Test it yourself

Start `Supabase` with:

```
supabase start
```

Copy anon key you get back after starting Supabase and put it into an `.env.local` file with the following:

```
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your_supabase_anon_key>
```

Run the development server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to confirm the app is running.

Now go to [http://localhost:3000/login](http://localhost:3000/login), enter any email (doesn't matter as the Magic Link will be sent to Inbucket, not your email) and click the Magic Link button.

Go to Inbucket at [http://127.0.0.1:54324](http://127.0.0.1:54324) and follow the Magic Link.

Now that you are back in the app, open the inspector to the Application tab, then view the cookies under the storage section. You will notice that there is no `sb-localhost-auth-token` cookie set.

Attempt to open [http://localhost:3000/private](http://localhost:3000/private) and notice that you can't view that page. This is the code for `app/private/page.tsx`:

```typescript
import { redirect } from 'next/navigation';

import { createClient } from '@/utils/supabase/server';

export default async function PrivatePage() {
  const supabase = createClient();

  const { data, error } = await supabase.auth.getUser();
  if (error || !data?.user) {
    redirect('/');
  }

  return <p>Hello {data.user.email}</p>;
}
```

Without the `sb-localhost-auth-token` cookie set `supabase.auth.getUser()` cannot get any data, so the user is redirected to `/`.
