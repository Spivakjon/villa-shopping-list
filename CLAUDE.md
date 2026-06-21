# CLAUDE.md — villa-shopping-list

הנחיות לסשנים עתידיים של Claude שעובדים על הפרויקט הזה.

## מה זה
דף HTML יחיד (`index.html`) — רשימת קניות אינטראקטיבית בעברית RTL לסופ"ש של 5 משפחות בווילה. vanilla JS, ללא build, ללא תלויות.

## שני מיקומי קבצים (חשוב!)
- **מקור עבודה ראשי:** `C:\Users\spiva\OneDrive\Desktop\כללי פרויקטים\Projects\villa-shopping-list.html`
- **ריפו/פריסה:** `C:\Users\spiva\villa-shopping-list\index.html`

עורכים את **המקור** ב-Projects, ואז **מעתיקים** אותו ל-`index.html` בריפו לפני push. שמור על שני הקבצים מסונכרנים.

## פריסה
- GitHub: `Spivakjon/villa-shopping-list` (public), ענף `main`, GitHub Pages משורש.
- URL חי: https://spivakjon.github.io/villa-shopping-list/
- זרימה: ערוך מקור → `Copy-Item` ל-`index.html` → `git add/commit/push` → Pages מתעדכן תוך ~1 דקה.
- commit ללא חתימה: `git -c commit.gpgsign=false commit ...`. סיים הודעות commit ב-`Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`.
- אימות: poll את ה-URL (עם `?v=` כדי לעקוף cache) עד שהתוכן החדש מופיע.

## ארכיטקטורת הקוד (הכל ב-`index.html`)
- `DEFAULT` — מערך הקטגוריות וברירת המחדל של הפריטים. **כאן משנים כמויות/פריטים.**
- כל קטגוריה: `{id, emoji, name, optional?, included?, items:[{n, q, tag?}]}`.
  - `optional:true` + `included` → קטגוריה עם **מתג** (כמו "על האש #1"). כשכבויה — מאופרת, לא נספרת ב-`updateProgress`, לא ניתנת לסימון.
  - `tag:"note"` → תג "אופציה"; `tag:"vegan"` → תג "טבעוני".
- State נשמר ב-`localStorage` תחת `KEY`. **כל שינוי מבני/תוכני משמעותי → להעלות את גרסת ה-KEY** (`villa_shop_v4` → `v5` ...) כדי לאלץ רענון אצל מי שכבר טען (אחרת הם רואים state ישן שמור).
- פונקציות עיקריות: `render`, `toggle`, `toggleInclude`, `upd`, `addItem`, `delItem`, `addCategory`, `updateProgress`, `toggleEdit`, `resetChecks`, `restoreDefault`.

## פרמטרים של האירוע (להזכר)
- 10 מבוגרים + 10 ילדים (≈4 מנות) = **14 מנות**. תינוק חודשיים = 0 (לא אוכל, לא ברשימה).
- 2 לילות. **מנגל גז** (לא פחם).
- 2 נשים → סלמון במקום בשר. **לאשר אם טבעוניות** — אם כן, להחזיר תחליפים צמחיים.
- ארוחות: על האש #1 (אופציונלי/מתג) · על האש #2 · צהריים המבורגרים · בוקר (שקשוקה/ג'חנון/פנקייקים/קורנפלקס/חטיפים) · נקניקיות בלחמניה · סלמון.

## העדפות שהמשתמש כבר ביקש (לא לחזור עליהן)
- **בלי הערות/תיבות מידע על המסך** — הדף נקי. אם צריך להבהיר משהו — לעשות זאת בצ'אט, לא בדף.
- כמויות נדיבות לפי 14 מנות; מנגל = הציר המרכזי.

## רעיון להמשך (לא מומש)
סנכרון חי בין כל המשפחות (Firebase/Supabase) במקום `localStorage` פר-מכשיר.
