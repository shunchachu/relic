# ⏳ Relic | 時光餘額

> **Live a life you will remember.**  
> 一款結合侘寂美學與 AI 智慧精算的陪伴倒數與回憶封存 App。

![Relic Banner](https://images.unsplash.com/photo-1516715094483-75da7dee9758?auto=format&fit=crop&w=1200&q=80)

---

## 💡 專案初衷 (Concept)

人們常說「有空再回家」，卻忽略了扣除工作、睡眠與通勤後，這輩子能真正圍坐在餐桌前陪伴父母或摯友的時間，可能只剩下幾百個小時。

**Relic (時光餘額)** 將抽象的時間具象化為「實質純陪伴小時數」，並結合陪伴任務與「回憶花園」養成系統，引導使用者將對時間的焦慮轉化為實質的陪伴行動。

---

## 🌟 核心功能 (Key Features)

- **⏳ 時光指針 (Time Pointer)**：獨立計算對象的剩餘相處時間，實時顯示純陪伴時數與動態刻度。
- **🤖 Gemini AI 陪伴精算**：輸入自然語言描述（如返鄉頻率、過年行程），由 AI 自動扣減無效時間並推算年均陪伴小時。
- **🌿 安排驚喜見面 (+Bonus)**：提供快捷與 AI 自訂行程推估，主動增加相聚餘額。
- **📜 時光手札 (Memory Capsule)**：支援多圖上傳與文字手札，無損封存每次相聚的片刻。
- **🌸 回憶花園 (Memory Garden - Pro)**：解鎖專屬庭園，完成每日暖心陪伴任務進行植物澆灌與花木培育。
- **🌐 多語系與系統模擬 (i18n & OS Mock)**：支援繁體中文、英文、日文，並可切換 iOS / Android 視覺體驗。

---

## 🛠️ 技術棧 (Tech Stack)

- **Front-end**: HTML5, Tailwind CSS, JavaScript (ES6+)
- **Icons & Fonts**: Google Fonts (Noto Serif TC, Inter), Canvas Confetti
- **AI Engine**: Google Gemini API (`gemini-3-flash-preview`)
- **Back-end & Database**: Supabase (PostgreSQL + Row Level Security)
- **Storage**: Supabase Storage (`relic-photos` bucket)
- **Audio & FX**: Web Audio API (528Hz Zen Chime System)

---

## 🗄️ 後端建置 (Supabase Database Setup)

請在 Supabase SQL Editor 執行以下建表腳本：

```sql
-- 1. 啟用 UUID 擴充功能
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 2. 建立計算對象表 (targets)
CREATE TABLE public.targets (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  relation TEXT,
  turning_date DATE NOT NULL,
  turning_reason TEXT,
  frequency TEXT DEFAULT 'monthly',
  hours_per_year INT DEFAULT 144,
  bonus_hours INT DEFAULT 0,
  model_type TEXT DEFAULT 'ai',
  ai_summary TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. 建立時光手札與照片紀錄表 (journals)
CREATE TABLE public.journals (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  target_id UUID REFERENCES public.targets(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  title TEXT NOT NULL,
  hours INT DEFAULT 24,
  location TEXT,
  content TEXT NOT NULL,
  photos TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 4. 建立回憶花園植物表 (garden_plants)
CREATE TABLE public.garden_plants (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  variety TEXT NOT NULL,
  meaning TEXT,
  story TEXT,
  stage INT DEFAULT 1,
  water_count INT DEFAULT 0,
  cultivations INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 5. RLS 隱私權限啟用
ALTER TABLE public.targets ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.journals ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.garden_plants ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users access own targets" ON public.targets FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "Users access own journals" ON public.journals FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "Users access own garden_plants" ON public.garden_plants FOR ALL USING (auth.uid() = user_id);
