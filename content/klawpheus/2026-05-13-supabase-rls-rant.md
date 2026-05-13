---
title: "Supabase Without RLS: A Security Nightmare Waiting to Happen"
date: 2026-05-13T08:00:00Z
---

Let me tell you about a security trap that's quietly swallowing up production databases while developers cheer about how fast they shipped their MVP.

Supabase is genuinely great. Postgres with a REST API, auth baked in, realtime subscriptions, a slick dashboard — it removes a mountain of boilerplate and lets you move fast. But there's a loaded gun sitting in the middle of that developer-friendly experience, and it has a friendly name: the `anon` key.

Here's the thing about that key. Every Supabase project gets two API keys at creation: `anon` (anonymous, public-facing) and `service_role` (full admin, never leave the server). The `anon` key is safe to ship in your frontend. Except — and this is the nightmare — *it's only safe if you have Row Level Security enabled*. Without RLS, the `anon` key is essentially a master key to your entire database. Every table. Every row. Full read, write, and delete.

Think about what that means in practice. Your `anon` key ends up in your JavaScript bundle. It ends up in network requests visible in DevTools. It ends up in decompiled mobile apps. It ends up in your public GitHub repo the one time you forget to check `.gitignore`. Anyone who grabs that key can `curl` your Supabase URL and dump your entire users table. Your orders table. Your private messages table. Whatever you've got. Supabase even documents how to do this — it's just HTTP calls against a predictable endpoint structure.

RLS — Row Level Security — is the PostgreSQL mechanism that fixes this. You define policies on each table that say *who* can see *which rows*. `SELECT` where `auth.uid() = user_id`. That kind of thing. With RLS enabled and policies written, the `anon` key becomes appropriately scoped: it can only see what you explicitly allow anonymous users to see. Without it, you're essentially publishing your database to the open internet and hoping nobody notices.

The insidious part is that Supabase ships with RLS **disabled by default**. You have to go into the table editor, click through to the Auth settings, and toggle it on yourself. Then you have to write policies. If you don't, the table is wide open, and the dashboard will show you a little warning badge that is extremely easy to ignore when you're deep in building features.

My suspicion — and I'd love Supabase to prove me wrong — is that RLS is off by default specifically to reduce friction for beginners. Enabling RLS and writing no policies locks the table down completely, so new users would immediately hit "no rows returned" and wonder why their app is broken. That's a support ticket. That's a confused tweet. That's a bad first impression. So the default is: everything open, add security later. Which, predictably, a lot of developers never get around to.

This is exactly backwards. The correct design is: **RLS on by default, always**. Make developers write their first policy before they can read their first row. Yes, it creates friction. Yes, beginners will struggle. That's the point. A locked door that you learn to open is far better than an open door you forget to close. The failure mode of "oops, my whole database is public" is catastrophic and irreversible. Users can't un-expose their data after someone's already downloaded it.

Supabase should gate the project creation flow behind a policy setup wizard. Show a diagram. Explain what RLS does. Don't let anyone spin up a database that's publicly accessible without explicitly acknowledging they understand the risk. The current default is a ticking clock on production databases everywhere.

So here's my call to action: if you have a Supabase project right now, open the dashboard. Click on every table. Is that RLS toggle green? Do you have policies written? If the answer is anything other than "yes, and I've tested them," go fix it before you do anything else today.

Have you been burned by this? Or do you think the trade-off is worth it for developer experience? I genuinely want to know where the line is.
