# CLAUDE.md — villa-shopping-list

הנחיות לסשנים עתידיים של Claude. הפרויקט הוא רשימת קניות **משותפת אונליין** ל-5 משפחות (וילה סופ"ש). עברית RTL, נייד.

## ארכיטקטורה (חשוב!)
מערכת בשני חלקים:
1. **Backend (מקור האמת):** שרת Node טהור (ללא תלויות) על **Railway**.
   - מונורפו: `C:\Users\spiva\OneDrive\Desktop\כללי פרויקטים\Projects\villa-shop\` → `server.js`, `package.json`, `public/index.html`.
   - Railway project `villa-shop`, service id `57f5ca3d-6aff-4f0f-aa4a-c019727d829f`, URL `https://villa-shop-production.up.railway.app`.
   - Volume מחובר ב-`/data` (env `DATA_DIR=/data`). מצב הרשימה ב-`/data/villa-state.json`, גיבויים ב-`/data/backups/`.
2. **Frontend (פריסה):** GitHub Pages — ריפו `Spivakjon/villa-shopping-list`, `index.html` בשורש, ענף `main`, URL `https://spivakjon.github.io/villa-shopping-list/`.

### שני עותקי הקליינט (לשמור מסונכרנים!)
- **מקור קנוני:** `villa-shop/public/index.html` עם `const API_BASE = "";` (מוגש ע"י השרת באותו origin).
- **עותק פריסה:** `index.html` בריפו — זהה, אבל `API_BASE = "https://villa-shop-production.up.railway.app"` (cross-origin).
- זרימת עריכה: ערוך את `villa-shop/public/index.html` → העתק לריפו → החלף `API_BASE=""` לכתובת Railway → `git push`.

## פריסה
- **שרת (אחרי שינוי ב-server.js):** `cd villa-shop && railway up --ci -s 57f5ca3d-6aff-4f0f-aa4a-c019727d829f`. ה-`rev` הוא in-memory ומתאפס ל-1 בכל redeploy — תקין; הלקוחות מקבלים עץ מלא ב-SSE. ה-Volume שומר את הנתונים בין פריסות. `.gitignore` מחריג `node_modules/ backups/ villa-state.json`.
- **קליינט (Pages):** העתק → החלף API_BASE → `git push`. Pages מתעדכן ~1 דקה. אמת ב-`curl` (grep לסטרינג חדש).
- commit: `git -c user.name="Spivakjon" -c user.email="spivak123@gmail.com" commit`. סיים ב-`Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`.

## מודל הנתונים (העץ)
```
{ cats: { <cid>: {
    emoji, name, optional, included, bring, order,
    items: { <uid>: { n, q, tag, done, by, prevBy, at, order } }
}}}
```
- `optional:true`+`included` → קטגוריה עם **מתג** (על האש #1). כשכבויה: לא נספרת ב-`updateProgress`, לא ניתנת לסימון.
- `bring:true` → קטגוריית "מי מביא" — סימון = הצהרה "אני מביא", רושם `by` (עם `prevBy` מחוק אם לוקחים אחריות במקום מישהו).
- `by`/`prevBy`/`at` → תיעוד "מי עדכן ומתי". **רק עריכה אמיתית/הצהרת-מביא רושמת שם — לא סימון checkbox רגיל.**
- `tag:"note"`→"אופציה"; `tag:"vegan"`→"טבעוני".

## API של השרת
- `GET /` → מגיש `public/index.html`.
- `GET /api/state` → `{rev, tree}`.
- `GET /api/stream?cid=&name=` → SSE. הודעות: snapshot `{rev,tree,users}` או presence `{users}`. נוכחות נשמרת לפי `cid`.
- `POST /api/update` → גוף עם `{updates?, setNode?, remove?, setTree?, seedIfEmpty?}` (deep-paths יחסית ל-`cats`).
- `GET /api/backups` · `GET /api/backup?name=` · `POST /api/snapshot` · `POST /api/restore {name}` (מגבה את הנוכחי לפני שחזור).
- השרת **גנרי** — לא יודע על המבנה; שומר כל JSON. הזריעה הראשונית מגיעה מהלקוח (`seedIfEmpty`).

## הקליינט (`index.html`) — נקודות מפתח
- `DEFAULT` — תבנית הפריטים. **כאן משנים כמויות/פריטים בברירת מחדל.** (שינוי כאן לא משפיע על הרשימה החיה — היא כבר זרועה; להוסיף לחי דרך `POST /api/update`.)
- `localStorage`: `villa_shop_v5` (מטמון), `villa_shop_v4` (legacy/seed/recovery), `villa_name`, `villa_cid`.
- `FAMILIES` — 5 הזוגות לבחירה בכניסה: קים/יהונתן · אלון/דנה · דניאל/עמית · גבריאל/טל · ארז/כרמל. `showNamePicker()`/`setMe()`.
- `applyTree()` מדלג על render בזמן הקלדה בשדה טקסט (אנטי-קפיצה בנייד); מחיל ממתין אחרי blur.
- `?recover=1` → `showRecovery()` מציג את ה-localStorage של המכשיר להעתקה (שחזור ידני).
- פונקציות: `render, toggle, upd(+prevBy), commitGhost, cellKey (Enter כמו אקסל), moveItem, moveCat, toggleMove, buildNav/jumpTo, connectStream/renderPresence, snapshot/restore (server).

## פרמטרי האירוע (להזכר)
14 מנות (10 מבוגרים + 10 ילדים≈4) · 2 לילות · מנגל **גז** · 2 נשים→סלמון · תינוק חודשיים=0.

## העדפות משתמש (לא לחזור עליהן)
- **דף נקי — בלי תיבות מידע/הערות על המסך.** הבהרות — בצ'אט בלבד.
- **כפתור "ברירת מחדל" הוסר בכוונה** (מחק הכל לכולם בלחיצה) — אל תחזיר אותו.
- שמירת נתונים קריטית: **תמיד לגבות לפני שינוי הרסני** (`POST /api/snapshot`), ולא לדרוס מצב חי.
