---
layout: post
title: "What Happens When Your API Has to Open a Door"
description: "Building software at ROOM that controls physical locks, tracks IoT sensors, and manages office spaces — the weird problems you don't get in pure SaaS."
date: 2021-03-15
keywords: "IoT development, smart locks API, room booking platform, particle IoT, rails node typescript, react native, physical space software"
---

I've been at [ROOM](https://room.com) for a while now, working across their platform. ROOM makes phone booths and meeting rooms for offices, and the software side is about making those physical spaces bookable, trackable, and manageable. We run a Rails API for the B2B portal, a Node/TypeScript API for the consumer booking app, and a React Native mobile app.

Most of my bugs can't be reproduced on localhost.

## Unlocking doors over HTTP

The booking app lets people reserve a phone booth, walk up to it, and unlock it from their phone. We integrate with Salto and Tapkey hardware locks. When a user taps "unlock," our API sends a command to a physical lock on a physical door in a physical building somewhere.

This changes how you think about error handling. HTTP timeouts mean something different when the consequence is someone standing in a hallway. We built fallback flows with OTP codes, lock status polling, and graceful degradation for when the hardware goes offline.

The interesting edge case: what happens when someone is inside and their card declines? You can't lock someone inside a room because their card declined. The physical world doesn't care about your payment flow.

![ROOM booking app — find a Focus Room on the map, reserve it, scan to start](/assets/images/room--booking-app.webp)

## The data pipeline nobody warned me about

Every ROOM unit has a Particle IoT device inside it. Tracks occupancy — when someone enters, when they leave, signal strength, device health. That data flows through Google Pub/Sub into BigQuery, and our Rails API pulls it back out for analytics dashboards.

Sounds simple: device sends event, Pub/Sub queues it, BigQuery stores it, Rails reads it back. In practice: devices drop offline, send duplicate events, have clock drift. A unit in Tokyo and a unit in New York need their sessions in the right timezone. Devices sometimes report "occupied" when nobody's there (the sensor picked up movement outside the booth). You end up writing reconciliation logic you never planned for.

The portal shows office managers how their spaces are actually used. Turns out most companies have no idea. The data is usually surprising — the meeting room that seats 4 gets used by 1 person for calls, and the phone booths are at 90% utilization.

![The Sense analytics dashboard — utilization data from IoT sensors across office spaces](/assets/images/room--sense-dashboard.webp)

## Two stacks, same door

The B2B portal is Rails — organizations, offices, units, NetSuite for invoicing, Salesforce sync. The consumer app is Node with Prisma and TypeScript — sessions, Stripe payments, magic link auth.

Different products, different users, different tech stacks, same physical units.

A unit registered in the portal needs to show up in the booking app. Pricing changes in one system need to reflect in the other. We made it work but it was a recurring source of bugs. Back when I was [just getting started with Rails](/blog/why-i-build-with-ruby-on-rails/) I thought picking the right framework was the hard part. Turns out keeping two of them in sync is harder.

## What I took away from this

Software for physical spaces is humbling. You can't roll back a deployment when someone is locked inside a phone booth. Latency matters differently when it's the gap between tapping a button and hearing a lock click. Your test suite can't open a real door.

Before ROOM everything I built lived in the browser. IoT work broke that mental model. Redis caching, background job patterns, careful error handling — I learned all of it here the hard way, because a sensor was lying about occupancy at 3am.
