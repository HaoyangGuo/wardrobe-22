# Foundations
## Data Models
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