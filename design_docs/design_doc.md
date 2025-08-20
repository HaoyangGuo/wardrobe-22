# wardrobe-22
## Intro
App Store Name: TBD

This document outlines the system design for a digital wardrobe ios app that helps users organize their clothes, create outfits, and virtually try on outfits/individual garments. The app leverages generative AI models to automate tasks such as background removal, clothes tagging, and to make novel features such as virtual try-on possible. Technologies such as React Native (TypeScript), Spring Boot (Java), and Terraform are used as per the developer's personal preference and learning goals.

The social aspect should be minimized. Users should not be aware of other users. For an outfit, a user can save the canvas as a watermarked image and share it on other platforms. The app should remain a utility tool.

In short, the target users of this app are young people who are fashion-conscious and would like to digitize their wardrobe to better organize their garments and outfits.

## Features
* Garment Management
    - Users can upload pictures of their clothes.
    - The background of uploaded pictures will be removed.
    - Each individual garment will be tagged based on a set of default tags.
    - Users can then browse their entire collection of clothing with sorting and filtering.
    - Users can create additional custom tags for granular filtering.
* Outfit Management
    - Users can drop individual uploaded garments onto a canvas to create outfits.
    - Each created outfit will be tagged based on a set of default tags.
    - Users can then browse their entire collection of outfits with sorting and filtering.
    - Users can create additional custom tags for granular filtering.
* Virtual Try-on
    - Users can upload a picture of themselves and select individual garments or outfits. An AI-generated picture of the user wearing such piece(s) will be created.

## Dev Roadmap
1. **Foundations**: Garment management + outfit management backend
2. **MVP**: Garment management + outfit management frontend
3. **Novelty**: Virtual try-on capabilities (fullstack)
4. **SaaS**: Credit system + Apple App Store payment system integration
5. **Infrastructure**: Terraform for Hetzner + AWS
6. **Testing**: Integration tests and TestFlight publishing
7. **Launch**: App Store release

## Current Data Models
- users
  - id: uuid primary key
  - email: required text, can get from aws cognito
  - aws_cognito_id: sub from aws cognito
  - created_at: timestamptz
  - updated_at: timestamptz

- user_settings: one-to-one relation with users
  - user_id: uuid primary and foreign key
  - watermark_text: text default to 'wardrobe 22'
  - created_at: timestamptz
  - updated_at: timestamptz

- tags: one-to-many relation with users
  - id: uuid primary key
  - user_id: uuid foreign key, each user also own their own default ones, which they can delete
  - name: required text, unique at user level
  - created_at: timestamptz
  - updated_at: timestamptz

- garments: one-to-many relation with users
  - id: uuid primary key
  - user_id: uuid foreign key
  - name: required text
  - image_key: text, likely will use the id to build
  - created_at: timestamptz
  - updated_at: timestamptz

- garment_tags: many-to-many join table for garments and tags
  - garment_id: uuid foreign key
  - tag_id: uuid foreign key
  - user_id: uuid foreign key (must match garment.user_id and tag.user_id)
  - PRIMARY KEY (user_id, garment_id, tag_id)
  - rules:
    - FK (garment_id, user_id) -> garments (id, user_id)
    - FK (tag_id, user_id) -> tags (id, user_id)
    - On delete garment/tag: cascade

- outfits: one-to-many relation with users
  - id: uuid primary key
  - user_id: uuid foreign key
  - name: required text, unique at user level
  - version: int default 1 to lock
  - created_at: timestamptz
  - updated_at: timestamptz

- outfit_tags: many-to-many join table for outfits and tags
  - outfit_id: uuid foreign key
  - tag_id: uuid foreign key
  - user_id: uuid foreign key (must match outfit.user_id and tag.user_id)
  - PRIMARY KEY (user_id, outfit_id, tag_id)
  - rules:
    - FK (outfit_id, user_id) -> outfits (id, user_id)
    - FK (tag_id, user_id) -> tags (id, user_id)
    - On delete outfit/tag: cascade

- outfit_items: normalized placement of garments within an outfit
  - id: uuid primary key
  - user_id: uuid foreign key (must match outfit.user_id and garment.user_id)
  - outfit_id: uuid foreign key
  - garment_id: uuid foreign key
  - x: number (canvas X position)
  - y: number (canvas Y position)
  - scale: number > 0
  - rotation: number in [0, 360)
  - z_index: int (rendering order)
  - created_at: timestamptz
  - updated_at: timestamptz
  - rules:
    - FK (outfit_id, user_id) -> outfits (id, user_id)
    - FK (garment_id, user_id) -> garments (id, user_id)
    - optionally UNIQUE (user_id, outfit_id, garment_id) if a garment cannot appear twice
    - On delete outfit: cascade; on delete garment: restrict (or cascade if desired)

