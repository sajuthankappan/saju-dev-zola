+++
title = "WCCG Website Architecture"
description = "WCCG Website Architecture (chennaicyclists.com)"
date = 2021-08-06
[taxonomies]
tags =["first"]
+++

### Overall View

Overall View of Main Site, Rider Site & Admin / Volunteer Sites

![WCCG Overall View](/wccg-simplified.png "Overall View")

### Strava Integration View

![WCCG Strava Integration View](/wccg-strava-integration.png "Strava Integration View")

### What we use?

- Firebase hosting (static website hosting)
- Firebase Authentication (User Regisration & Authentication)
- Azure APIM (Api Gateway)
- Google Cloud Run (Microservices hosting)
- Contentful (Headless CMS)
- Cloudinary (Image CDN)
- Atlas Mongodb (Databsae)
- CloudAMQP (RabbitMQ as a Service)

### Tech Stack

#### Programming Languages

- Javascript & Typescript (Most of frontend)
- Java (Backend Services)
- Rust (Backend Services & Strava Worker)

#### Frameworks / Libraries

- React & Next.js (For main site)
- Svelte (For Rider & Admin site)
- Spring Boot (Java Microservices Framework)
- Warp (Rust Microservices Library)
-

#### Drivers
- [Mongodb Rust Driver](https://github.com/mongodb/mongo-rust-driver)
- [Rust AMQP Client Library](https://github.com/CleverCloud/lapin)