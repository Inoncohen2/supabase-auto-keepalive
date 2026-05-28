# 🟢 Keep Supabase Alive: מניעת השהיית פרויקטים

<p align="center">
  <img alt="Supabase Active" src="https://img.shields.io/badge/Supabase-Active-green" />
  <img alt="GitHub Actions" src="https://img.shields.io/badge/GitHub-Actions-blue" />
  <img alt="Automation" src="https://img.shields.io/badge/Automation-Enabled-success" />
</p>

<p align="center">
  <img src="./preview.png" alt="Preview" />
</p>

פתרון אוטומטי למניעת השהייה (Pause) של פרויקטים חינמיים ב-Supabase עקב חוסר פעילות, באמצעות GitHub Actions שמבצע פעילות Database אמיתית באופן אוטומטי.

---

# 🎯 הבעיה

במסלול החינמי של Supabase, פרויקט שלא מתבצעת בו פעילות Database במשך 7 ימים רצופים — מושעה אוטומטית.

זה קורה הרבה בפרויקטים כמו:

- מחשבון חשמל / מים
- דשבורד אישי
- מערכת פנימית
- אפליקציות לשימוש עצמי
- פרויקטי Side Project

במיוחד כשנכנסים אליהם רק פעם בכמה שבועות.

---

# 💡 הפתרון

האוטומציה הזו משתמשת ב־GitHub Actions כדי לבצע Query אמיתי למסד הנתונים כל יומיים.

כך:
- Supabase מזהה פעילות אמיתית
- הטיימר של 7 הימים מתאפס
- והפרויקט נשאר פעיל תמיד

---

# ✅ מה הסקריפט כולל

- ✅ Query אמיתי ל־Database
- ✅ ריצה אוטומטית כל יומיים
- ✅ תמיכה בהרצה ידנית
- ✅ הגנה מקריסות רשת רגעיות
- ✅ שמירה על GitHub Actions פעיל
- ✅ שימוש מאובטח ב־GitHub Secrets

---

# ⚙️ הוראות התקנה

---

# שלב 1: השגת פרטי Supabase

1. היכנסו ל־Supabase
2. בחרו את הפרויקט שלכם
3. בתפריט הצד:

```txt
Settings → API
```

העתיקו:

- Project ID
- anon public key

---

# שלב 2: הגדרת Secrets ב־GitHub

היכנסו ל־Repository שלכם:

```txt
Settings → Secrets and variables → Actions
```

לחצו:

```txt
New repository secret
```

והוסיפו:

## Secret ראשון

```txt
Name: SUPABASE_PROJECT_ID
Value: YOUR_PROJECT_ID
```

## Secret שני

```txt
Name: SUPABASE_ANON_KEY
Value: YOUR_SUPABASE_ANON_KEY
```

---

# שלב 3: יצירת קובץ האוטומציה

צרו קובץ חדש בנתיב:

```txt
.github/workflows/keep-supabase-alive.yml
```

---

# ⚠️ חשוב לפני ההדבקה

לפני שמריצים את הסקריפט:

החליפו את:

```txt
your_table_name
```

בשם של טבלה אמיתית וציבורית שקיימת בפרויקט שלכם.

---

# 📄 הקוד המלא

```yaml
name: Keep Supabase Alive

on:
  schedule:
    # Runs every 2 days at 08:00 UTC
    - cron: '0 8 */2 * *'

  # Allows manual execution
  workflow_dispatch:

jobs:
  ping-supabase:
    runs-on: ubuntu-latest
    name: Ping Supabase Project

    permissions:
      contents: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v5
        with:
          fetch-depth: 1

      - name: Query Supabase Database
        continue-on-error: true

        env:
          SUPABASE_PROJECT_ID: ${{ secrets.SUPABASE_PROJECT_ID }}
          SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}

        run: |
          # Real database activity
          # Replace 'your_table_name' with a real public table
          URL="https://${SUPABASE_PROJECT_ID}.supabase.co/rest/v1/your_table_name?select=id&limit=1"

          echo "Querying Supabase DB: $SUPABASE_PROJECT_ID..."

          HTTP_CODE=$(curl -s -o /tmp/resp.json -w "%{http_code}" -X GET "$URL" \
            -H "apikey: $SUPABASE_ANON_KEY" \
            -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
            -H "Accept: application/json" \
            --max-time 15)

          echo "HTTP status: $HTTP_CODE"
          echo "Response body:"
          cat /tmp/resp.json || true
          echo ""

          if [ "$HTTP_CODE" = "200" ]; then
            echo "✅ Real DB query succeeded — activity registered."
          else
            echo "⚠️ Unexpected status code: $HTTP_CODE"
            exit 1
          fi

      - name: Keep workflow alive
        run: |
          # Prevent GitHub Actions auto-disable on inactive repos
          LAST_COMMIT=$(git log -1 --format=%ct)
          NOW=$(date +%s)
          DAYS_AGO=$(( (NOW - LAST_COMMIT) / 86400 ))

          if [ "$DAYS_AGO" -ge 30 ]; then
            echo "No commits for $DAYS_AGO days, pushing keepalive commit..."

            git config user.name "github-actions[bot]"
            git config user.email "github-actions[bot]@users.noreply.github.com"

            git commit --allow-empty -m "chore: keep workflow alive"
            git push
          else
            echo "Last commit was $DAYS_AGO days ago, no keepalive needed."
          fi
```

---

# 🛠️ איך לבדוק שזה עובד

היכנסו לטאב:

```txt
Actions
```

בחרו:

```txt
Keep Supabase Alive
```

ולחצו:

```txt
Run workflow
```

אם הכול תקין:
- תקבלו HTTP 200
- ו־Supabase ירשום פעילות

---

# ❗ תקלות נפוצות

## 401 / 403

בדקו שה־Secrets הוגדרו נכון:

```txt
SUPABASE_PROJECT_ID
SUPABASE_ANON_KEY
```

---

## 404 Not Found

כנראה שעדיין מופיע:

```txt
your_table_name
```

החליפו אותו בשם טבלה אמיתי.

---

## אין פעילות ב־Supabase

ודאו:
- שהטבלה קיימת
- שהיא ציבורית
- וש־RLS מאפשר SELECT

---

# 🔐 אבטחה

לעולם אל תכניסו מפתחות Supabase ישירות לקוד.

השתמשו תמיד ב־GitHub Secrets.

---

# ✅ סיימתם

מעכשיו הפרויקט שלכם ב־Supabase יישאר פעיל אוטומטית 🚀
