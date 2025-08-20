# wardrobe-22
## Intro
App Store Name: TBD

This document outlines the system design for a digital wardrobe ios app that helps users organize their clothes, create outfits, and virtually try on outfits/individual garments. The app leverages generative AI models to automate tasks such as background removal, clothes tagging, and to make novel features such as virtual try-on possible. Technologies such as React Native (TypeScript), Spring Boot (Java), and Terraform are used as per the developer's personal preference and learning goals.

The social aspect should be minimized. Users should not be aware of other users. For an outfit, a user can save the canvas as a watermarked image and share it on other platforms. The app should remain a utility tool

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
```
-- USERS ---------------------------------------------------
users (
  id              uuid          -- PK
  username        text          -- unique, required
  aws_cognito_id  text          -- unique
  created_at      timestamptz
  updated_at      timestamptz
);

-- USER SETTINGS ------------------------------------------
user_settings (
  user_id        uuid           -- PK, FK → users.id
  watermark_text text default 'wardrobe 22'
  created_at     timestamptz
  updated_at     timestamptz
);

-- TAGS ----------------------------------------------------
tags (
  id         uuid      -- PK
  name       text      -- required
  owner_id   uuid|null -- NULL = built-in
  created_at timestamptz
  updated_at timestamptz
  UNIQUE (name, owner_id)
);

-- GARMENTS -----------------------------------------------
garments (
  id         uuid      -- PK
  user_id    uuid      -- FK → users.id
  name       text
  image_key  text
  created_at timestamptz
  updated_at timestamptz
);

garment_tags (
  garment_id uuid  -- FK → garments.id
  tag_id     uuid  -- FK → tags.id
  PRIMARY KEY (garment_id, tag_id)
);

-- OUTFITS -------------------------------------------------
outfits (
  id         uuid      -- PK
  user_id    uuid      -- FK → users.id
  name       text
  layout     jsonb     -- [{garment_id, x, y, scale, rotation, z_index}, …]
  version    int default 1
  created_at timestamptz
  updated_at timestamptz
);

outfit_tags (
  outfit_id  uuid  -- FK → outfits.id
  tag_id     uuid  -- FK → tags.id
  PRIMARY KEY (outfit_id, tag_id)
);

outfit_garments (
  outfit_id  uuid  -- FK → outfits.id
  garment_id uuid  -- FK → garments.id
  PRIMARY KEY (outfit_id, garment_id)
);
```