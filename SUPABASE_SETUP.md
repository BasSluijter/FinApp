# Financieel Overzicht - Supabase Setup

## Stap 1: Supabase Account Maken

1. Ga naar [supabase.com](https://supabase.com)
2. Klik op "Start your project"
3. Maak een gratis account aan met je email
4. Klik op "New Project"
5. Vul in:
   - **Name**: finance-tracker (of een andere naam)
   - **Database Password**: kies een sterk wachtwoord (sla dit op!)
   - **Region**: West EU (Netherlands) - dichtst bij Nederland
   - **Pricing Plan**: Free
6. Klik op "Create new project"
7. Wacht 2-3 minuten tot het project klaar is

## Stap 2: Project Instellingen Ophalen

1. Klik in het linker menu op het **tandwiel icoon** (Settings)
2. Ga naar **API**
3. Kopieer de volgende waardes (je hebt deze straks nodig):
   - **Project URL** (bijvoorbeeld: https://abcdefgh.supabase.co)
   - **anon public** key (lange string, begint vaak met "eyJ...")

## Stap 3: Database Schema Aanmaken

1. Klik in het linker menu op **SQL Editor**
2. Klik op **+ New query**
3. Kopieer de SQL code hieronder en plak in de editor
4. Klik op **Run** (of druk Ctrl+Enter)

```sql
-- Households (gezinnen/bedrijven)
CREATE TABLE households (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  invite_code TEXT UNIQUE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()) NOT NULL
);

-- Users (uitgebreide user info)
CREATE TABLE profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  full_name TEXT,
  display_name TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()) NOT NULL
);

-- Items (transacties/items)
CREATE TABLE items (
  id BIGSERIAL PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE NOT NULL,
  name TEXT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('inkomsten', 'uitgaven')),
  frequency TEXT NOT NULL CHECK (frequency IN ('eenmalig', 'maandelijks', 'specifiek')),
  month INTEGER CHECK (month >= 0 AND month <= 11),
  year INTEGER,
  selected_months INTEGER[] DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()) NOT NULL
);

-- Checked items (afvinkstatus per maand)
CREATE TABLE checked_items (
  id BIGSERIAL PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE NOT NULL,
  item_id BIGINT REFERENCES items(id) ON DELETE CASCADE NOT NULL,
  year INTEGER NOT NULL,
  month INTEGER NOT NULL CHECK (month >= 0 AND month <= 11),
  checked BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()) NOT NULL,
  UNIQUE(household_id, item_id, year, month)
);

-- Sort order per maand
CREATE TABLE sort_orders (
  id BIGSERIAL PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE NOT NULL,
  year INTEGER,
  month INTEGER CHECK (month >= 0 AND month <= 11),
  item_order BIGINT[] NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()) NOT NULL,
  UNIQUE(household_id, year, month)
);

-- Settings (thema voorkeur etc)
CREATE TABLE user_settings (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  dark_mode BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc', NOW()) NOT NULL
);

-- Enable Row Level Security
ALTER TABLE households ENABLE ROW LEVEL SECURITY;
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE items ENABLE ROW LEVEL SECURITY;
ALTER TABLE checked_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE sort_orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_settings ENABLE ROW LEVEL SECURITY;

-- RLS Policies voor households
CREATE POLICY "Users can view their own household"
  ON households FOR SELECT
  USING (id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can update their own household"
  ON households FOR UPDATE
  USING (id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

-- RLS Policies voor profiles
CREATE POLICY "Users can view their own profile"
  ON profiles FOR SELECT
  USING (id = auth.uid());

CREATE POLICY "Users can update their own profile"
  ON profiles FOR UPDATE
  USING (id = auth.uid());

CREATE POLICY "Users can insert their own profile"
  ON profiles FOR INSERT
  WITH CHECK (id = auth.uid());

-- RLS Policies voor items
CREATE POLICY "Users can view items from their household"
  ON items FOR SELECT
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can insert items to their household"
  ON items FOR INSERT
  WITH CHECK (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can update items in their household"
  ON items FOR UPDATE
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can delete items from their household"
  ON items FOR DELETE
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

-- RLS Policies voor checked_items
CREATE POLICY "Users can view checked items from their household"
  ON checked_items FOR SELECT
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can insert checked items to their household"
  ON checked_items FOR INSERT
  WITH CHECK (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can update checked items in their household"
  ON checked_items FOR UPDATE
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can delete checked items from their household"
  ON checked_items FOR DELETE
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

-- RLS Policies voor sort_orders
CREATE POLICY "Users can view sort orders from their household"
  ON sort_orders FOR SELECT
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can insert sort orders to their household"
  ON sort_orders FOR INSERT
  WITH CHECK (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can update sort orders in their household"
  ON sort_orders FOR UPDATE
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

CREATE POLICY "Users can delete sort orders from their household"
  ON sort_orders FOR DELETE
  USING (household_id IN (
    SELECT household_id FROM profiles WHERE id = auth.uid()
  ));

-- RLS Policies voor user_settings
CREATE POLICY "Users can view their own settings"
  ON user_settings FOR SELECT
  USING (id = auth.uid());

CREATE POLICY "Users can insert their own settings"
  ON user_settings FOR INSERT
  WITH CHECK (id = auth.uid());

CREATE POLICY "Users can update their own settings"
  ON user_settings FOR UPDATE
  USING (id = auth.uid());

-- Function om automatisch een household en profile aan te maken bij registratie
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
DECLARE
  new_household_id UUID;
  new_invite_code TEXT;
BEGIN
  -- Genereer een unieke invite code (6 karakters)
  new_invite_code := upper(substring(md5(random()::text) from 1 for 6));
  
  -- Maak een nieuw household aan met invite code
  INSERT INTO public.households (name, invite_code)
  VALUES ('Mijn Huishouden', new_invite_code)
  RETURNING id INTO new_household_id;
  
  -- Maak profile aan en link aan household
  INSERT INTO public.profiles (id, household_id, email)
  VALUES (NEW.id, new_household_id, NEW.email);
  
  -- Maak default settings aan
  INSERT INTO public.user_settings (id, dark_mode)
  VALUES (NEW.id, FALSE);
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger om bovenstaande functie uit te voeren bij nieuwe user
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

## Stap 4: Email Authenticatie Instellen

1. Klik in het linker menu op **Authentication**
2. Ga naar **Providers**
3. Zorg dat **Email** aan staat (standaard aan)
4. (Optioneel) Pas email templates aan onder **Email Templates**

## Stap 5: Klaar!

Je Supabase backend is nu klaar! Je hebt nodig voor de app:
- ✅ Project URL
- ✅ Anon key

Bewaar deze goed - je hebt ze nodig om de HTML app te configureren.

---

## Extra: Household Delen (Voor jou en je vrouw)

**Optie A: Zelfde account delen**
- Beide inloggen met hetzelfde email/wachtwoord
- Simpelste oplossing

**Optie B: Aparte accounts, gedeeld household**
1. Eerste persoon registreert → krijgt automatisch een household
2. Via SQL Editor, link tweede account aan zelfde household:

```sql
-- Haal het household_id op van de eerste user
SELECT household_id FROM profiles WHERE email = 'eerste@email.com';

-- Update de tweede user om hetzelfde household_id te gebruiken
UPDATE profiles 
SET household_id = 'PLAK_HIER_HET_HOUSEHOLD_ID'
WHERE email = 'tweede@email.com';
```

Later kun je hier een mooie invite-functie voor bouwen!
